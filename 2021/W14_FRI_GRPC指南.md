# grpc全指南

[TOC]

## 什么是grpc

**Grpc** 看到百度上口水话极多，大多都是互相抄袭，今儿我直接教大家先用起来，你们自己再去看文档关于grpc的其他特性和玩法。

go-rpc简写就是grpc，是go语言实现的rpc远程过程调用框架，http协议也是RPC的一种实现，grpc是一个高性能、开源和拥有统一规定的RPC框架，面向对象的http/2通信协议，能够能节省空间和IO密集度的开销。

## grpc使用场景

既然有http通信协议，有json这种易读的传输格式，为啥还需要grpc呢？首先，grpc可以建立长连接，且不同于websocket，grpc是不依赖于浏览器的客户端与服务端来通信，多路复用让大并发都在一个流上跑。二进制格式传输，虽然人不可读但对于机器来说十分友好，header头的压缩，传输效率大幅提高。从以上特点就可以看出，grpc用于高效传输，减少多次握手与挥手的开销，当然grpc的好处不限上述内容，还不会用grpc，知道那么多好处有什么用呢，如果想了解更多grpc的好处，请看官方文档 https://grpc.io/docs/

## grpc的模式

4种模式使用与场景：

**1、简单模式** 客户端发起一次请求，服务端响应一次请求，标准RPC通信

**2、客户端数据流** 客户端发送多次请求给服务端，服务端在客户端发送完毕后，服务端响应一次；如搜集用户信息，客户端向服务端多次发送数据，服务端做处理并返回一个最终的结果，减少客户端多次连接服务端的开销。

**3、服务端数据流** 客户端发起一次请求，服务端不断的返回连续的数据流；比如我司项目中需要不断获取某区域点位变化情况，从而实时展现给用户数据，这时候客户端发送一次区域信息，服务端不断返回数据

**4、双向数据流** 这时候客户端和服务端都建立起长连接，同时互相发送数据，实现实时交互。

## 如何安装

跟着我写的做，我不信你跑不起来GRPC：

我以官方截至2021年3月31日最新内容给大家做展示例子：

因为grpc是基于IDL文件定义服务，所以需要用proto3工具来生成咱们go语言用的代码，所以首先下载工具protoc

**（1）、下载protoc ** https://github.com/protocolbuffers/protobuf/releases 下载自己对应系统的可执行文件，我是windows的，所以下载[protoc-3.15.6-win64.zip](https://github.com/protocolbuffers/protobuf/releases/download/v3.15.6/protoc-3.15.6-win64.zip)

**（2）、通过go get获取 protoc-gen-go与 protoc-gen-go-grpc** 

```go
go get google.golang.org/protobuf/cmd/protoc-gen-go
go get google.golang.org/grpc/cmd/protoc-gen-go-grpc
```

以上三个文件 protoc.exe protoc-gen-go.exe protoc-gen-go-grpc.exe 三个文件均放入go的bin目录下，并把bin目录加入系统环境变量

之后稍微验证一下：

```cmd
PS C:\Users\anyanfei\Desktop> protoc --version
libprotoc 3.15.6
```

## 定义proto文件

由于我个人特别喜欢谢大的beego框架，所以当前目录以beego2.0作为基础，项目就叫beego20了：

beego20

​	|_conf

​	|_controllers

​	|_grpc

​		|_protoFile

​			|_Chat.proto

​		|_service

​			|_Chat.pb.go

​			|_Chat_grpc.pb.go

​	|_models

​	|_routers

​	|_tests

​	go.mod

​	main.go

从目录中可以看到，我自己创建了一个名为grpc的文件夹，然后在这下面创建了一个叫protoFile和service的两个文件夹，其中我们先在protoFile的根目录下创建一个名为Chat.proto文件，里面的内容是这样的，我会一段一段的做注释：

```protobuf
syntax = "proto3"; // 使用protocol buffer proto3 语法

package chat; //定义一个包名
option go_package = "../service"; //定义需要自动生成的go文件 我现在想放在service文件夹下

//请求数据 Request格式定义 结构数据定义从1开始，如果是枚举类型，字段要从0开始
message Request {
  string name = 1;
}
//响应数据 Response格式定义 同上
message Response{
  string message = 1;
}

/*
定义一个服务名:Chat
其中只有名为AutoResponse的一个RPC服务，当然你也可以定义多个，可以再在这里面写一个rpc AutoResponse2(Request) returns (Response){}
输入是Request格式的内容，输出是Response格式的内容
 */
service Chat{
  rpc AutoResponse(Request) returns (Response){}
}
```

以上就定义了一个简单模式，如果要定义个双向数据流模式，可以更改服务内的代码为：（当然第一个例子我们还是以简单模式来做）

```protobuf
  rpc AutoResponse(stream Request) returns (stream Response){}
```

定义好了文件后，就需要开始生成我们所需要的代码：

**就在上面创建Chat.proto 文件目录下执行以下代码**

```
protoc --go_out=. ./Chat.proto
protoc --go-grpc_out=. ./Chat.proto
```

**我解释一下上面两句话分别做了什么事儿：**

```protobuf
protoc --go_out=. ./Chat.proto 执行后会产生名为Chat.pb.go文件，这个文件里面包含用于填充、序列化 和 检索请求和响应消息类型的所有协议的代码。
protoc --go-grpc_out=. ./Chat.proto 执行后会产生名为Chat_grp.pb.go文件，这个文件中包含的内容有：
1、客户端使用Chat服务中定义的方法(AutoResponse)调用的接口类型，我们只需要调用这个方法即可。
2、服务端要实现的接口类型（AutoResponse），我们需要实现这个方法。
```

我知道大家开始发懵了，别急，接下来，我会慢慢用代码来实现，并逐行解释

## 简单模式代码实现

我们在Chat_grpc.pb.go文件中找到 下面接口代码的所在位置，其中ChatServer是我们在proto文件中定义的service Chat

```go

type ChatServer interface {
	AutoResponse(context.Context, *Request) (*Response, error)
	mustEmbedUnimplementedChatServer()
}

```

要实现一个接口，在golang中只需要实现他所有的方法即可，为了方便展示，我将方法写在了main.go里：

```go
package main

import (
	pb "beego20/grpc/service" //这里是我们引入生成文件的包，并取了一个叫pb的别名
	_ "beego20/routers"
	"context"
	beego "github.com/beego/beego/v2/server/web"
	"google.golang.org/grpc" //这里是引入grpc框架库
	"log"
	"net"
)

type server struct{ //创建一个结构体
	pb.UnimplementedChatServer //实现接口ChatServer interface的方法
}

func(s *server) AutoResponse(ctx context.Context, req *pb.Request) (*pb.Response, error){ //实现服务端收到客户端请求后的应答方法
	log.Println("客户端传来：",req.GetName()) 
	return &pb.Response{Message: "服务端应答---->客户端传来的内容：" + req.GetName()},nil //返回给客户端的应答内容 req.GetName()就是客户端发来的名字，也就是我们在proto文件中定义Request中的 string name = 1;
}

func listenGRPC(){ //创建一个监听GRPC的方法
	lis , err := net.Listen("tcp",":8001") //监听一个tcp服务，端口为8001
	if err !=nil{
		log.Fatalln("监听端口失败：",err)
	}
	s := grpc.NewServer() // NewServer创建一个gRPC服务，虽然还未注册，但已经开始接受请求了。返回一个用于服务RPC请求的gRPC服务。
	pb.RegisterChatServer(s,&server{}) //将我们的服务注册到rpc里
	if err = s.Serve(lis); err !=nil{ //开始启动GRPC服务
		log.Fatalln("启动服务失败:",err)
	}
}

func main() {
	go listenGRPC() //给grpc开一个单独的协程，以便能让beego的http服务也能够正常启动
	beego.Run()
}
```

我们开始开启GRPC服务端：go run main.go

运行服务端的时候可能会遇到以下问题：

```go
# beego20/grpc/service
grpc\service\Chat_grpc.pb.go:15:11: undefined: grpc.SupportPackageIsVersion7
grpc\service\Chat_grpc.pb.go:25:5: undefined: grpc.ClientConnInterface
grpc\service\Chat_grpc.pb.go:28:23: undefined: grpc.ClientConnInterface
grpc\service\Chat_grpc.pb.go:65:27: undefined: grpc.ServiceRegistrar
```

不要看百度的回答，什么给protoc-gen-go降级，官方都建议我们升级最新的，以便获得更高效更稳定的体验，所以我们直接进undefined: grpc.SupportPackageIsVersion7查看：

```go
// This is a compile-time assertion to ensure that this generated file
// is compatible with the grpc package it is being compiled against.
// Requires gRPC-Go v1.32.0 or later. //这里专门说了，需要GRPC-GO在1.32.0版本及其以上，官方早就预料到可能go-grpc版本太低
const _ = grpc.SupportPackageIsVersion7
```

我们现在去go.mod中修改我们的grpc版本

```go
module beego20

go 1.16

require github.com/beego/beego/v2 v2.0.1

require (
	github.com/golang/protobuf v1.5.2 // indirect
	github.com/smartystreets/goconvey v1.6.4
	google.golang.org/grpc v1.26.0 
	google.golang.org/protobuf v1.26.0
)

```

**我发现我的grpc版本才v1.26.0，那么将它改为v1.32.0  👆👆👆，之后再运行自动解决依赖 go mod tidy**  

再次启动服务端，正常开启grpc服务与beego的http服务：

```go
2021/04/02 13:57:44.494 [I] [parser.go:85]  E:\code\go\src\beego20\controllers no changed

2021/04/02 13:57:44.515 [I] [server.go:241]  http server Running on http://:8080
```

至此，服务端启动完毕，开始写简单模式下客户端的代码：

我在grpc文件夹下创建了一个叫client的文件夹，然后在client文件夹下创建了一个main.go文件作为例子：

```go
package main

import (
	pb "beego20/grpc/service" //和服务端一样，引入生成文件的包，还是取了一个叫pb的别名
	"context"
	"google.golang.org/grpc" //客户端也同样需要grpc框架
	"log"
	"os"
	"time"
)

func main(){
	conn , err := grpc.Dial(":8001",grpc.WithInsecure(),grpc.WithBlock()) // 用grpc去连接服务端开放的8001端口，后面两个参数grpc.WithInsecure()设置后不需要传入证书，grpc.WithBlock()参数让客户端进入连接状态返回连接套接字
    //这里是需要证书的情况：
    //tlsInsecure, _ := credentials.NewClientTLSFromFile("xxx.crt", “服务器名”)
    //conn , err := grpc.Dial(":8001",grpc.WithTransportCredentials(tlsInsecure))
	if err !=nil{
		log.Fatalln("客户端连接不到",err)
	}
	defer conn.Close()
	c := pb.NewChatClient(conn) //调用方法创建一个客户端连接
	ctx , cancel := context.WithTimeout(context.Background(),time.Second) //设置上下文超时取消
	defer cancel()
	resp , err := c.AutoResponse(ctx,&pb.Request{ //开始向服务端发送数据，数据来源是cmd命令行输入的内容
		Name: os.Args[1],
	})
	if err !=nil{
		log.Fatalln("没有得到响应",err)
	}
	log.Println(resp.GetMessage())
}
```

## 简单模式运行效果

刚才咱们已经启动了服务端，现在开始启动客户端，并传入我的名字，客户端用os.Args[1]简单获取，在cmd命令行下输入：

```cmd
E:\code\go\src\beego20\grpc\client>go run main.go anyanfei
2021/04/02 14:14:47 服务端应答---->客户端传来的内容：anyanfei
```

服务端则会显示：

```
2021/04/02 14:14:42.553 [I] [parser.go:85]  E:\code\go\src\beego20\controllers no changed

2021/04/02 14:14:42.579 [I] [server.go:241]  http server Running on http://:8080

2021/04/02 14:14:47 客户端传来： anyanfei
```

你已经学会简单模式下的GRPC使用了，实际上写多了你就会自己封装一个连接方法了，就那么个套路而已

## 双向数据流代码实现

接下来，我要变形啦~直接来一个双向数据流模式的客户端与服务端代码，自己对着打一次，运行一下，就明白了

AutoRobot.proto文件：

```protobuf
syntax = "proto3"; // 使用protocol buffer proto3 语法

package robot;//包名
option go_package = "../service";

//请求数据 Request格式定义
message Request {
  string input = 1;
}
//响应数据 Response格式定义
message Response{
  string output = 1;
}

/*
服务名:RobotChat
其中只有名为AutoResponse的一个RPC服务，
输入是Request格式的数据流，输出是Response格式的数据流
 */
service RobotChat{
  rpc ChatRpc(stream Request) returns (stream Response){}
}
```

服务端实现：

```go
package main

import (
	pb "beego20/grpc/service"
	_ "beego20/routers"
	beego "github.com/beego/beego/v2/server/web"
	"google.golang.org/grpc"
	"io"
	"log"
	"net"
)

type server struct{
	pb.UnimplementedRobotChatServer
}

func (s server) ChatRpc(serverStream pb.RobotChat_ChatRpcServer) error{
	ctx := serverStream.Context()
	for{
		select {
		case <- ctx.Done():
			log.Println("收到客户端断开连接的信号")
			return ctx.Err()
		default:
			req, err := serverStream.Recv()
			if err == io.EOF{
				log.Println("客户端发出数据流结束")
				return nil
			}
			if err !=nil{
				log.Println("服务端接收错误",err)
				return err
			}
			switch req.Input {
			case "Exit":
				log.Println("服务端收到了'Exit'指令，客户端已退出")
				if err = serverStream.Send(&pb.Response{
					Output: "服务端收到了您的退出指令",
				});err !=nil{
					log.Println("返回给客户端数据出错",err)
					return err
				}
				return nil
			default:
				if err = serverStream.Send(&pb.Response{
					Output: "【服务端返回】：你好，" + req.Input,
				});err !=nil{
					log.Println("返回给客户端数据出错",err)
					return err
				}
			}
		}
	}
}

func listenGRPC(){ //创建一个监听GRPC的方法
	lis , err := net.Listen("tcp",":8001") //监听一个tcp服务，端口为8001
	if err !=nil{
		log.Fatalln("监听端口失败：",err)
	}
	s := grpc.NewServer() // NewServer创建一个gRPC服务，虽然还未注册，但已经开始接受请求了。返回一个用于服务RPC请求的gRPC服务。
	pb.RegisterRobotChatServer(s,&server{}) //将我们的服务注册到rpc里
	if err = s.Serve(lis); err !=nil{ //开始启动GRPC服务
		log.Fatalln("启动服务失败:",err)
	}
}

func main() {
	go listenGRPC()
	beego.Run()
}
```

客户端实现：

```go
package main

import (
	pb "beego20/grpc/service"
	"bufio"
	"context"
	"fmt"
	"google.golang.org/grpc"
	"io"
	"log"
	"os"
)

func main(){
	conn, err := grpc.Dial(":8001",grpc.WithInsecure(),grpc.WithBlock())
	if err !=nil{
		log.Println("连接客户端失败",err)
		return
	}
	defer conn.Close()
	c := pb.NewRobotChatClient(conn)
	clientStream , err := c.ChatRpc(context.Background())
	if err !=nil{
		log.Println("创建客户端数据流失败",err)
		return
	}
	go func() {
		fmt.Println("请输入信息：")
		reader := bufio.NewReader(os.Stdin)
		for  {
			cmdString , _ , _ := reader.ReadLine()
			if err = clientStream.Send(&pb.Request{
				Input: string(cmdString),
			});err !=nil{
				log.Println("发送给服务端发生错误")
				os.Exit(1)
			}
		}
	}()
	for  {
		resp , err := clientStream.Recv()
		if err !=nil{
			log.Println("服务端已退出",err)
			break
		}
		if err == io.EOF{
			log.Println("收到服务端结束信号")
			break
		}
		fmt.Println("【客户端收到响应】:",resp.Output)
	}
}

```

## 双向数据流运行效果

直接运行服务端，然后运行客户端 go run main.go

客户端内容展示：

```cmd
PS E:\code\go\src\beego20\grpc\client> go run .\main.go
请输入信息：
anyanfei
【客户端收到响应】: 【服务端返回】：你好，anyanfei
我叫春卷虎
【客户端收到响应】: 【服务端返回】：你好，我叫春卷虎
Exit
【客户端收到响应】: 服务端收到了您的退出指令
2021/04/02 16:24:40 服务端已退出 EOF
```

服务端内容展示：

```cmd
2021/04/02 16:23:18.245 [I] [parser.go:85]  E:\code\go\src\beego20\controllers no changed

2021/04/02 16:23:18.266 [I] [server.go:241]  http server Running on http://:8080

2021/04/02 16:24:40 服务端收到了'Exit'指令，客户端已退出
```

## 参考文档：

- GRPC官方文档: https://grpc.io/docs  GRPC GO语言快速入门： https://grpc.io/docs/languages/go/quickstart/
- protobuf ： https://gocn.vip/topics/11840
