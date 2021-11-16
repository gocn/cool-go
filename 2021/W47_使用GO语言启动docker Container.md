# 使用GO语言启动docker Container

## **推荐背景**

在日常开发中，测试是不可避免的，在针对DAO层的代码写测试用例的时候，直接将依赖的存储服务（比如mongodb）的client给mock掉，可能达不到检验代码中语法或数据操作正确性的目的。如果在本地起一个相关的存储服务又会由于不同的项目带来环境的污染，并且测试代码由于依赖本地环境可能导致多人协作困难。在云原生时代，你可能第一想到的就是利用docker container 来解决环境问题，而本文所推荐的就是用 go 语言来操作docker的开源项目。

项目链接： [https://github.com/moby/moby](https://github.com/moby/moby)

## **快速使用**

### 准备环境

安装docker： [https://www.docker.com/](https://www.docker.com/)

### 安装

```go
// 安装 docker client
go get github.com/docker/docker/client
```

### 启动mongo

```go
package main

import (
	"context"
	"fmt"
	"github.com/docker/docker/api/types"
	"github.com/docker/docker/api/types/container"
	"github.com/docker/docker/client"
	"github.com/docker/go-connections/nat"
)

const (
	mongoExposedPort = "27017/tcp"
)

func main(){
	cli, err := client.NewEnvClient() // 初始化 docker client
	if err != nil{
		panic(err)
	}
	ctx := context.Background()
	// 创建一个container
	resp, err := cli.ContainerCreate(ctx,
		&container.Config{
			Image: "mongo:latest", // 镜像名：推荐提前下好
			ExposedPorts: nat.PortSet{
				mongoExposedPort: {},   // 暴露的端口号：可以使用想用命令启动一个，然后通过docker ps 看
			},
		},
		&container.HostConfig{
			PortBindings: nat.PortMap{
				mongoExposedPort: []nat.PortBinding{ // 端口映射：将容器里的 27017 映射到本机的 27017 端口
					{
						HostIP: "127.0.0.1",
						HostPort: "0", // 这个值如果是0，就会选一个未被占用的端口
					},
				},
			},
		},
		nil, // 网络配置：默认将可以
		nil,	    // 平台描述：不用传
		"", // 容器名：传空会随机分配
	)
	if err != nil{
		panic(err)
	}
	err = cli.ContainerStart(ctx, resp.ID, types.ContainerStartOptions{}) // start container
	if err != nil {
		panic(err)
	}
	fmt.Println("container start ...")

	result, err := cli.ContainerInspect(ctx, resp.ID)
	if err != nil {
		panic(err)
	}
	addr := result.NetworkSettings.Ports[mongoExposedPort][0] // 查看绑定到的地址
	fmt.Printf("binding port: %v\n", addr)
	err = cli.ContainerRemove(ctx,resp.ID,types.ContainerRemoveOptions{ // remove container
		Force: true, // 强制删除
	})
	if err != nil{
		panic(err)
	}
	fmt.Println("container kill ...")
}
```

### 封装容器运行的库函数

```go
// 封装在容器运行mongodb的函数
package mongotest

import (
	"context"
	"fmt"
	"github.com/docker/docker/api/types"
	"github.com/docker/docker/api/types/container"
	"github.com/docker/docker/client"
	"github.com/docker/go-connections/nat"
	"testing"
)

const (
	mongoExposedPort = "27017/tcp"
)

func RunWithMongo(m *testing.M, mongoURI *string) int{
	cli, err := client.NewEnvClient() // 初始化 docker client
	if err != nil{
		panic(err)
	}
	ctx := context.Background()
	// 创建一个container
	resp, err := cli.ContainerCreate(ctx,
		&container.Config{
			Image: "mongo:latest", // 镜像名：推荐提前下好
			ExposedPorts: nat.PortSet{
				mongoExposedPort: {},   // 暴露的端口号：可以使用想用命令启动一个，然后通过docker ps 看
			},
		},
		&container.HostConfig{
			PortBindings: nat.PortMap{
				mongoExposedPort: []nat.PortBinding{ // 端口映射：将容器里的 27017 映射到本机的 27017 端口
					{
						HostIP: "127.0.0.1",
						HostPort: "0", // 这个值如果是0，就会选一个未被占用的端口
					},
				},
			},
		},
		nil, // 网络配置：默认将可以
		nil,	    // 平台描述：不用传
		"", // 容器名：传空会随机分配
	)
	if err != nil{
		panic(err)
	}
	containerID := resp.ID
	defer func() {
		err = cli.ContainerRemove(ctx,containerID,types.ContainerRemoveOptions{ // remove container
			Force: true, // 强制删除
		})
		if err != nil{
			panic(err)
		}
	}()

	err = cli.ContainerStart(ctx, containerID, types.ContainerStartOptions{}) // start container
	if err != nil {
		panic(err)
	}
	result, err := cli.ContainerInspect(ctx, resp.ID)
	if err != nil {
		panic(err)
	}
	addr := result.NetworkSettings.Ports[mongoExposedPort][0] // 查看绑定到的地址
	*mongoURI = fmt.Sprintf("mongodb://%s:%s", addr.HostIP, addr.HostPort)
	return m.Run()
}
```

```go
// 真实的测试代码
package docker_demo

import (
	mongotest "github.com/zrruirui/docker-demo/mongo"
	"os"
	"testing"
)

var mongoURI = ""

func TestMain(m *testing.M){
	os.Exit(mongotest.RunWithMongo(m, &mongoURI))
}

func TestA(t *testing.T){
	// 写自己的单测代码
	t.Log(mongoURI)
}
```

## 总结

[https://github.com/moby/moby](https://github.com/moby/moby) 这个项目中有很多对容器的操作，在我的demo项目中只用了其中的很小一部分，用来解决单测 mock db问题，其他更复杂的功能有需要可自行查阅。

## 感谢

特别感谢 **ccmouse** 老师的讲解，我是从他那里了解了这个库并在日常开发中使用。
