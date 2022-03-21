如果一个应用需要同时对外提供 HTTP 和 gRPC 服务，通常情况下我们会为两个服务绑定不同的监听端口，而本文要介绍的 cmux 为我们提供了一种连接多路复用的新选择，使用 cmux 可以将不同服务绑定在同一个网络端口上！

## 简介

多路复用是个很常见的概念，我们在编写 HTTP 服务时通常会用 `http.ServeMux` 或类似的三方工具实现请求与 handler 的匹配和绑定，这类多路复用一般作用在某个服务内部。而 cmux（github.com/soheilhy/cmux） 则提供了基于请求内容对连接进行多路复用的能力，使用 cmux 我们可以在同一个 TCP 监听（端口）上提供包括 gRPC、SSH、HTTPS、HTTP、Go RPC 在内的几乎任何协议。

cmux 通过扩展 `net.Listener` 的方式实现了连接多路复用能力，在接收到客户端请求后，cmux 会根据注册的规则对客户端请求进行鉴别和匹配，并根据匹配结果将请求转发给相应的服务。

## 使用举例

cmux 的使用特别简单，我们只需要为每个服务指定相应的匹配规则即可：

```go
package main

import (
	"context"
	"log"
	"net"
	"net/http"

	"google.golang.org/grpc"
	pb "google.golang.org/grpc/examples/helloworld/helloworld"

	"github.com/soheilhy/cmux"
)

func main() {
	lis, err := net.Listen("tcp", "localhost:50051")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	mux := cmux.New(lis)

	// gRPC 匹配规则
	grpcL := mux.MatchWithWriters(
		cmux.HTTP2MatchHeaderFieldSendSettings("content-type", "application/grpc"),
	)
	// otherwise serve http
	httpL := mux.Match(cmux.Any())

	// gRPC server
	grpcS := grpc.NewServer()
	pb.RegisterGreeterServer(grpcS, &server{})

	// HTTP server
	httpS := &http.Server{
		Handler: http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			w.Write([]byte("greet from HTTP\n"))
		}),
	}

	// start serving!
	go grpcS.Serve(grpcL)
	go httpS.Serve(httpL)
	log.Printf("serve both grpc and http at %v", lis.Addr())
	if err := mux.Serve(); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}

type server struct {
	pb.UnimplementedGreeterServer
}

func (*server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	return &pb.HelloReply{Message: "greet from gRPC"}, nil
}
```

先来试试 http 访问是否正常

```text
$ curl -D - http://localhost:50051
HTTP/1.1 200 OK
Date: Mon, 21 Mar 2022 08:20:15 GMT
Content-Length: 16
Content-Type: text/plain; charset=utf-8

greet from HTTP
```

再来试试 gRPC 访问

```text
$ ${GOPATH}/bin/greeter_client -addr localhost:50051
2022/03/21 16:12:11 Greeting: greet from gRPC
```

> 测试客户端安装：
> go install google.golang.org/grpc/examples/helloworld/greeter_client

如上，我们通过 50051 这个端口同时承载了 HTTP 和 gRPC 两个服务，是不是特别神奇！

## 总结

使用 cmux 我们可以轻松实现对服务端连接的多路复用，cmux 仅会在连接建立之初读取少量请求样本进行路由匹配，因此在客户端长连接场景下的性能损失几乎可以忽略不计。大家来试试吧！


## 参考资料

* [https://github.com/soheilhy/cmux](https://github.com/soheilhy/cmux)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)