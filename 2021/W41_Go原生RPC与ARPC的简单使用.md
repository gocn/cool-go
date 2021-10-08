# Go原生RPC与ARPC的简单使用



## 什么是 RPC？

RPC叫做远程过程调用，意思是两台不同服务器上的服务，可以互相像调用函数一样调用。



## 我用HTTP API不一样能达到同样的效果吗?

其实对于新人来说，两台服务器之间的数据交互，用HTTP提供的API真的可以解决，但效率不高，延迟也高，且连接不会复用，因为大家都知道HTTP是无状态传输协议，每次传输都不知道对方是谁，因此，体现在以下方面：

- 每次要获取数据前，都会进行三次握手确认与四次挥手的过程。
- 不能建立长连接进行通信，多次请求
- 数据转换效率低，无论是使用form表单或者json传输，都不如直接传输二进制数据来得快，序列化与反序列化资源占用高



## 快速使用原生RPC

首先通讯双方都应拥有同样结构体，所以服务端与客户端都创建创建 go_rpc -----> RpcParams.go：

```go
package go_rpc

type MyWayRpc struct{ // 客户端要传输的数据，也是服务端要接收的数据
   Name string
   Age int
}

type MyWayRpcReply struct{ // 服务端要返回的数据，也是客户端想要获得的结果
   SystemInfo string
}
```



服务端编写main.go文件 ：

```go
package main

import (
   "go_rpc"
   "fmt"
   "log"
   "net"
   "net/rpc"
   "runtime"
)

// 其实按照web api中，我们应当把TakeMyWayRpc结构体和 GetSystem方法写在单独的控制器中，这里为了代码演示就写在main包下了

type TakeMyWayRpc struct{}

func (t TakeMyWayRpc) GetSystem(arg *go_rpc.MyWayRpc, result *go_rpc.MyWayRpcReply) error {
   fmt.Println("客户端发送了：",arg.Name,arg.Age)
   //返回给服务端的
   result.SystemInfo = runtime.GOOS
   return nil
}

func main() {
   tmwr := new(TakeMyWayRpc)
   err := rpc.Register(tmwr) // 注册RPC可以一次性注册多个RPC服务，可以用FOR注册多个结构体，用web api的话就是注册多个控制器
   if err !=nil{
      log.Fatalln("注册方法时出现问题：",err)
   }
   l, err := net.Listen("tcp", ":30001") // 从这里就可以看出，实际上RPC的底层也是基于TCP链接的，我们这里开放30001端口，供客户端连入
   defer l.Close()
   if err != nil {
      fmt.Println("监听失败，端口可能已经被占用")
   }
   fmt.Println("正在监听30001端口")
   for {
      var conn net.Conn
      conn, err = l.Accept()
      if err !=nil{
         log.Fatalln("创建句柄失败")
      }
      go rpc.ServeConn(conn)
   }
}
```



开始在另外一台服务器上写客户端的main.go：

```go
package main

import (
   "go_rpc"
   "log"
   "net/rpc"
)

func main() {
   client , err := rpc.Dial("tcp","服务端IP:30001")
   if err !=nil{
      log.Fatalln("这里试试用错误的东西:",err)
   }
   defer client.Close()

   var myWayRpcArg go_rpc.MyWayRpc	// 初始化客户端要发送的内容
   var myWayRpcReply go_rpc.MyWayRpcReply // 初始化服务端要返回的内容
	
   // 填客户端要发送的内容
   myWayRpcArg.Name = "安彦飞啊"
   myWayRpcArg.Age = 31

   if err = client.Call("TakeMyWayRpc.GetSystem",myWayRpcArg,&myWayRpcReply);err !=nil{ // 直接开始发送给服务端，并获得服务端的响应
      log.Fatalln("返回服务端数据错误:",err)
   }
   log.Println("返回成功，",myWayRpcReply)
}
```

## RPC演示：

![rpc_main](images\W43_Go原生RPC与ARPC的简单使用\rpc_main.gif)



## 为什么需要APRC：

很好，通过上面的例子，已经可以写出RPC的信息交互了，但如果遇到需要服务端主动给客户端发消息，客户端异步调用服务端的函数，这些就不是原生RPC能够做到的了



## 我们做一个ARPC的简单示例：

使用ARPC前应当先：

```go
go get github.com/lesismal/arpc
```

其实ARPC看上去更像是服务端定义了一个ROUTER，客户端去根据路由寻址找到了这个函数

首先还是看服务端的实现，服务端main.go如下：

```go
package main

import (
	"github.com/lesismal/arpc"
	"log"
	"runtime"
)

func main() {
	server := arpc.NewServer()
	registerHandler := server.Handler

	testStruct := new(TestArpcStruct)

	registerHandler.Handle( // 若这里有多个，就像写router
			"/TestArpcStruct.GetSystem",
		testStruct.GetSystem,
		)
	server.Run(":8888")
}

type TestArpcStruct struct{}

func (TestArpcStruct) GetSystem(ctx *arpc.Context)  {
	var str string
	if err := ctx.Bind(&str);err ==nil{
		log.Println(str)
		ctx.Write(runtime.GOOS)
	}
}
```



然后写客户端的代码：

```go
package main

import (
	"github.com/lesismal/arpc"
	"log"
	"net"
	"time"
)

func main() {
	client , err := arpc.NewClient(func() (net.Conn, error) {
		return net.DialTimeout("tcp","localhost:8888",3 * time.Second)
	})
	if err !=nil{
		panic(err)
	}
	defer client.Stop()

	req := "hello"
	resp := ""
	err  = client.Call("/TestArpcStruct.GetSystem",&req,&resp,5 * time.Second)
	if err != nil {
		log.Fatalf("Call failed: %v", err)
	} else {
		log.Printf("Call Response: \"%v\"", resp)
	}
}
```

最后也能和RPC一样，得到东西：

客户端发出hello，服务端str获得了hello并打印，且像客户端发送了当前服务器的运行系统名runtime.GOOS，客户端：Call Response : windows/linux 这样的结果



## 总结

其实ARPC还有很多其他功能，比如还可以提供给WS的调用方法等，官方压测后性能与RPC几乎持平，甚至比RPC更高。

我们在使用一项新技术之前，一定要弄明白这项新技术为什么会被发明出来，解决了什么痛点，提升了怎样的性能，努力思考实际应用场景在哪里，比如ARPC可以用到即时通讯，可以复用连接池，不用资源一直申请与释放，监控数据的实时展示等。

## 参考链接

https://github.com/lesismal/arpc

