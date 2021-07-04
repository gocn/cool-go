# 使用 tableflip 实现应用的优雅热升级

## 推荐 tableflip 的背景

在日常研发过程中，我们负责的 web 应用常常会因发布过程中的服务重启而出现短时间的服务不可用或大量请求报错。随着互联网行业研发模式的逐渐敏捷和迭代周期的不断缩短，应用升级导致的服务抖动对系统稳定性的影响已不可忽视。在应用中集成 `tableflip` 或许可以缓解大家在新功能上线时的担忧。

[**`tableflip`**](https://github.com/cloudflare/tableflip) 是 [Cloudflare](https://www.cloudflare.com/) 针对 golang 进程实现优雅重启而设计的一套开源类库，集成 `tableflip` 可以让我们的 go 应用获得与 nginx reload 一样强大的热更新能力。如果你的应用尚未接入负载均衡与滚动发布，或者你的应用本身就是需要特殊处理的有状态应用，赶快试试 `tableflip` 吧！

## tableflip 简介

`tableflip` 的设计宗旨就是实现类似 nginx 的优雅热更新能力，包括：

* 新进程启动成功后，老进程不会有资源残留
* 优雅的新进程初始化（新进程启动和初始化的过程中服务不会中断）
* 容忍新进程初始化的失败（如果新进程初始化失败，老进程会继续工作而不是退出）
* 同一时间只能有一个更新动作执行

`tableflip` 中的核心类型是 `Upgrader`，调用 `Upgrader.Upgrade` 会产生一个继承必要的 `net.Listeners` 的新进程，并等待新进程发出表明其已成功完成初始化、退出或超时的信号。如果当前已有升级的任务在执行，则直接返回相应的错误。

当新进程启动成功后，调用 `Upgrader.Ready` 会清除无效的 fd 并向父进程发出初始化成功完成的信号，然后父进程就可以安心退出。至此，我们就完成了一次优雅的进程重启。

![tableflip 状态流转图](https://gocn.oss-cn-shanghai.aliyuncs.com/photo/bupt_xingxin/0a4ad13a-7025-42ce-b763-ef08113a3c2a.png?x-oss-process=image%2Fresize%2Cw_1920)

> 注：`tableflip` 目前只适用于 Linux 和 macOS

## tableflip 应用举例

接下来我们设计一个集成 `tableflip` 的简单 http server，完整代码如下：

```go
package main

import (
    "context"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/cloudflare/tableflip"
)

// 当前程序的版本
const version = "v0.0.1"

func main() {
    upg, err := tableflip.New(tableflip.Options{})
    if err != nil {
        panic(err)
    }
    defer upg.Stop()

    // 为了演示方便，为程序启动强行加入 1s 的延时，并在日志中附上进程 pid
    time.Sleep(time.Second)
    log.SetPrefix(fmt.Sprintf("[PID: %d] ", os.Getpid()))

    // 监听系统的 SIGHUP 信号，以此信号触发进程重启
    go func() {
        sig := make(chan os.Signal, 1)
        signal.Notify(sig, syscall.SIGHUP)
        for range sig {
            // 核心的 Upgrade 调用
            err := upg.Upgrade()
            if err != nil {
                log.Println("Upgrade failed:", err)
            }
        }
    }()

    // 注意必须使用 upg.Listen 对端口进行监听
    ln, err := upg.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalln("Can't listen:", err)
    }

    // 创建一个简单的 http server，/version 返回当前的程序版本
    mux := http.NewServeMux()
    mux.HandleFunc("/version", func(rw http.ResponseWriter, r *http.Request) {
        log.Println(version)
        rw.Write([]byte(version + "\n"))
    })
    server := http.Server{
        Handler: mux,
    }

    // 照常启动 http server
    go func() {
        err := server.Serve(ln)
        if err != http.ErrServerClosed {
            log.Println("HTTP server:", err)
        }
    }()

    if err := upg.Ready(); err != nil {
        panic(err)
    }
    <-upg.Exit()

    // 给老进程的退出设置一个 30s 的超时时间，保证老进程的退出
    time.AfterFunc(30*time.Second, func() {
        log.Println("Graceful shutdown timed out")
        os.Exit(1)
    })

    // 等待 http server 的优雅退出
    server.Shutdown(context.Background())
}
```

上面的代码实现了一个返回当前 version 的 http server，我们还在启动过程中插入了 1s 的延时来拉长进程的初始化时间，以观察升级过程中服务是否依旧可用。

编译并运行之：

```bash
go build -o demo main.go
./demo
```

使用 curl 模拟一些客户端请求（10 qps）：

```bash
while true; do curl http://localhost:8080/version; sleep 0.1; done
```

```text
...
[PID: 18939] 2021/07/04 15:02:47 v0.0.1
[PID: 18939] 2021/07/04 15:02:47 v0.0.1
[PID: 18939] 2021/07/04 15:02:47 v0.0.1
[PID: 18939] 2021/07/04 15:02:48 v0.0.1
...
```

然后，我们对应用进行了一些升级，将版本号修改为 `v0.0.2`，并重新编译程序：

```bash
go build -o demo main.go
```

最后，来试试优雅的热重启是否奏效吧！

```bash
kill -s HUP 18939
```

```text
...
[PID: 19306] 2021/07/04 15:04:57 v0.0.2
[PID: 19306] 2021/07/04 15:04:57 v0.0.2
[PID: 19306] 2021/07/04 15:04:57 v0.0.2
[PID: 19306] 2021/07/04 15:04:57 v0.0.2
...
```

可见，客户端完全不会受服务端的升级和重启的影响，我们的应用实现了优雅升级！

```bash
...
v0.0.1
v0.0.1
v0.0.2
v0.0.2
v0.0.2
...
```

## 总结

`tableflip` 是实现 go 进程优雅重启的优秀工具。因为其支持对连接进行保持和绑定，所以几乎适用于所有的 web 框架（HTTP、gRPC 等）。通过简单的配置，集成 `tableflip` 的程序也可以非常方便地被 systemd 等工具进行管控。

## 参考资料

* [https://github.com/cloudflare/tableflip](https://github.com/cloudflare/tableflip)
* [https://blog.cloudflare.com/graceful-upgrades-in-go/](https://blog.cloudflare.com/graceful-upgrades-in-go/)
* [https://github.com/fvbock/endless](https://github.com/fvbock/endless)
* [https://pkg.go.dev/github.com/astaxie/beego/grace](https://pkg.go.dev/github.com/astaxie/beego/grace)
* [https://grisha.org/blog/2014/06/03/graceful-restart-in-golang/](https://grisha.org/blog/2014/06/03/graceful-restart-in-golang/)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
