# errgroup 并发小工具

##### 推荐编辑：不落凡尘

## 使用场景：微服务中的并发请求
并发编程是Golang语言的强大特性之一。在微服务架构中，面对用户的请求，我们常常需要向下游请求大量的数据继而组装成所需数据，不同的数据很可能会由不同的服务提供，这里一一请求显然是效率十分低效的，所以并发成为提高响应效率的优选方法。


## errgroup库
**基础版本安装**
```go
$ go get -u golang.org/x/sync/errgroup
```
**加强版本**
https://github.com/go-kratos/kratos/tree/v1.0.x/pkg/sync/errgroup

## 演变历程
### channel版本

```go
    res_ch := make(chan interface{},3)
    go func() {
        r := funA()
        res_ch <- r
    }()
    go func() {
        r := funB()
        res_ch <- r
    }()
    go func() {
        r := funC()
        res_ch <- r
    }()
    res := make([]interface{},0,3)
    for i := 0; i < 3; i++ {
        data := <- res_ch
        res = append(res,data)
    }
```
此版本运用了官方推荐的用于goroutine通信的channel结构。预计完整接收goroutine的结果。

问题1：goroutine数量控制较为繁琐

问题2：若goroutine内部发生错误，会导致接收程序阻塞，无法正常退出

### 基本版本errgroup

#### 源码
```go
    //源代码结构
    type Group struct {
    	cancel func()

	    wg sync.WaitGroup

	    errOnce sync.Once
	    err     error
    }

    func WithContext(ctx context.Context) (*Group, context.Context) {
        ctx, cancel := context.WithCancel(ctx)
        return &Group{cancel: cancel}, ctx
    }

    func (g *Group) Wait() error {
        g.wg.Wait()
        if g.cancel != nil {
            g.cancel()
        }
        return g.err
    }

    func (g *Group) Go(f func() error) {
        g.wg.Add(1)

        go func() {
            defer g.wg.Done()

            if err := f(); err != nil {
                g.errOnce.Do(func() {
                    g.err = err
                    if g.cancel != nil {
                        g.cancel()
                    }
                })
            }
        }()
    }
```
阅读源码我们可以得知，Group结构中使用sync.WaitGroup来控制goroutine的并发，成员变量err来记录运行中发生的错误，这里只记录第一次返回的错误值。


#### 使用
```go
group,ctx := errgroup.WithContent(context.Background())
urls :=[]string{
    ...
}
for _,v := range urls {
    group.Go(func()error{
        resp,err := http.Get(v)
        if err != nil {
            resp.Body.Close()
        }
        ...
        return err
    })
}
if err := g.Wait();err != nil {
    fmt.Println(err)
}
```
**一些说明**
* Wait函数在所有goroutine运行结束才会返回，返回值记录了第一个发生的错误。
* WithContext函数的第二返回值为ctx,Group会在goroutine发生错误时调用与ctx对应的cancel函数，所以ctx不适合作为其他调用的参数。

### 加强版本
下面是kratos的errgroup加强版，其针对几个问题作出的改进。

```
//基础版本
type Group struct {
	cancel func()

    wg sync.WaitGroup

    errOnce sync.Once
    err     error
}    

//kratos 版本
type Group struct {
    err     error
    wg      sync.WaitGroup
    errOnce sync.Once

    workerOnce sync.Once
    ch         chan func(ctx context.Context) error
    chs        []func(ctx context.Context) error

    ctx    context.Context
    cancel func()
}
```
我们先从结构体定义的角度来看待加强点。
* ch、chs、workerOnce用于控制goroutine的**并发数量**,在基础版的代码中我们发现在使用Go(function()error)函数的调用过程中是全开放的，即对于同时进行的goroutine数量并没有做限制。kratos在基础版本的基础上添加了一个chan控制并发数量，一个slice来缓存为并发的函数指针。
* kratos将产生的context对象缓存，并且更改了方法Go的函数签名加入了context参数，即func (g *Group) Go(f func(ctx context.Context) error)。在基础版本中，当error发生的是时候函数，仍然需要等到所有goroutine运行结束才会返回，kratos的Group可以使用成员函数ctx作为参数，从而控制全部并发的**生命周期**。

#### 控制并发数量源码分析
```
func (g *Group) Go(f func(ctx context.Context) error) {
	g.wg.Add(1)
	if g.ch != nil {
		select {
		case g.ch <- f:
		default:
			g.chs = append(g.chs, f)
		}
		return
	}
	go g.do(f)
}

func (g *Group) GOMAXPROCS(n int) {
	if n <= 0 {
		panic("errgroup: GOMAXPROCS must great than 0")
	}
	g.workerOnce.Do(func() {
		g.ch = make(chan func(context.Context) error, n)
		for i := 0; i < n; i++ {
			go func() {
				for f := range g.ch {
					g.do(f)
				}
			}()
		}
	})
}

func (g *Group) Wait() error {
	if g.ch != nil {
		for _, f := range g.chs {
			g.ch <- f
		}
	}
	g.wg.Wait()
	if g.ch != nil {
		close(g.ch) // let all receiver exit
	}
	if g.cancel != nil {
		g.cancel()
	}
	return g.err
}
```
从Go函数中我们看到，当g.ch != nil时，f函数首先尝试进入g.ch中，当g.ch满的时候存入g.chs中，这就是上面提到的，利用chan控制并发数量，利用slice作为函数指针的缓存。

GOMAXPROCE 函数初始化g.ch用于开启并发数量控制的开关。并且启动n个goroutine来消费传入的函数。

Wait函数中会不断将缓存中的函数不断压入chan中进行消费。

#### 使用案例
```
func sleep1s(context.Context) error {
	time.Sleep(time.Second)
	return nil
}   

{
    ...
    g := Group{}
    g.GOMAXPROCS(2)//开启并发控制
    g.Go(sleep1s)
    g.Go(sleep1s)
    g.Go(sleep1s)
    g.Go(sleep1s)
    g.Wait()
    ....
}
```

## 总结
errgroup 在sync.WaitGroup的功能之上添加了错误传递，以及在发生不可恢复的错误时取消整个goroutine集合的功能(返回值cancel)。

kratos的加强版errgroup从统一goroutine控制，defer错误捕获，并发数量控制等方面对errgroup进行了功能扩充，利用匿名函数的参数context.Context的参数传递从整体上控制goroutine的生命周期。

## 参考资料
https://github.com/golang/sync/blob/master/errgroup/errgroup.go

https://github.com/go-kratos/kratos/tree/v1.0.x/pkg/sync/errgrou
