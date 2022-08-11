# JSON 数据获取器 JID

## 推荐理由

JSON 格式数据适用范围非常广泛，一个内容丰富的json数据可能很大，使用 JID 可以让你非常舒服的获取到想要到数据。

## 简介

JID 是一个过滤JSON格式数据 cli 工具，提供数据格式提醒，颜色区分显示功能。

## 快速开始

### 安装

```go
go install github.com/simeji/jid/cmd/jid@latest
```

## 获取数据

```go
// 使用 | 输入字符串数据 
echo '{"info":{"date":"2016-10-23","version":1.0},"users":[{"name":"simeji3","uri":"https://example.com/simeji3","id":3}],"userCount":3}}'|jid
// 使用 < 输入文件数据
jid < a.json
// 结合 curl 使用 (抓 gocn 的 job 数据)
curl 'https://gocn.vip/apiv3/home/jobs' \
  -H 'authority: gocn.vip' \
  -H 'sec-ch-ua: " Not A;Brand";v="99", "Chromium";v="98", "Google Chrome";v="98"' \
  -H 'sec-ch-ua-mobile: ?0' \
  -H 'user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.109 Safari/537.36' \
  -H 'sec-ch-ua-platform: "macOS"' \
  -H 'accept: */*' \
  -H 'sec-fetch-site: same-origin' \
  -H 'sec-fetch-mode: cors' \
  -H 'sec-fetch-dest: empty' \
  -H 'referer: https://gocn.vip/' \
  -H 'accept-language: zh-CN,zh;q=0.9' \
  -H 'cookie: _ga=GA1.2.350463308.1621685590; Hm_lvt_928280caf87c86e98e74eefae01ebae4=1636260423; Hm_lvt_d297fc76d67d518fad9ea498b48ed0b1=1650077725; oauth_state=af800884fe5b5d0fe7949063733a9b22339f561e027bb1a225a4933985488054; Hm_lpvt_d297fc76d67d518fad9ea498b48ed0b1=1650077733' \
  --compressed | jid
```

## 效果

![https://raw.githubusercontent.com/wiki/simeji/jid/images/demo-jid-main-640-colorize.gif](https://raw.githubusercontent.com/wiki/simeji/jid/images/demo-jid-main-640-colorize.gif)

*图片来自JID [https://github.com/simeji/jid](https://github.com/simeji/jid)*

## 参考

[https://github.com/simeji/jid](https://github.com/simeji/jid)