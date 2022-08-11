# Kscan 让黑客红客，一念天堂一念地狱的工具

## 1.前言

通常，一个红客会扫描自己的子网下是否有遗漏的时候，会用很多现成的工具，比如对某个IP扫描端口，确认其用途，并在适当的情况下将其放行或阻拦，避免遭到不法分子的攻击与突破，那么[Kscan]([GitHub - lcvvvv/kscan: Kscan是一款纯go开发的全方位扫描器，具备端口扫描、协议检测、指纹识别，暴力破解等功能。支持协议1200+，协议指纹10000+，应用指纹2000+，暴力破解协议10余种。](https://github.com/lcvvvv/kscan))就是一个很好的自测工具，当然也是一个很好的攻击工具，一切为了安全的东西都是一把双刃剑。

## 2.特性

- 纯go开发的全方位扫描器
- 端口扫描、协议检测、指纹识别，暴力破解
- 支持协议1200+，协议指纹10000+，应用指纹2000+，暴力破解协议10余种
- 支持最大2048线程工作

更多功能特性请查看：[Kscan]([GitHub - lcvvvv/kscan: Kscan是一款纯go开发的全方位扫描器，具备端口扫描、协议检测、指纹识别，暴力破解等功能。支持协议1200+，协议指纹10000+，应用指纹2000+，暴力破解协议10余种。](https://github.com/lcvvvv/kscan))

## 3.快速安装

建议的是自己下载源码来进行编译，因为有很多内容可以学习一下。

```go
$ https://github.com/lcvvvv/kscan/releases
```

## 4.简单举例

我们用简单的开1024个线程，对103.53.126.180这个ip下的子网掩码大于24的所有网段进行扫描

```shell
kscan -t 103.53.126.180/24 --threads 1024
```

执行，控制台输出如下：

```

     _   __
    |#| /#/    轻量级资产测绘工具
    |#|/#/   _____  _____     *     _   _
    |#.#/   /Edge/ /Forum|   /#\   |#\ |#|
    |##|   |#|____ |#|      /kv2\  |##\|#|
    |#.#\   \r0cky\|#|     /#/_\#\ |#.#.#|
    |#|\#\ /\___|#||#|____/#/###\#\|#|\##|
    |#| \#\\lcvvvv/ \aels/#/ v1.76#\#| \#|

Tips:可以使用-Pn参数关闭存活性探测，将会将所有主机视为存活

[+]2022/07/18 10:12:55 当前环境为：windows, 输出编码为：utf-8
[+]2022/07/18 10:12:55 成功读取URL地址:[0]个,成功读取主机地址:[256]个，待检测端口:[102400]个
[+]2022/07/18 10:12:55 成功加载NMAP探针:[149]个,指纹[11914]条,favicon指纹:[493]条，keyword指纹:[1327]条
[*]2022/07/18 10:12:55 未检测到qqwry.dat,将关闭CDN检测功能，如需开启，请执行kscan --download-qqwry下载该文件
[+]2022/07/18 10:12:56 主机存活性探测任务下发完毕
smtp://103.53.126.22:22        220http://                 http
smtp://103.53.126.58:21        220http://                 http
ftp://103.53.126.62:21         220FileZil                 FileZilla
ftp://103.53.126.76:21         220Serv-UF                 Serv-Uftpd,Windows,6.4
ftp://103.53.126.86:21         220Microso                 Microsoftftpd,Windows
ftp://103.53.126.86:22         220Microso                 Microsoftftpd,Windows
[*]2022/07/18 10:13:10 主机存活性探测任务完成
unknown://103.53.126.33:21     220ӭSlyar
ftp://103.53.126.86:25         220Microso                 Windows,Microsoftftpd
smtp://103.53.126.131:21       220http://                 http
ssh://103.53.126.98:22         SSH-2.0-Op                 OpenSSH,protocol2.0,7.4
ssh://103.53.126.102:22        SSH-2.0-Op                 7.4,OpenSSH,protocol2.0
ssh://103.53.126.108:22        SSH-2.0-Op                 OpenSSH,protocol2.0,7.4
http://103.53.126.15           该网站暂时无法进行访问     服务活动，促进互联网服务健康有序发展，根,MicrosoftIIShttpd,Windows,7.5
http://103.53.126.5            该网站暂时无法进行访问     2.0,MicrosoftHTTPAPIhttpd,Windows,法进行访问该网站暂时无法进行访问可能由
于
ftp://103.53.126.60:21         220ӭSlyar                  2.0.8orlater,vsftpd
ftp://103.53.126.29:21         220ӭSlyar                  vsftpd,2.0.8orlater
ftp://103.53.126.134:21        220EasyFZS                 Windows,FileZillaftpd
ftp://103.53.126.11:21         220530Plea                 vsftpd,2.0.8orlater
ssh://103.53.126.216:22        SSH-2.0-Op                 OpenSSH,protocol2.0,7.4
http://103.53.126.62:81        BadRequest                 tBadRequest-InvalidH,Windows,2.0,MicrosoftHTTPAPIhttpd
http://103.53.126.124          该网站暂时无法进行访问     未进行备案;原因二：根据工信部相关法规，,7.5,MicrosoftIIShttpd,Windows
http://103.53.126.113          该网站暂时无法进行访问     7.5,MicrosoftIIShttpd,Windows,部备案提示：为了规范互联网信息服务活动，
http://103.53.126.3            该网站暂时无法进行访问     Windows,规，您尚未进行备案;原因二：根据工信部相,MicrosoftIIShttpd,7.5
mssql://103.53.126.5:1433      %4                         Windows,11.00.2100;RTM,MicrosoftSQLServer2012
http://103.53.126.62           该网站暂时无法进行访问     MicrosoftIIShttpd,Windows,8.5,规，您尚未进行备案;原因二：根据工信部相
http://103.53.126.22           该网站暂时无法进行访问     度。未履行备案手续的，不得从事互联网信息,Windows,MicrosoftHTTPAPIhttpd,2.0
http://103.53.126.27           该网站暂时无法进行访问     7.5,MicrosoftIIShttpd,由于以下原因导致：原因一：根据工信部相关,Windows
http://103.53.126.194:88       403-禁止访问:访问被拒绝。  Windows,ASP.NET,7.5,3-禁止访问:访问被拒绝。您无权使用所提,MicrosoftIIShttpd
http://103.53.126.10           该网站暂时无法进行访问     nginx,访问该网站暂时无法进行访问可能由于以下原
http://103.53.126.39           该网站暂时无法进行访问     您尚未进行备案;原因二：根据工信部相关法,MicrosoftIIShttpd,Windows,7.5
http://103.53.126.20           该网站暂时无法进行访问     8.5,MicrosoftIIShttpd,互联网信息服务实行备案制度。未履行备案手,Windows
http://103.53.126.206          该网站暂时无法进行访问     展，根据国务院令第292号《互联网信息服,MicrosoftIIShttpd,Windows,10.0
http://103.53.126.77           该网站暂时无法进行访问     ;原因三：您的网站存在不适宜传播，请联系,MicrosoftIIShttpd,Windows,7.5
http://103.53.126.51:88        该页必须通过安全通道查看   MicrosoftIIShttpd,该页必须通过安全通道查看,Windows,8.5
ftp://103.53.126.66:21         220ӭSlyar                  2.0.8orlater,vsftpd
http://103.53.126.216          该网站暂时无法进行访问     nginx,行备案制度。未履行备案手续的，不得从事互
http://103.53.126.196:88       BadRequest                 2.0,RequestBadRequest-In,MicrosoftHTTPAPIhttpd,Windows
http://103.53.126.161:161      403-禁止访问:访问被拒绝。  2.0,ASP.NET,Windows,问:访问被拒绝。您无权使用所提供的凭据查,MicrosoftHTTPAPIhttpd
ssh://103.53.126.248:22        SSH-2.0-Op                 OpenSSH,protocol2.0,7.4
http://103.53.126.234:88       jkak155d                   户端）登录器本地下载（无需客户端）防劫持,MicrosoftIIShttpd,ASP.NET,Windows,7.5
http://103.53.126.29           该网站暂时无法进行访问     7.5,MicrosoftIIShttpd,Windows,规范互联网信息服务活动，促进互联网服务健
http://103.53.126.190:81       《无尘诛仙》官方网站_无尘诛仙私服-PoweredbyDiscuz! MicrosoftIIShttpd,Windows,7.5,服-PoweredbyDiscuz!MU,ASP.NET
http://103.53.126.32           该网站暂时无法进行访问     MicrosoftIIShttpd,Windows,7.5,您尚未进行备案;原因二：根据工信部相关法
http://103.53.126.46           该网站暂时无法进行访问     7.5,网信息服务备案管理办法》规定，国家对互联,MicrosoftIIShttpd,Windows
http://103.53.126.201          该网站暂时无法进行访问     7.5,法进行访问可能由于以下原因导致：原因一：,MicrosoftIIShttpd,Windows
http://103.53.126.18           该网站暂时无法进行访问     7.5,MicrosoftIIShttpd,：您的网站存在不适宜传播，请联系网站管理,Windows
http://103.53.126.208          该网站暂时无法进行访问     系网站管理员处理。工信部备案提示：为了规,7.5,MicrosoftIIShttpd,Windows
http://103.53.126.202          该网站暂时无法进行访问     MicrosoftIIShttpd,7.5,因导致：原因一：根据工信部相关法规，您尚,Windows
http://103.53.126.142          该网站暂时无法进行访问     法》和信息产业部令第33号《非经营性互联,MicrosoftIIShttpd,Windows,7.5
http://103.53.126.205          该网站暂时无法进行访问     7.5,MicrosoftIIShttpd,Windows,为了规范互联网信息服务活动，促进互联网服
http://103.53.126.224          该网站暂时无法进行访问     Windows,7.5,定，国家对互联网信息服务实行备案制度。未,MicrosoftIIShttpd
http://103.53.126.152          该网站暂时无法进行访问     法》和信息产业部令第33号《非经营性互联,MicrosoftIIShttpd,7.5,Windows
http://103.53.126.167          该网站暂时无法进行访问     因二：根据工信部相关法规，您当前接入商不,nginx
http://103.53.126.78:8088      SwaggerUI                  SwaggerUI,SwaggerUI,Kestrel
http://103.53.126.155          该网站暂时无法进行访问     Windows,，根据国务院令第292号《互联网信息服务,MicrosoftHTTPAPIhttpd,2.0
http://103.53.126.191          该网站暂时无法进行访问     MicrosoftIIShttpd,7.5,一：根据工信部相关法规，您尚未进行备案;,Windows
http://103.53.126.240          该网站暂时无法进行访问     2.4.23,。未履行备案手续的，不得从事互联网信息服,Apachehttpd,(Win32)OpenSSL/1.0.2jPHP/5.4.45
http://103.53.126.136          该网站暂时无法进行访问     MicrosoftIIShttpd,Windows,7.5,服务健康有序发展，根据国务院令第292号
unknown://103.53.126.116:7000  `uwWMZ_hi_
http://103.53.126.133          该网站暂时无法进行访问     2.0,。未履行备案手续的，不得从事互联网信息服,Windows,MicrosoftHTTPAPIhttpd
http://103.53.126.85:1158      BadRequest                 Windows,eHTTPError400.Thereq,MicrosoftHTTPAPIhttpd,2.0
https://103.53.126.214:80      HTTPS                      1.21.6,nginx
http://103.53.126.234          该网站暂时无法进行访问     Windows,7.5,MicrosoftIIShttpd,站管理员处理。工信部备案提示：为了规范互
ssl://103.53.126.51:443        SSL                        MicrosoftIISSSL,Windows
http://103.53.126.66:9004      NoTitle                    /帅哥
http://103.53.126.203          该网站暂时无法进行访问     2.0,MicrosoftHTTPAPIhttpd,Windows,第292号《互联网信息服务管理办法》和信
unknown://103.53.126.13:7004   lAlb?H@Mar
unknown://103.53.126.28:7003   tHlKORfD<m
unknown://103.53.126.85:7008   #Kl<<<FuJT
unknown://103.53.126.23:7009   ddIp=>vZKa
http://103.53.126.103:9005     NoTitle                    /帅哥
unknown://103.53.126.23:7003   ?TLlKPXJ[`
unknown://103.53.126.178:7003  xEGxWgfbLl
unknown://103.53.126.28:7009   J?as?Ma<bP
unknown://103.53.126.201:7010  Cbvs=eh^QM
unknown://103.53.126.219:7075  #Tl<<<FuJS
unknown://103.53.126.136:7200  UUrE06'5
http://103.53.126.202:8022     神话王者公益服不售货币     Windows,7.5,ASP.NET,身合理安排时间享受健康生活Allrigh,MicrosoftIIShttpd
http://103.53.126.37:8787      神陨                       Windows,7.5,﻿神陨本站由灵通网络工作室设计制作,MicrosoftIIShttpd
```

## 

## 5.简单举例2

我们不能简单的里面还提供了暴力破解，账号密码当然是自己定的可以使用文件，也可以修改一下代码即可

```go
// 位置在：core/hydra 下 default_xxx_authlist.go文件中就可以修改
```

```shell
kscan -t 103.53.126.180/24 --threads 1024 --hydra
```



执行，控制台输出如下：

```

     _   __
    |#| /#/    轻量级资产测绘工具
    |#|/#/   _____  _____     *     _   _
    |#.#/   /Edge/ /Forum|   /#\   |#\ |#|
    |##|   |#|____ |#|      /kv2\  |##\|#|
    |#.#\   \r0cky\|#|     /#/_\#\ |#.#.#|
    |#|\#\ /\___|#||#|____/#/###\#\|#|\##|
    |#| \#\\lcvvvv/ \aels/#/ v1.76#\#| \#|

Tips:可以使用--check参数，对-f、--fofa的返回结果进行存活性验证，且不会进行端口扫描

[+]2022/07/18 11:39:39 当前环境为：windows, 输出编码为：utf-8
[+]2022/07/18 11:39:39 成功读取URL地址:[0]个,成功读取主机地址:[256]个，待检测端口:[102400]个
[+]2022/07/18 11:39:40 成功加载NMAP探针:[149]个,指纹[11914]条,favicon指纹:[493]条，keyword指纹:[1327]条
[*]2022/07/18 11:39:40 未检测到qqwry.dat,将关闭CDN检测功能，如需开启，请执行kscan --download-qqwry下载该文件
[+]2022/07/18 11:39:40 hydra模块已开启，开始监听暴力破解任务
[*]2022/07/18 11:39:40 当前已开启的hydra模块为：[ssh rdp ftp smb telnet mysql mssql oracle postgresql mongodb redis]
[+]2022/07/18 11:39:41 主机存活性探测任务下发完毕
ftp://103.53.126.76:21         220Serv-UF                 6.4,Serv-Uftpd,Windows
[+]2022/07/18 11:39:42 [hydra]->开始对103.53.126.76:21[ftp]进行暴力破解，字典长度为：224
ftp://103.53.126.62:21         220FileZil                 FileZilla
[+]2022/07/18 11:39:42 [hydra]->开始对103.53.126.62:21[ftp]进行暴力破解，字典长度为：224
smtp://103.53.126.22:22        220http://                 http
smtp://103.53.126.58:21        220http://                 http
smtp://103.53.126.131:21       220http://                 http
```

可以看到，当对主机进行活性探测下发后，就开始根据不同的常用端口进行普通的暴力破解，比如21ftp:// 3306mysql://等...

## 当你获取到了后台后

你就可以自行根据自己需要，对其进行其他的暴力破解，此工具仅仅是对常用的端口进行破解，对于那些改过端口的，他就不会自己破解了。

## 6.总结

其实黑客红客就在一念之间，红客拿工具是用来发现系统漏洞，发现服务器未合理配置造成的潜在威胁；而黑客可能就用来作为攻击别人的手段，避免自己重复造轮子，但希望当黑客的你永远记住，当你凝视深渊的同时，深渊也在凝视你。

## 参考资料

* [Kscan]([GitHub - lcvvvv/kscan: Kscan是一款纯go开发的全方位扫描器，具备端口扫描、协议检测、指纹识别，暴力破解等功能。支持协议1200+，协议指纹10000+，应用指纹2000+，暴力破解协议10余种。](https://github.com/lcvvvv/kscan))

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
