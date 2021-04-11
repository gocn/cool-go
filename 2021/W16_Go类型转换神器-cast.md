# Go 类型转换神器 cast

## 什么是 cast？

[cast](https://github.com/spf13/cast) 用于一致且简单的方式在不同的 go 类型之间进行安全的转换。

## 为什么使用 cast?

在 Go 程序中，我们通常需要将数据由一种类型转换为另一种类型。

cast 使用一致且简单的方式来提供安全的类型转换。它不仅仅适用于类型断言，更强大的功能在于我们使用接口来处理动态数据的时候，cast 提供了一种简单的方法将接口优雅的转换为我们需要的数据类型。

使用 cast 将会极大的增加我们的开发效率，因为它本身就是为了开源项目 [Hugo](https://github.com/gohugoio/hugo) 而生。

## 使用 Go 标准库进行类型转换的痛点

在实际开发中我们往往需要对一些常用的数据类型进行转换，如 string，int，int64，float等数据类型。

标准库 strconv 提供了字符串与基本数据类型之间的转换功能，我们也可以使用`fmt.Sprintf`函数进行转换。

但是往往使用起来不够直观，且当数据类型为接口时，若需要转换成需要的数据类型较为繁琐。

## 快速使用 cast

### 安装cast

```go
go get github.com/spf13/cast
```

### 入门案例

```go
cast.ToString("gocn")         	  // "gocn"
cast.ToString(8)                  // "8"
cast.ToString(8.31)               // "8.31"
cast.ToString([]byte("gocn")) 	  // "gocn"
cast.ToString(nil)                // ""

var foo interface{} = "I love gocn"
cast.ToString(foo)                // "I love gocn"
```

```go
cast.ToInt(8)                  // 8
cast.ToInt(8.31)               // 8
cast.ToInt("8")                // 8
cast.ToInt(true)               // 1
cast.ToInt(false)              // 0

var eight interface{} = 8
cast.ToInt(eight)              // 8
cast.ToInt(nil)                // 0
```

cast 实现了多种常见类型之间的相互转换，并返回最符合直觉的结果。如：

* nil 转 string 的结果为 ""
* true 转 string 的结果为 "true"，true 转 int 的结果为 1
* interface{} 转为其他类型，要看它里面存储的值类型

这些类型包括了：

* 基本类型：整形，浮点型，布尔类型，字符串
* 空接口：interface{}
* nil
* 时间：time.Time
* 时间段：time.Duration
* 切片类型：[]Type
* map[string]Type

使用起来非常的丝滑。

## 进阶使用

cast提供了两组函数：

* toType，将参数转换为 Type 类型。若转换失败，则返回 Type 类型的零值
* ToTypeE，返回转换后的值和 error

### 时间和时间段的转换

代码如下：

```go
package main

import (
	"time"

	"github.com/spf13/cast"
)

func main() {
	timeStamp := time.Now().Unix() 		//1617975806
	cast.ToTime(timeStamp)         		//2021-04-09 21:43:26 +0800 CST
    
	timeStr := "2021-04-09 21:43:26"
	cast.ToTime(timeStr) 			   //2021-04-09 21:43:26 +0000 UTC

	duration, _ := time.ParseDuration("1m30s")
	cast.ToDuration(duration) 	  	    //1m30s

	strDuration := "90s"
	cast.ToDuration(strDuration)   		//1m30s
}
```

### 转换为切片

代码如下：

```go
package main

import (
  	"fmt"

  	"github.com/spf13/cast"
)

func main() {
  	sliceOfInt := []int{1, 3, 7}
  	arrayOfInt := [3]int{8, 12}
  	// ToIntSlice
  	cast.ToIntSlice(sliceOfInt)  // [1 3 7]
  	cast.ToIntSlice(arrayOfInt)  // [8 12 0]

  	sliceOfInterface := []interface{}{1, 2.0, "gocn"}
  	sliceOfString := []string{"I", "love", "gocn"}
  	stringFields := " I   love  gocn   "
  	any := interface{}(666)
  	// ToStringSliceE
  	cast.ToStringSlice(sliceOfInterface)  // [1 2 gocn]
  	cast.ToStringSlice(sliceOfString)     // [I love gocn]
  	cast.ToStringSlice(stringFields)      // [I love gocn]
  	cast.ToStringSlice(any)               // [666]
}
```

### 转为 map[string]Type

代码如下：

```go
package main

import (
	"github.com/spf13/cast"
)

func main() {
	m := map[interface{}]interface{}{
		"name": "gocn",
		"age":  999,
	}
	cast.ToStringMapString(m) 		//map[age:999 name:gocn]
    
	data := `{"name":"gocn", "url":"https://gocn.vip"}`
	cast.ToStringMapString(data) 	//map[name:gocn url:https://gocn.vip]
}
```

## 总结

cast 库能在几乎所有常见类型之间转换，小巧但是非常的实用。

cast 提供一致且简单的方式在各种常见的类型之间进行转换，能极大的提高开发效率。

## 参考链接

* https://github.com/spf13/cast
* https://darjun.github.io/2020/01/20/godailylib/cast



