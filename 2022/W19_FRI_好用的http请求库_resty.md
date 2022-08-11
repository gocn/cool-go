# 好用的http请求库 resty

## 1.前言

开发时，如果你需要调用其他http接口服务，但是官方的http包太原始，用起来很麻烦。
[go-resty](https://github.com/go-resty/resty) 是 Go 语言的一个 HTTP client 库。它功能强大，特性丰富。它支持几乎所有的 HTTP 方法,，并提供了简单易用的 API。

## 2.特性
go-resty 有很多特性：

- 发起 GET, POST, PUT, DELETE, HEAD, PATCH, OPTIONS, etc. 请求
- 简单的链式书写
- 自动解析 JSON 和 XML 类型的文档
- 上传文件
- 重试功能
- 客户端测试功能
- Resty client
- Custom Root Certificates and Client Certificates
... ....

更多功能特性请查看：[go-resty features](https://github.com/go-resty/resty#features)

## 3.快速安装
直接get即可使用。
```go
$ go get -u github.com/go-resty/resty/v2
````

## 4.get请求举例

以获取百度首页信息为例说明：

```go
package main

import (
	"fmt"
	"log"

	"github.com/go-resty/resty/v2"
)

func main() {
	client := resty.New()

	resp, err := client.R().Get("http://baidu.com")

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Response Info:")
	fmt.Println("Status Code:  ", resp.StatusCode())
	fmt.Println("Status:       ", resp.Status())
	fmt.Println("Proto:        ", resp.Proto())
	fmt.Println("Time:         ", resp.Time())
	fmt.Println("Received At:  ", resp.ReceivedAt())
	fmt.Println("Size:         ", resp.Size())
}
```

执行，控制台输出如下：
```
$ go run main.go 
Response Info:
Status Code:   200
Status:        200 OK
Proto:         HTTP/1.1
Time:          103.103015ms
Received At:   2022-05-07 10:41:31.83543 +0800 CST m=+0.103656121
Size:          81

```

## 5.post请求举例

```go
package main

import (
	"fmt"
	"log"

	"github.com/go-resty/resty/v2"
)


func main() {
	client := resty.New()

	resp, err := client.R().
		SetHeader("Content-Type", "application/json").
		SetBody(`{"start":0, "limit":10, "is_suspect": false, "rank_flag":2}`).
		Post("https://backend-dev.btfs.io/api/v0/btfsscan/rank")

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Response Info:")
	fmt.Println("Status Code:  ", resp.StatusCode())
	fmt.Println("Status:       ", resp.Status())
	fmt.Println("Proto:        ", resp.Proto())
	fmt.Println("Time:         ", resp.Time())
	fmt.Println("Received At:  ", resp.ReceivedAt())
	fmt.Println("Size:         ", resp.Size())
	fmt.Println("Headers:      ", resp.String())
}
```

执行，控制台输出如下：
```
$ go run post.go 
Response Info:
Status Code:   200
Status:        200 OK
Proto:         HTTP/2.0
Time:          1.36234381s
Received At:   2022-05-07 11:00:42.575102 +0800 CST m=+1.363053536
Size:          433
Headers:       {"code":0,"data":[{"host_pid":"16Uiu2HAmFCAhR1eNu8egfojU596iEHQLW6gCpjZ6wW1d1dF9t7rd","work_amount":70884,"file_size":20298,"left_file_size":388999979702,"suspect_cheat":false,"reward_btt":0,"country":"CN"},{"host_pid":"16Uiu2HAm1TxyZTqGsb55CsuXuSnEa1wWkBMDtdHjm8rEyJfTxGdF","work_amount":70320,"file_size":20136,"left_file_size":388703990096,"suspect_cheat":false,"reward_btt":0,"country":"CN"}],"msg":"","now":1651892442,"total":2}

```

## 6.总结

`go-resty` 是一个非常简单，又好用的http请求库。如果你在开发时候有这方面的需求，不妨试试看，相信一定会喜欢上的！

## 参考资料

* [go-resty](https://github.com/go-resty/resty)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
