# Go程序配置利器-viper库

## 推荐理由

日常开发中，程序配置项会包含多种源，如：配置文件，系统环境变量、分布式config服务等等，常规方式是每种配置源写一套逻辑，虽然开发量不大，但总要花精力去维护后续的变更。Viper库恰好能解决这类痛点，同时还支持多种配置文件格式，以及热加载能力，所以程序配置管理场景可以尝试用Viper库。

## 功能介绍

Viper具体功能特性如下：

* 设置配置项默认值
* 支持显式设置配置项
* 支持读取JSON、TOML、YAML、HCL、envfile和Java properties等配置格式
* 支持读取环境变量
* 支持读取etcd、Consul等分布式配置服务
* 支持读取命令行参数
* 支持读取内存
* 配置热加载

## 使用指南

Viper使用起来也是比较简单的，主要的代码流程大致如下：

1. 初始化viper，可以使用单个viper实例，也可以多个viper实例

2. 针对不同的配置源设置相对应的参数，如：配置文件路径，文件类型，ectd/consul服务器访问地址和key等

3. 读取配置项，可以直接使用viper.GetXXX()方法获取某个具体配置项，也可以所以或部分配置项反序列化为struct或map

以读取配置文件为例，配置文件内容：

```yaml
Hacker: true
name: steve
hobbies:
- skateboarding
- snowboarding
- go
clothing:
  jacket: leather
  trousers: denim
age: 35
eyes : brown
beard: true

```

代码：

```go
package main

import (
    "fmt"
    "log"

    "github.com/spf13/viper"
)

func main() {
    viper.SetConfigName("config") // 配置文件名称
    viper.SetConfigType("yaml") // 文件名无扩展名，需要显式指定
    viper.AddConfigPath("/etc/appname/")   // 配置文件搜索路径
    viper.AddConfigPath("$HOME/.appname")  // 多次调用以添加多个
    viper.AddConfigPath(".")               // 也可以设置为工作目录
    if err := viper.ReadInConfig(); err != nil {
        if _, ok := err.(viper.ConfigFileNotFoundError); ok {
            // Config file not found; ignore error if desired
        } else {
            // Config file was found but another error was produced
        }
        log.Fatal(err)
    }
    fmt.Println("获取配置文件的Hacker", viper.GetBool("hacker"))
    fmt.Println("获取配置文件的hobbies", viper.GetStringSlice("hobbies"))
    fmt.Println("获取配置文件的clothing.jacket", viper.GetString(`clothing.jacket`))
}
```

更详细的使用可以参考Viper[官方文档](https://pkg.go.dev/github.com/spf13/viper)

## 总结

日常开发中程序配置管理的绝大部分场景都可以使用Viper，实际使用中也可以对Viper二次封装，来支持读取更多平台的远程文件。

最后Viper v2启动了，也在收集广大使用者的需求和反馈。[传送门](https://docs.google.com/forms/d/e/1FAIpQLSeesGIS8Rya-iOXmXjUst8sT3NMSmdclBPa-aJT3HkPjDWnng/viewform)。

## 参考资料

[https://github.com/spf13/viper](https://github.com/spf13/viper)