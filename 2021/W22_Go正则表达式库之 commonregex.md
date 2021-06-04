## 为什么使用 commonregex?

在开发的时候，我们会遇到一些需要使用字符串的匹配和查找的任务。我们可以使用正则表达式去提取感兴趣的数据，如手机号码，邮件，超链接等。但是正则表达式写起来费时费力，而且容易遗忘。`commonregex` 它提供了很多内置的正则表达式，开箱即用，能极大的提高开发体验和开发效率。

## commonregex 简介

提供经常使用的正则表达式的集合。

它提供了这些作为获取与特定模式对应的匹配字符串的简单函数。

- 日期
- 时间
- 电话号码
- 超链接
- 邮件地址
- IPv4/IPv6/IP 地址
- 价格
- 十六进制颜色值
- 信用卡卡号
- 10/13 位 ISBN
- 邮政编码
- MD5
- SHA1
- SHA256
- GUID，全局唯一标识
- Git 仓库地址

## 快速使用 commonregex

### 安装 commonregex

```shell
go get -u github.com/mingrammer/commonregex
```

### 简单使用 commonregex

```go
package main

import (
  "fmt"

  cregex "github.com/mingrammer/commonregex"
)

func main() {
  text := `John, please get that article on www.linkedin.com to me by 5:00PM on Jan 9th 2012. 4:00 would be ideal, actually. If you have any questions, You can reach me at (519)-236-2723x341 or get in touch with my associate at harold.smith@gmail.com`

  dateList := cregex.Date(text)
  timeList := cregex.Time(text)
  linkList := cregex.Links(text)
  phoneList := cregex.PhonesWithExts(text)
  emailList := cregex.Emails(text)

  fmt.Println("date list:", dateList)
  fmt.Println("time list:", timeList)
  fmt.Println("link list:", linkList)
  fmt.Println("phone list:", phoneList)
  fmt.Println("email list:", emailList)
}
```

运行结果

```shell
date list: [Jan 9th 2012]
time list: [5:00PM 4:00 ]
link list: [www.linkedin.com harold.smith@gmail.com]
phone list: [(519)-236-2723x341]
email list: [harold.smith@gmail.com]
```

`commonregex`提供的 API 非常易于使用，调用相应的类别方法返回一段文本中符合这些格式的字符串列表。

上面依次从`text`获取**日期列表**，**时间列表**，**超链接列表**，**电话号码列表**和**电子邮件列表**。

## 总结

`commonregex` 提供了常用的正则表达式的函数，足以应付我们日常开发场景，能较大的提高我们的开发效率。

## 参考资料

* https://github.com/mingrammer/commonregex
* https://darjun.github.io/2020/09/05/godailylib/commonregex/