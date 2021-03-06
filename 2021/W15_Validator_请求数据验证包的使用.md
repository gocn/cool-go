
## **背景介绍**

<br />[validator](https://github.com/go-playground/validator) 是提供了比较强大的，可支持定制化的数据验证规则的包，它提供了很方便的方法及验证规则来验证传入的 HTTP 请求数据，可以一次定义多次使用，非常优雅地解决了数据验证杂乱问题，有利于拆解业务代码中的数据验证相关代码，这样整个控制器中的代码看起来比较整洁便于阅读。<br />

## **快速使用**
> 1. 执行 `go get github.com/go-playground/validator/v10` 以安装包。
> 1. 把包导入到需要使用的项目文件中。
> 1. 构造验证规则。


```go
package main

import (
	"fmt"
	"github.com/go-playground/validator/v10"
)

// 用户信息
type User struct {
	Name      string     `validate:"required"`
	Age       uint8      `validate:"gte=0,lte=130"`
	Email     string     `validate:"required,email"`
	Phone     int64      `validate:"required,number"`
	Addresses []*Address `validate:"required,dive,required"`
}

// 用户地址信息
type Address struct {
	Street string `validate:"required"`
	City   string `validate:"required"`
	Phone  string `validate:"required"`
}

// 用 Validate 的单个实例来缓存结构体信息
var validate *validator.Validate


func main() {
	//创建一个示例
	validate = validator.New()

	address := &Address{
		Street: "Lujiazui No.21 ",
		City:   "Shanghai",
		Phone:  "021-7777777",
	}

	user := &User{
		Name:      "Aklman",
		Age:       135,
		Email:     "aklman888@gmail.com",
		Phone:     17702177888,
		Addresses: []*Address{address},
	}
	//验证结构体
	validateStruct(user)
	//单一验证变量
	validateVariable()
}

//复杂结构数据的验证
func validateStruct(user *User) {
	err := validate.Struct(user)
	if err != nil {
		if _, ok := err.(*validator.InvalidValidationError); ok {
			fmt.Println(err)
			return
		}

		for _, err := range err.(validator.ValidationErrors) {

			fmt.Println(err.Namespace())
			fmt.Println(err.Field())
			fmt.Println(err.StructNamespace())
			fmt.Println(err.StructField())
			fmt.Println(err.Tag())
			fmt.Println(err.ActualTag())
			fmt.Println(err.Kind())
			fmt.Println(err.Type())
			fmt.Println(err.Value())
			fmt.Println(err.Param())
			fmt.Println()
		}
		return
	}

	// 执行下一步操作
}

//单一结构体的数据验证
func validateVariable() {
	myEmail := "aklman888@gmail.com"

	errs := validate.Var(myEmail, "required,email")

	if errs != nil {
		fmt.Println(errs)
		return
	}
}
```


## **基本数据验证规则符号**


### **0. 比较运算符**


| 运算符 | 运算描述 |
| :--- | :--- |
| eq | 等于 |
| gt | 大于 |
| gte | 大于等于 |
| lt | 小于 |
| lte | 小于等于 |
| ne | 不等于 |


### **1. 字段验证运算**


| 运算符 | 运算描述 |
| :--- | :--- |
| eqcsfield | 跨不同结构体字段相等 |
| eqfield | 同一结构体字段相等 |
| fieldcontains | 包含字段 |
| fieldexcludes | 未包含字段 |
| gtcsfield | 跨不同结构体字段大于 |
| gtecsfield | 跨不同结构体字段大于等于 |
| gtefield | 同一结构体字段大于等于 |
| gtfield | 同一结构体字段相等 |
| ltcsfield | 跨不同结构体字段小于 |
| ltecsfield | 跨不同结构体字段小于等于 |
| ltefield | 同一个结构体字段小于等于 |
| ltfield | 同一个结构体字段小于 |
| necsfield | 跨不同结构体字段不想等 |
| nefield | 同一个结构体字段不想等 |


### **2. 网络字段验证运算**

| 运算符 | 运算描述 |
| :--- | :--- |
| cidr | 有效 CIDR |
| cidrv4 | 有效 CIDRv4 |
| cidrv6 | 有效 CIDRv6 |
| datauri | 是否有效 URL |
| fqdn | 有效完全合格的域名 (FQDN) |
| hostname | 是否站点名称 RFC 952 |
| hostname_port | 是否站点端口 |
| ip | 是否包含有效 IP |
| ip4_addr | 是否有效 IPv4 |
| ip6_addr | 是否有效 IPv6 |
| ip_addr | 是否有效 IP |
| ipv4 | 是否有效 IPv4 |
| ipv6 | 是否有效 IPv6 |
| mac | 是否媒体有效控制有效 MAC 地址 |
| tcp4_addr | 是否有效 TCPv4 传输控制协议地址 |
| tcp6_addr | 是否有效 TCPv6 传输控制协议地址 |
| tcp_addr | 是否有效 TCP 传输控制协议地址 |
| udp4_addr | 是否有效 UDPv4 用户数据报协议地 |
| udp6_addr | 是否有效 UDPv6 用户数据报协议地 |
| udp_addr | 是否有效 UDPv 用户数据报协议地 |
| unix_addr | Unix域套接字端点地址 |
| uri | 是否包含有效的 URI |
| url | 是否包含有效的 URL |


## **3. 字符串验证运算**

| 运算符 | 运算描述 |
| :--- | :--- |
| alpha | 是否全部由字母组成的 |
| alphanum | 是否全部由数字组成的 |
| alphanumunicode | 是否全部由 unicode 字母数字组成 |
| alphaunicode | 是否全部由 unicode 字母组成 |
| ascii | ASCII |
| contains | 是否包含全部 |
| endswith | 尾部是否以此字符结束 |
| lowercase | 是否全小写字符组成 |
| multibyte | 是否多字节字符 |
| number | 是否数字 |
| numeric | 是否包含基本的数值 |
| printascii | 是否可列印的 ASCII |
| startswith | 开头是否以此字符开始 |
| uppercase | 是否全大写字符组成 |


## **4. 数据格式验证运算**


| 运算符 | 运算描述 |
| :--- | :--- |
| base64 | 是否 Base64 |
| base64url | 是否 Base64UR |
| btc_addr | 是否 Bitcoin 地址 |
| btc_addr_bech32 | 是否 Bitcoin Bech32 地址 |
| datetime | 是否有效 Datetime |
| email | 是否有效 E-mail |
| html | 是否有效 HTML 标签 |
| json | 是否有效 JSON |
| rgb | 是否有效 RGB |
| rgba | 是否有效 RGBA |
| uuid | 是否有效通用唯一标识符 |


## **5. 其他验证运算**


| 运算符 | 运算描述 |
| :--- | :--- |
| dir | 是否有效目录 |
| file | 是否有效文件目录 |
| isdefault | 是默认值 |
| len | 指定长度 |
| max | 最大值 |
| min | 最小值 |
| required | 必须传入 |
| unique | 唯一的 |


## **总结**

<br />validator 好用之处是一次写就可以多次使用无需写繁琐代码来进行数据验证，通常验证规则单独拿出来放在一个包中，需要时直接调用，不用在业务逻辑代码中验证。

[点击了解更多](https://github.com/go-playground/validator)

## **参考资料**


1. [GitHub README](https://github.com/go-playground/validator/blob/master/README.md)
1. [pkg.go.dev validator](https://pkg.go.dev/github.com/go-playground/validator#readme-package-validator)
