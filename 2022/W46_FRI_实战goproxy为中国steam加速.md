# 实战goproxy为中国steam登录加速

## 前言

  我们公司领沃云电脑平台的用户总是抱怨steam登录常常失败，自从steam撤走了中国的CM服务器后，每到高峰期就是网络连接失败的重灾区，增量了N多平台的东西，这次终于有时间来收拾这个登录的问题了。



## 先抓个包，确定需要的加速

首先还是用了fiddler工具对steam平台登陆时抓了个包，抓到了他登陆时必用的接口api.steampowered.com 就是这个罪魁祸首了，高峰期响应基本超过20秒以上，或者直接无响应。



## 引入一个新知识，PAC脚本

```go
我最初也只知道手动代理，知道可以填写socks=xxxxx或者直接就是http代理，没想到上面的自动脚本才是真的好用。
PAC也叫代理自动配置脚本，全名是Proxy Auto-config
以下均以windows为例，因为本次用例也主要是为公司的steam平台进行加速
1、打开方式WIN+I打开WINDOWS设置--------->网络和INTERNET--------->左侧：代理--------->最上面的自动检测设置和使用设置脚本都打开
2、开始设置脚本地址，记得脚本地址目前在win10及其以上系统已经不能使用file:// 的方式打开本地文件了，仅支持http , 就连https都不支持，不支持其他协议比如ftp等。
3、脚本的编写，创建一个以pac为后缀名的文件，pac的写法就是js代码（如果不生效或者有问题，一定是你的pac文件，你写错了，你写错了，你写错了！)
```

```js
var proxy = "PROXY 127.0.0.1:9081;";
var direct = "DIRECT;";

function FindProxyForURL(url, host) {
   if (dnsDomainIs(host,"api.steampowered.com")){
      return proxy;
   }else{
      return direct;
   }
}
```

```js
// 注解：先来说说这个proxy 可以写 SOCKS xxx.xxx.xxx.xxx:xxxx 
// 支持容灾：比如这个proxy可以写成如下：
var proxy = "PROXY 127.0.0.1:8080; SOCKS 127.0.0.1:8081; DIRECT";
// 解释一下，意思就是先走http代理PROXY 127.0.0.1:8080，如果走不通就走SOCKS 127.0.0.1:8081,如果还是走不通，就直连不走任何代理了。
// 然后就是一个方法，FindProxyForURL ,里面有不少方法，比如正则匹配shExpMatch(url,"*baidu*")，只要跟baidu沾边的比如api.baidu.com
// 当然本例中只是为了固定的api.steampowered.com域名进行代理加速，就用的是dnsDomainIs(host,"api.steampowered.com")，然后就返回的是proxy代理的内容。其他域名就直接走直连，我默认是没有做容灾的，因为服务器服务是放k8s上面，遇到服务出现问题直接替换服务。
// 还有很多方法可查看文档，就不多赘述https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Proxy_servers_and_tunneling/Proxy_Auto-Configuration_PAC_file
```

## 开启PAC自动配置

编写一份open_pac.bat 的批处理文件

这里自行更改一下倒数第三行最后的http://xxx.pac 改成你自己上传后的地址。

```cil
@echo off
color 0a
title Use autoconfig script
echo Starting......
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v ProxyEnable /t REG_DWORD /d 0 /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Connections" /v DefaultConnectionSettings /t REG_BINARY /d 46000000020000000900 /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Connections" /v SavedLegacySettings /t REG_BINARY /d 46000000020000000900 /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v AutoConfigURL /d "http://xxx.pac" /f
echo End
@echo off
```



## ## 安装goproxy

```go
go get github.com/elazarl/goproxy
```

## 直接上服务端，顺手给http代理加上校验

稍微解释一下原理，我这边儿是做了一个本地代理来进行中转，客户端连接到服务端，然后开一个本地端口8091，然后系统http代理就连接到这个本地服务上去，本地服务拿到请求，带上请求头账号密码，就发给服务端；

服务端此时就是等待用户请求，并查看当前请求的所带头的账号密码是否正确，正确就进行proxy代理即可。

以下是服务端代码： 编译后启动用-p设置端口，如 server -p 8082

```go
package main

import (
   "flag"
   "fmt"
   "log"
   "net/http"

   "github.com/elazarl/goproxy"
   "github.com/elazarl/goproxy/ext/auth"
)

const (
   username = "anyanfei"
   password = "**********"
)

var (
   serverPort = ""
)

func init() {
   flag.StringVar(&serverPort, "p", "8001", "Setting server http proxy port like 8080")
}

func main() {
   flag.Usage = func() {
      flag.PrintDefaults()
   }
   flag.Parse()
   if serverPort == "" {
      log.Fatalln("Error: please setting correct server port")
   }
   endProxy := goproxy.NewProxyHttpServer()
   endProxy.Verbose = true
   auth.ProxyBasic(endProxy, "my_realm", func(user, pwd string) bool {
      return user == username && pwd == password
   })
   log.Println("serving proxy server port at", serverPort)
   http.ListenAndServe(":"+serverPort, endProxy)
   fmt.Println("The proxy server end.")
}


```



### 客户端代码：

```go
package main

import (
   "encoding/base64"
   "fmt"
   "log"
   "net/http"
   "net/url"
   "os"
   "os/signal"
   "syscall"

   "github.com/elazarl/goproxy"
)

const (
   serverIp   = "http://xxx.xxx.xxx.xxx"
   serverPort = ":8082"
   proxyAuthHeader = "Proxy-Authorization"
   username        = "anyanfei"
   password        = "**********"
   middleProxyPort = ":9081"
)

func main() {
   middleProxy := goproxy.NewProxyHttpServer()
   middleProxy.Verbose = true
   middleProxy.Tr.Proxy = func(req *http.Request) (*url.URL, error) {
      return url.Parse(serverIp + serverPort)
   }
   connectReqHandler := func(req *http.Request) {
      SetBasicAuth(username, password, req)
   }
   middleProxy.ConnectDial = middleProxy.NewConnectDialToProxyWithHandler(serverIp+serverPort, connectReqHandler)
   middleProxy.OnRequest().Do(goproxy.FuncReqHandler(func(req *http.Request, ctx *goproxy.ProxyCtx) (*http.Request, *http.Response) {
      return req, nil
   }))
   //log.Println("the final server at", serverIp+serverPort) // 上线这句话有一定不能有
   log.Println("the middle proxy at", middleProxyPort)
   err := http.ListenAndServe(middleProxyPort, middleProxy)
   if err != nil {
      log.Fatalln(err)
   }

   onSignal := make(chan os.Signal)
   signal.Notify(onSignal, syscall.SIGINT, syscall.SIGTERM)
   select {
   case sig := <-onSignal:
      fmt.Println("exit proxy client", sig)
      return
   }
}

// SetBasicAuth 设置proxy请求头，以base64加密当前字符串
func SetBasicAuth(username, password string, req *http.Request) {
   req.Header.Set(proxyAuthHeader, fmt.Sprintf("Basic %s", base64.StdEncoding.EncodeToString([]byte(username+":"+password))))
}

// 直接运行即可
```

  因为goproxy包里面解析的时候用的是Proxy-Authorization 并用base64decode解析了这个请求头，所以客户端加密的时候，也用到了base64encode，详情可以自行去看源码，写得挺不错的。



启动完了就可以用了，为我们领沃云电脑解决steam用户登录不上的问题贡献一份力量。

## 参考资料

[goproxy](https://github.com/elazarl/goproxy)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)