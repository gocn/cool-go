# errgroup 并发小工具

## 背景：微服务中的并发请求
并发编程是Golang语言的强大特性之一。在微服务架构中，面对用户的请求，我们常常需要向下游请求大量的数据继而组装成所需数据，不同的数据很可能会由不同的服务提供，这里一一请求显然是效率十分低效的，所以并发成为提高响应效率的优选方法。

## errgroup库
**基础版本**
```go
$ go get -u golang.org/x/sync
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
* Wait函数在所有goroutine运行结束才会返回，返回值记录了第一个发生的错误
* WithContext函数的第二返回值为ctx,Group会在goroutine发生错误时调用与ctx对应的cancel函数，所以ctx不适合作为其他调用的参数。

### 加强版本

