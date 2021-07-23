# 快且灵活的JSON解析器-Jsoniter

## 一、简介

### 什么是jsoniter？

jsoniter（json-iterator）是一款快且灵活的 JSON 解析器，同时提供 Java 和 Go 两个版本。从 dsljson 和 jsonparser 借鉴了大量代码。

根据官网介绍：Jsoniter 的 Golang 版本可以比标准库（encoding/json）快 6 倍之多。而且这个性能是在不使用代码生成的前提下获得的。
![benchmark](http://jsoniter.com/benchmarks/go-benchmark.png)

同时官网还提供了完整的 [性能评测报告](https://jsoniter.com/benchmark.html)
和 [性能优化](https://jsoniter.com/benchmark.html#optimization-used) 是怎么做的的详尽解释。

### 为什么用jsoniter？

这是一个悲伤的故事，在工作的时候碰到了上游接口返回的json包括了两个时间格式（时间戳和字符串），分别用"createtime"和"createTime"
作为key。众所周知Go的json是大小写不敏感的，而这两个字段类型又不同。当我同时解析这两个字段的时候是可以的，但是当我只解析其中一个字段时就会产生error。当时妥协选择了两个字段同时解析，后来又选择了更合适的jsoniter来控制大小写敏感。

## 二、原生json演示

通过下面的例子我们可以复现我当时出现的问题。 只解析createtime

```go
package main

import (
	"encoding/json"
	"fmt"
)

const jsonText = "{\"createtime\":\"2021-04-03T17:41:48\",\"createTime\":1617442908}\n"

func main() {
	type co struct {
		Createtime string `json:"createtime"`
		//CreateTime int    `json:"createTime"`
	}
	var data co
	text := []byte(jsonText)
	err := json.Unmarshal(text, &data)
	fmt.Println(err)
	fmt.Printf("%+v", data)
}
```

打印结果为：

```
json: cannot unmarshal number into Go struct field co.createtime of type string
{Createtime:2021-04-03T17:41:48}
```

只解析createTime

```go
type co struct {
    //Createtime string `json:"createtime"`
    CreateTime int `json:"createTime"`
}
```

打印结果为：

```
json: cannot unmarshal string into Go struct field co.createTime of type int
{CreateTime:1617442908}
```

两个都解析

```go
type co struct {
    Createtime string `json:"createtime"`
    CreateTime int    `json:"createTime"`
}
```

打印结果为：

```
<nil>
{Createtime:2021-04-03T17:41:48 CreateTime:1617442908}
```

## 三、jsoniter演示

### 默认不区分大小写

只解析createtime

```go
package main

import (
	"fmt"

	jsoniter "github.com/json-iterator/go"
)

const jsonText = "{\"createtime\":\"2021-04-03T17:41:48\",\"createTime\":1617442908}\n"

func main() {
	type co struct {
		Createtime string `json:"createtime"`
		//CreateTime int    `json:"createTime"`
	}
	var data co
	text := []byte(jsonText)
	err := jsoniter.Unmarshal(text, &data)
	fmt.Println(err)
	fmt.Printf("%+v", data)
}
```

打印结果为：

```
main.co.Createtime: ReadString: expects " or n, but found 1, error found in #10 byte of ...|ateTime":1617442908}|..., bigger context ...|{"createtime":"2021-04-03T17:41:48","createTime":1617442908}
|...
{Createtime:}
```

由此我们还可以看出json虽然大小写不敏感，只解析其中一个字段时也会报错，但是可以解析正确的结果，jsoniter默认是无法解析的。

### 设置为区分大小写
```go
package main

import (
	"fmt"

	jsoniter "github.com/json-iterator/go"
)

const jsonText = "{\"createtime\":\"2021-04-03T17:41:48\",\"createTime\":1617442908}\n"

func main() {
	type co struct {
		Createtime string `json:"createtime"`
		//CreateTime int    `json:"createTime"`
	}
	var data co
	text := []byte(jsonText)
	json := jsoniter.Config{CaseSensitive: true}.Froze()
	err := json.Unmarshal(text, &data)
	fmt.Println(err)
	fmt.Printf("%+v", data)
}
```

打印结果为：

```
<nil>
{Createtime:2021-04-03T17:41:48}
```

最后总结一下区别

|    解析器   | 解析字段 | 结果 |  err == nil   |
| ---------- | --- | --- | ---   |
| json  |   createtime   | {Createtime:2021-04-03T17:41:48} | false |
| json  |   createTime   | {CreateTime:1617442908} | false |
| json  |   createtime、createTime   | {Createtime:2021-04-03T17:41:48 CreateTime:1617442908}| true |
| jsoniter  |   createtime   | {Createtime:} | false |
| jsoniter  |   createTime   | {CreateTime:0} | false |
| jsoniter  |   createtime、createTime   |{Createtime:2021-04-03T17:41:48 CreateTime:1617442908}| true |
| jsoniter <br>(区分大小写)|   createtime   | {Createtime:2021-04-03T17:41:48} | true |
| jsoniter <br>(区分大小写)|   createTime   | {CreateTime:1617442908} | true |
| jsoniter <br>(区分大小写)|   createtime、createTime   |{Createtime:2021-04-03T17:41:48 CreateTime:1617442908}| true |

## 四、jsoniter用法

除了像上面这种jsoniter自定义的特性，我们还可以将jsoniter替代encoding/json使用


将

```go
import "encoding/json"
json.Marshal(&data)
```

替换为

```go
import jsoniter "github.com/json-iterator/go"

var json = jsoniter.ConfigCompatibleWithStandardLibrary
json.Marshal(&data)
```

将

```go
import "encoding/json"
json.Unmarshal(input, &data)
```

替换为

```go
import jsoniter "github.com/json-iterator/go"

var json = jsoniter.ConfigCompatibleWithStandardLibrary
json.Unmarshal(input, &data)
```

[更多用法点此查看](http://jsoniter.com/migrate-from-go-std.html)


## 参考文档：

- https://jsoniter.com/index.cn.html
- https://github.com/json-iterator/go
- http://jsoniter.com/migrate-from-go-std.html

欢迎加入我们GOLANG中国社区：https://gocn.vip/
