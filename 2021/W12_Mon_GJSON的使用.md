# Golang | Gjson库

简介 什么是Gjson：

GJSON是一个Golang包，它提供了一种快速，简单的方法来从json格式文档中获取值。它拥有比如单行检索，用"."符号来寻找路径，迭代和解析多行json的功能。



## 个人理解

**Gjson**实际上就是一个比原生json解析更快更简单的一种工具，对于API来说，我不关心这个json格式是否有错，我只需要关心这个json里面有没有我想要的数据，快速格式化成我想要的格式。



## 安装

GO MOD模式下，执行：

```go
$ go get -u github.com/tidwall/gjson 
```



## 直接获取值

我还是会由浅入深的给大家介绍这个库的使用方法，如果我们拿到了一个json字符串的时候应该这样做：

```go
package main

import (
	"fmt"
	"github.com/tidwall/gjson"
)

func main() {
	exampleJsonString := `{
    "code":"000",
    "data":{
        "all_count":441353,
        "lists":[
            {
                "id":441353,
                "job_name":"经营日报-同步职位信息",
                "job_recall_time":"2021-03-13 15:05:04",
                "job_recall_content":"请求成功：great",
                "create_time":"2021-03-13 15:05:04"
            },
            {
                "id":441352,
                "job_name":"经营日报-Check张继学列表",
                "job_recall_time":"2021-03-13 15:05:00",
                "job_recall_content":"请求成功：OK",
                "create_time":"2021-03-13 15:05:00"
            }
        ]
    },
    "msg":"获取列表成功",
    "success":true
}`
	jsonCode := gjson.Get(exampleJsonString, "code")  //这个后面你可以继续跟想要的结果类型，不加的话会是json字符串的类型
	fmt.Println(jsonCode) // 结果 000
	jsonOneJobName := gjson.Get(exampleJsonString,"data.lists.#.job_name").Array() //比如我这里就希望返回是一个切片类型
	fmt.Println(jsonOneJobName) // 结果 [经营日报-同步职位信息 经营日报-Check张继学列表]
}
```

上面的同学开始疑问了，如果我自己写错了怎么办，或者没有那个key字段怎么办，没关系，你在获取到了后，加上自己想要的判断类型，再判断一次是否为空即可。

我都不需要定义任何结构体，用最简单的办法获取到我想要的内容

#### 路径语法的快速概述：

路径语法的快速概述，以上面json字符串为例

| 路径                  | 结果                                             | 解释                                 |
| --------------------- | ------------------------------------------------ | ------------------------------------ |
| data.lists.#          | 2                                                | 获取当前json数组的长度               |
| data.lists.1.job_name | 经营日报-Check张继学列表                         | 获取data下lists的索引为1的job_name值 |
| data.lists.#.job_name | [经营日报-同步职位信息 经营日报-Check张继学列表] | 获取data下lists下所有的job_name值    |

还有一些路径通配符，比如你有模糊查询或者想在json取值时有判断的需求，可查看官方文档：https://github.com/tidwall/gjson



## 返回函数

列举一些常用的返回函数使用

```go
package main
// ...
// ...

fmt.Println(gjson.Get(exampleJsonString,"data.lists.1.create_time").Exists()) // 查看当前路径的值是否存在 结果 true
fmt.Println(gjson.Get(exampleJsonString,"data.lists").IsArray()) //查看当前路径是否是json数组 结果 true
fmt.Println(gjson.Get(exampleJsonString,"data.lists.0").IsObject()) //查看当前路径是否是一个json对象 结果 true
gjson.Get(exampleJsonString,"data.lists.1").ForEach(func(key, value gjson.Result) bool {
		fmt.Println(value)
		return true
	}) 
//获取到路径结果后，遍历取值（其实觉得自己遍历可读性更高）
fmt.Println(gjson.Get(exampleJsonString,"data.lists.1.id").Float()) //所有标准类型都可以获取到，比如 Bool,Int,Value（这个是接口类型）,Unit,String 
```



## 直接解析bytes类型

实际上，很多时候我们拿到的JSON数据都是从API中获得，比如从http请求中获得了body，之后ioutil.ReadAll获得了[]byte类型的数据

```go
package main

import (
	"fmt"
	"github.com/tidwall/gjson"
)

func main() {
	exampleJsonByte := []byte(`{
    "code":"000",
    "data":{
        "all_count":441353,
        "lists":[
            {
                "id":441353,
                "job_name":"经营日报-同步职位信息",
                "job_recall_time":"2021-03-13 15:05:04",
                "job_recall_content":"请求成功：great",
                "create_time":"2021-03-13 15:05:04"
            },
            {
                "id":441352,
                "job_name":"经营日报-Check张继学列表",
                "job_recall_time":"2021-03-13 15:05:00",
                "job_recall_content":"请求成功：OK",
                "create_time":"2021-03-13 15:05:00"
            }
        ]
    },
    "msg":"获取列表成功",
    "success":true
}`)
	fmt.Println(gjson.GetBytes(exampleJsonByte, "data.lists.#.job_name").Array()) //好吧，结果一样 [经营日报-同步职位信息 经营日报-Check张继学列表]
}

```



## 总结

GJSON真的太简单了，可以说是小白golang解析json数据的必备良品，如果涉及到多人开发，需要用到同样的接口结构体，我建议还是老老实实的写结构体，毕竟数据模型的搭建是多人协同开发基础之一。



## 还想了解更多吗？

更多请查看：https://github.com/tidwall/gjson

欢迎加入我们GOLANG中国社区：https://gocn.vip/

