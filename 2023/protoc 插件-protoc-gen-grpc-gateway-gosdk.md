* # protoc 插件-protoc-gen-grpc-gateway-gosdk

  ## 基本介绍

  protoc-gen-grpc-gateway-gosdk 是一个 protoc 插件, 能根据 proto 文件一键生成 go http sdk 客户端代码. 通过借助 grpc-gateway 插件将 grpc 接口转化为 http 的方式, 进而可以通过本插件生成 http sdk 代码. 

  <img src="https://oss.jaronnie.com/image-20230422194716039.png" alt="image-20230422194716039" style="zoom: 67%;" />

  ## 特性

  - 一键自动生成 go 客户端代码, 无需人工维护
  - 通过设置统一网关, 支持将多个微服务的客户端整合
  - 根据 service 或者根据路由进行资源分类, 直观调用
  - 能对资源接口进行扩展, 如支持 websocket 接口以及支持扩展 http 原生框架的接口
  - 自带 http rest frame 封装, 并支持 Direct 方式调用接口
  - 能自动生成接口 fake 调用, 让单元测试更加简单

  ## 安装

  ```shell
  go install github.com/golang/protobuf/protoc-gen-go@v1.3.2
  go install github.com/jaronnie/protoc-gen-grpc-gateway-gosdk@v1.8.0
  ```

  ## 快速使用

  ### 编写 proto

  ```protobuf
  syntax = "proto3";
  option go_package = "./userpb";
  package user;
  
  import "google/api/annotations.proto";
  
  message AddUserReq {
        string name = 1;
        int32 age = 2;
  }
  
  message AddUserResp {
        int32 id = 1;
  }
  
  service user {
        rpc Add(AddUserReq) returns (AddUserResp) {
              option (google.api.http) = {
                    post: "/api/v1.0/user/add"
                    body: "*"
              };
        };
  }
  ```

  目录结构如下:

  ```shell
  $ tree proto              
  proto
  ├── google
  │   └── api
  │       ├── annotations.proto
  │       └── http.proto
  └── user.proto
  
  2 directories, 3 files
  ```

  ### 生成 httpsdk

  * 生成的 sdk 代码在服务端

  ```shell
  mkdir -p pkgsdk/pb
  protoc -I./proto --go_out=./pkgsdk/pb --grpc-gateway-gosdk_out=logtostderr=true,v=1,scopeVersion=userv1,sdkDir=pkgsdk:pkgsdk proto/user.proto
  ```

  生成的 httpsdk 目录结构如下:

  ```shell
  $ tree pkgsdk 
  pkgsdk
  ├── clientset.go
  ├── fake
  │   └── fake_clientset.go
  ├── pb
  │   └── userpb
  │       └── user.pb.go
  ├── rest
  │   ├── client.go
  │   ├── option.go
  │   └── request.go
  └── typed
      ├── direct_client.go
      ├── fake
      │   └── fake_direct_client.go
      └── userv1
          ├── fake
          │   ├── fake_user.go
          │   ├── fake_user_expansion.go
          │   └── fake_userv1_client.go
          ├── user.go
          ├── user_expansion.go
          └── userv1_client.go
  
  8 directories, 14 files
  ```

  * 生成的 httpsdk 独立 module

  ```shell
  mkdir -p modsdk/pb
  protoc -I./proto --go_out=./modsdk/pb --grpc-gateway-gosdk_out=logtostderr=true,v=1,scopeVersion=userv1,goModule=github.com/jaronnie/autosdk,goVersion=1.18:modsdk proto/user.proto
  cd modsdk
  go mod tidy
  ```

  生成的 httpsdk 目录结构如下:

  ```shell
  $ tree modsdk
  modsdk
  ├── clientset.go
  ├── fake
  │   └── fake_clientset.go
  ├── go.mod
  ├── go.sum
  ├── pb
  │   └── userpb
  │       └── user.pb.go
  ├── rest
  │   ├── client.go
  │   ├── option.go
  │   └── request.go
  └── typed
      ├── direct_client.go
      ├── fake
      │   └── fake_direct_client.go
      └── userv1
          ├── fake
          │   ├── fake_user.go
          │   ├── fake_user_expansion.go
          │   └── fake_userv1_client.go
          ├── user.go
          ├── user_expansion.go
          └── userv1_client.go
  
  8 directories, 16 files
  ```

  ## http sdk 结构剖析

  * clientset.go: 客户端集合
  * fake/fake_clientset.go: fake 客户端集合
  * pb: 使用 protoc-gen-go 插件生成的 pb 文件
  * rest: rest frame 封装
  * typed: 所有接口封装
    * typed/direct_client.go: direct 方式调用的 client
    * typed/fake/fake_direct_client.go: direct 方式调用的 fake client
    * typed/userv1: 即 user 服务 v1 接口实现
      * typed/userv/fake: 即 user 服务 v1 接口 fake 实现
        * typed/userv1/fake/fake_user.go:  userv1 服务 user 资源的 fake 实现
        * typed/userv1/fake_user_expansion.go: userv1 服务 user 资源的 fake 接口扩展定义
        * typed/userv1/fake_userv1_client.go: userv1 服务 fake user client 定义
      * typed/userv1/user.go:  userv1 服务 user 资源的实现
      * typed/userv1/user_expansion.go: userv1 服务 user 资源的接口扩展定义
      * typed/userv1/userv1_client.go: userv1 服务 user client 定义

  ## 高级使用(微服务版)

  一般而言, 都是多服务形式的, 如 A 服务, B 服务. 前端通过统一的网关打入到 A, B 服务当中. 通过该插件可生成 A, B 服务的统一 httpsdk.

  ### 编写 A 服务 proto

  ```protobuf
  syntax = "proto3";
  option go_package = "./apb";
  package a;
  
  import "google/api/annotations.proto";
  
  message AddUserReq {
        string name = 1;
        int32 age = 2;
  }
  
  message AddUserResp {
        int32 id = 1;
  }
  
  service a {
        rpc Add(AddUserReq) returns (AddUserResp) {
              option (google.api.http) = {
                    post: "/api/v1.0/user/add"
                    body: "*"
              };
        };
  }
  ```

  ### 编写 B 服务 proto

  ```protobuf
  syntax = "proto3";
  option go_package = "./bpb";
  package b;
  
  import "google/api/annotations.proto";
  
  message AddUserReq {
        string name = 1;
        int32 age = 2;
  }
  
  message AddUserResp {
        int32 id = 1;
  }
  
  service b {
        rpc Add(AddUserReq) returns (AddUserResp) {
              option (google.api.http) = {
                    post: "/api/v1.0/user/add"
                    body: "*"
              };
        };
  }
  ```

  ### 编写 env_file.yaml

  ```yaml
  scopeVersions: [av1, bv1]
  goModule: github.com/jaronnie/autosdk
  goVersion: 1.18
  ```

  ### 生成 httpsdk

  > 通过设置 gatewayPrefix 变量统一网关

  ```shell
  mkdir -p mutilmodsdk/pb
  protoc -I./proto --go_out=./mutilmodsdk/pb --grpc-gateway-gosdk_out=logtostderr=true,v=1,scopeVersion=av1,gatewayPrefix=/gateway/a,env_file=./etc/modmutilsdk.yaml:mutilmodsdk proto/a.proto
  protoc -I./proto --go_out=./mutilmodsdk/pb --grpc-gateway-gosdk_out=logtostderr=true,v=1,scopeVersion=bv1,gatewayPrefix=/gateway/b,env_file=./etc/modmutilsdk.yaml:mutilmodsdk proto/b.proto
  cd mutilmodsdk
  go mod tidy
  ```

  生成的目录结构如下:

  ```shell
  $ tree mutilmodsdk 
  mutilmodsdk
  ├── clientset.go
  ├── fake
  │   └── fake_clientset.go
  ├── go.mod
  ├── go.sum
  ├── pb
  │   ├── apb
  │   │   └── a.pb.go
  │   └── bpb
  │       └── b.pb.go
  ├── rest
  │   ├── client.go
  │   ├── option.go
  │   └── request.go
  └── typed
      ├── av1
      │   ├── av1_client.go
      │   ├── fake
      │   │   ├── fake_av1_client.go
      │   │   ├── fake_user.go
      │   │   └── fake_user_expansion.go
      │   ├── user.go
      │   └── user_expansion.go
      ├── bv1
      │   ├── bv1_client.go
      │   ├── fake
      │   │   ├── fake_bv1_client.go
      │   │   ├── fake_user.go
      │   │   └── fake_user_expansion.go
      │   ├── user.go
      │   └── user_expansion.go
      ├── direct_client.go
      └── fake
          └── fake_direct_client.go
  
  11 directories, 23 files
  ```

  ## 实战篇

  采用 go-zero 微服务框架, 编写 user 服务, 并加上 http 接口.

  完整代码示例: https://github.com/jaronnie/protoc-gen-grpc-gateway-gosdk/tree/main/examples/grpc-restful

  ```shell
  git clone github.com/jaronnie/protoc-gen-grpc-gateway-gosdk
  cd protoc-gen-grpc-gateway-gosdk/examples/grpc-restful
  go run user.go
  
  # 使用插件生成 pkgsdk
  mkdir -p pkgsdk/pb
  protoc -I./proto --go_out=./pkgsdk/pb --grpc-gateway-gosdk_out=logtostderr=true,v=1,scopeVersion=userv1,sdkDir=pkgsdk:pkgsdk proto/user.proto
  ```

  ### 使用示例

  ```go
  package main
  
  import (
  	"context"
  	"fmt"
  	"github.com/jaronnie/protoc-gen-grpc-gateway-gosdk/grpc-restful/pkgsdk"
  	"github.com/jaronnie/protoc-gen-grpc-gateway-gosdk/grpc-restful/pkgsdk/pb/userpb"
  	"github.com/jaronnie/protoc-gen-grpc-gateway-gosdk/grpc-restful/pkgsdk/rest"
  	"net/http"
  )
  
  func main() {
  	cs, err := pkgsdk.NewClientWithOptions(
  		rest.WithProtocol("http"),
  		rest.WithAddr("127.0.0.1"),
  		rest.WithPort("8081"),
  		rest.WithHeaders(http.Header{"Content-Type": []string{"application/json"}}),
  	)
  
  	if err != nil {
  		panic(err)
  	}
  
  	data, err := cs.Userv1().User().Add(context.Background(), &userpb.AddUserReq{
  		Name: "jaronnie",
  		Age:  22,
  	})
  	if err != nil {
  		panic(err)
  	}
  	fmt.Println(data)
  }
  ```

  ## 参考链接

  * [五分钟给你的 gRPC 服务加上 HTTP 接口](https://mp.weixin.qq.com/s/0v0zM9FkYSVw1iyRiQRNYw)
  * [protoc-gen-grpc-gateway-gosdk](https://protoc-gen-grpc-gateway-gosdk.jaronnie.com)
  * [grpc-restful](https://github.com/kevwan/grpc-restful)
  * [jaronnie/grpc-restful](https://github.com/jaronnie/protoc-gen-grpc-gateway-gosdk/tree/main/examples/grpc-restful)

  
