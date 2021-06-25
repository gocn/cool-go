# 单机限流器time/rate库

##### 推荐编辑：不落凡尘

## 1、限流是什么
顾名思义，限流就是只在一段时间内对流通量的管控限制。限流器是后台服务中的非常要的组件，用于限制请求速率，保护服务，避免服务过载。

golang标准库中自带了令牌桶限流算法的实现**golang.org/x/time/rate**。
## 2、如何安装
使用前要安装它，安装要在当前项目所在目录中执行 `go get -u golang.org/x/time/rate` 即可。

## 3、什么是限流器

### 漏桶限流器 

漏桶算法是比较常见的限流算法，其原理就如漏斗一样，进水口是请求流，出水口是服务流，如果请求流量过大，则会导致溢出从而拒绝请求。
<img src="https://image-static.segmentfault.com/312/532/3125325285-5b6e490397691_fix732" >

在Go语言中有一种数据结构可以十分贴切的模拟漏桶，那就是chan。
```
	//桶的容量为Max
	ch := make(chan *Request， Max)

	//流量进入
	if(len(ch)<Max){
		ch <- req //桶未满
	}else {
		cancel(req) //桶满，拒绝请求
	}

	//流量处理 预设处理间隔 m
	for {
		time.Sleep(m * time.Second)
		r := <- ch
		Handle(r)
	}
```
这是最简单的漏桶实现的代码。

当桶没有积水时，即入桶速率更小，此时以入桶速率处理请求。当桶中有积水时，即入桶速率更大，此时以预设出桶速率处理请求。当桶满时，丢弃入桶请求。
### 令牌桶限流器
令牌桶的工作流程则分为三个阶段：生产令牌，判定是否处理，消耗令牌。
<img src="https://pics7.baidu.com/feed/00e93901213fb80e919bd8536aa18328b8389467.jpeg?token=c9f7d7694e37a48f86021991a825145f">

生产令牌：系统会以一定的频率向桶中放入令牌，桶满时丢弃令牌。

判定是否处理：当请求到来时，判断桶中的令牌是否足够，不足够时是否等待，是否丢弃。

消耗令牌：令牌足够时，减少桶中的令牌并处理请求。

## 4、使用令牌桶限流器
我们依旧可以通过channel模拟的方式来写令牌桶的算法，具体实现类似漏桶的倒置。然而golang的标准库自带的库在没有使用chan的情况下完成了高效的令牌桶算法。

### 导入库
```
	import "golang.org/x/time/rate"
```
### 创建令牌桶对象
```
	// r(float64)代表每秒向桶中放入多少令牌
	// b(int)代表 桶 的容量
	limiter := rate.NewLimiter(r,b)
```
### 令牌桶方法
```
	func (lim *limter)	Wait(ctx context.Context)(err error) //等同于WaitN(ctx，1)
	func (lim *limter)	WaitN(ctx context.Context,n int)(err error)

	func (lim *limter)	Allow()(bool) //等同于Allow(time.Now()，1)
	func (lim *limter)	AllowN(now time.Time, n int)(bool)


	func (lim *limter)	Reserve() *Reservation //等同于ReserveN(time.Now()，1)
	func (lim *limter)	ReserveN(now time.Time,n int) *Reservation

```
**WaitN**方法会在令牌数量满足时返回，当不满足时会阻塞一段时间，传入的ctx参数可以设定最长等待时间。

**AllowN**方法则会判断截止now时刻，桶中的令牌是否足够n，足够时则消费令牌返回true，不足时直接返回false，不消费令牌。

**ReserveN**方法返回一个结构体指针用于描述针对时刻now和需求令牌n的反馈，其Delay()方法返回需等待时间。Cancel方法会将获取的令牌返回。

## 5、time/rate 源码分析
先介绍一下Limter结构体
```
type Limiter struct {
	mu     sync.Mutex
	limit  Limit
	burst  int
	tokens float64
	// last is the last time the limiter's tokens field was updated
	last time.Time
	// lastEvent is the latest time of a rate-limited event (past or future)
	lastEvent time.Time
}
```	
从结构体中，我们可以很清楚的发现，在令牌桶的代码中没有chan的变量。

源码中采用时间参与计算的方式来模拟令牌桶。
* mu Mutex用于在重新设定limit和burst时保证异步安全
* limit 初始化时传入，表示放入令牌的频率
* burst 令牌桶容量，允许突发高流量请求
* last 上一次令牌数量更新的时间
* lastEvent 上一次请求事件的时间 

下面我们另一个重要的结构**Reservation**
```
type Reservation struct {
	ok        bool
	lim       *Limiter
	tokens    int
	timeToAct time.Time
	// This is the Limit at reservation time, it can change later.
	limit Limit
}	
func (lim *Limiter) reserveN(now time.Time,n int, maxFutureReserve time.Duration) Reservation {...}
```
Reservation描述了一次请求的结果（消耗令牌，取消时返还）
* ok 表示令牌是否满足
* lim Limiter指针，用于取消时归还令牌
* tokens 模拟时令牌桶内的令牌数量
* timeToAct 模拟的时间
* limit 模拟时的令牌放入频率

**reserveN**函数是对当前请求的一次模拟，**Reservationf**反映模拟的结果。常用的六个方法Wait/WaitN，Allow/AllowN，Reserve/ReserveN都通过调用reserveN函数进行模拟，然后各自处理模拟结果。
* Allow/AllowN直接返回ok值。
* Reserve/ReserveN直接返回此模拟结果。
* Allow/AllowN通过Reserve提供的信息进行阻塞，并在超时时调用Cancel返还令牌。

# 总结
相比于漏桶，令牌桶最大的优势在于可以接受一定量的流量洪峰(根据桶的大小)，其原理在于当请求速率小于预设值时，令牌桶积累令牌，当请求速率大于预设值时，令牌桶额外消耗令牌。

对于限流器的实现，标准库采用数学模拟的方式来避免chan和Goroutine的肆意创建运行，这样可以避免限流器的滥用对协程管理以及GC所带来的压力。

# 缺点
无论是漏桶还是令牌桶，说到底都是单机限流器，其缺点在于创建限流器的时候要设定阈值，在微服务集群中很难预见性的去给予合适的阈值来保证服务的稳定。

# 实际应用
在使用令牌桶限流器时，动态调整令牌桶的放入频率和桶的大小，其依据可以是令牌桶在运行时的被访问频率，也可以是[CPU的使用率](https://gocn.vip/topics/12183)等等。

# 参考资料

- [Golang限流器time/rate使用介绍](https://zhuanlan.zhihu.com/p/89820414)