# 对开发者友好的Gorm生成工具

## **推荐背景**

操作数据库是大多数服务端程序员必不可少的工作，而 Gorm 作为一个拥有 25k star 的项目已经是go语言操作关系型数据库的首选了，由于 Gorm 中提供了很多 interface{} 形式的参数，这让程序员很容易误用，导致线上项目存在SQL注入的风险。

对于这种情况，可以尝试使用 [gorm.io/gen](https://github.com/go-gorm/gen) 生成 Gorm 模型，这样就可以很好的规避SQL注入的风险，并且由于 gen 是 Gorm 作者开发的生成工具，可以完美的兼容 Gorm 的操作。

### gen 是什么

基于 [Gorm](https://github.com/go-gorm/gorm) 的代码生成器，旨在对开发员友好。

> The code generator base on GORM, aims to be developer friendly.
> 

[gorm.io/gen](https://github.com/go-gorm/gen) 提供以下的代码生成功能

- 自动将数据库表结构映射成GORM模型
- CRUD 代码生成
- 自定义（通过注释或模版语法）生成查询方法
- 事物、嵌套事物、保存点、回滚到保存点
- 与GORM完全兼容

## 快速使用

### 安装

```go
go get -u gorm.io/gorm
go get -u gorm.io/gen
// 安装对应数据库的 driver
// sqlite driver
// go get -u gorm.io/driver/sqlite
// mysql driver
// go get -u gorm.io/driver/mysql
```

### 入门案例

功能描述：有一个 `user` 表，包含用户名，密码，昵称，性别，头像等字段，需要提供如下功能：

1. 对用户的 CURD 操作
2. 查询所以男性用户
3. 更新用户的昵称

按照官方给的最佳实践将项目目录分成如下

```go
demo
├── cmd
│   └── generate
│       └── generate.go # 用于执行代码生成的
├── dal
│   ├── dal.go          # 创建数据库连接
│   ├── model
│   │   ├── method.go   # 定制自己的 query 方法
│   │   └── model.go    # 存储数据库 ORM 模型
│   └── query           # 生成的目录
|       ├── user.gen.go # generated code for user
│       └── gen.go      # generated code
├── biz
│   └── query.go        # 业务代码，用来调用生成的代码
├── config
│   └── config.go       # 数据库连接的 DSN 配置
├── generate.sh         # 用来执行生成代码命令的脚本
├── go.mod
├── go.sum
└── main.go
```

1. 在 `dal/model/method.go` 中可以采用占位符和模版的形式定义自己的方法

```go
package model

import "gorm.io/gen"

type UserMethod interface {
	//where(id=@id)
	FindByID(id int) (gen.T, error) // 通过 id 查找

	//select * from users where gender=0
	QueryMan() ([]gen.T, error) // 查找所有男性

	//sql(select * from users)
	FindAll() ([]gen.M, error) // 查找全部

	//update user
	//	{{set}}
	//		update_time=now(),
	//		{{if nickname != ""}}
	//			nickname=@nickname
	//		{{end}}
	//	{{end}}
	// where id=@id
	UpdateNickname(id int64, nickname string) error // 更新昵称
}
```

2. 在 `config/config.go` 中定义数据库连接 DSN

```go
package config

const MysqlDSN = "root:669988@(localhost:3306)/demo?charset=utf8mb4&parseTime=True&loc=Local"
```

3. 在 `dal/dal.go` 中创建连接

```go
package dal

import (
	"github.com/gorm_gen_learn/config"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"sync"
)

var (
	db *gorm.DB
	once sync.Once
)

func GetDBConnect()(conn *gorm.DB){
	once.Do(func() {
		var err error
		db,err = gorm.Open(mysql.Open(config.MysqlDSN))
		if err != nil {
			panic(err)
		}
	})
	return db
}
```

4. 在 `cmd/generate/generate.go` 中注册模型生成

```go
package main

import (
	"github.com/gorm_gen_learn/dal"
	"github.com/gorm_gen_learn/dal/model"
	"gorm.io/gen"
)

func main(){
	g := gen.NewGenerator(gen.Config{
		OutPath: "../../dal/query", // 生成代码的输出路径
	})
	g.UseDB(dal.GetDBConnect())
	g.ApplyBasic(g.GenerateModel("user")) // 绑定表结构
	g.ApplyInterface(func(model.UserMethod) {},g.GenerateModelAs("user","User"))
	g.Execute()
}
```

5. 在 `generate.sh]` 中添加生成代码的脚本

```bash
GENERATE_DIR="./cmd/generate"

cd $GENERATE_DIR || exit

echo "Start Generating"
go run .
```

6. 运行 `generate.sh` 就可以在 `dal/query`  目录看到生成的代码了

7. 在 `biz/query.go` 中使用

```bash
package biz

import (
	"context"
	"github.com/gorm_gen_learn/dal"
	"github.com/gorm_gen_learn/dal/model"
	"github.com/gorm_gen_learn/dal/query"
)

// CreateUser 创建用户
func CreateUser(ctx context.Context, username , password, nickname , avatar string) error{
	manager := query.Use(dal.GetDBConnect()).User.WithContext(ctx)
	user := &model.User{
		Nickname: nickname,
		Password: password,
		Username: username,
		Avatar: avatar,
	}
	return manager.Create(user)
}

// QueryMan 查询全部男用户
func QueryMan(ctx context.Context) ([]*model.User, error){
	manager := query.Use(dal.GetDBConnect()).User.WithContext(ctx)
	return manager.QueryMan()
}

// UpdateNickName 更新用户昵称
func UpdateNickName(ctx context.Context, uid int64, nickname string) error{
	manager := query.Use(dal.GetDBConnect()).User.WithContext(ctx)
	return manager.UpdateNickname(uid, nickname)
}
```

## 总结

通过注释和模版子句的形式生成的模型和方法可以实现后期迭代时代码风格相同，不需要考虑存在SQL注入的风险，并且内置生成的 CURD 操作可以节省研发人员堆代码的时间，美中不足的就是项目创建时间较短（dome 项目使用 v0.0.23），还没有代码生成工具需要用户手动写一`generate.go` 的代码，同时配一个 `generate.sh` 的脚本。不过目前项目还在持续更新中，相信很快就有更加方便的工具了。

## 参考连接

https://github.com/go-gorm/gen