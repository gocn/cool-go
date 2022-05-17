# 微软开源的支持多平台多协议的网络性能检测工具 Ethr

## 什么是 Ethr?

Ethr 是一个用 golang 编写的跨平台并可支持多种协议的网络性能测量工具。支持 TCP, UDP, HTTP, HTTPS 等多种协议，对带宽、连接/s, 延迟、数据包/s、丢失和抖动有一个全面的网络性能测量。

## Ethr 特性与优势

* 跨平台，支持多种协议
* 支持多线程，支持多客户端与单个服务器的通信
* 与其他工具相比提供了更多的测试指标
* 可作为服务端或者客户端对网络性能进行测量

### 快速体验 Ethr

安装 Ethr

```shell
go install github.com/microsoft/ethr@latest
```

1. 启动服务端, 默认监听 8888 端口, 默认使用 TCP 协议

```shell
ethr -s -ui
# ethr -s -ui -port 9999
```

![image-20220517215137197](https://resource.gocloudcoder.com/image-20220517215137197.png)

2. 使用客户端进行测试

* -c 指定 server
* -t 指定测试类型
  * b: Bandwidth
  * c: connections/s
  * p: Packets/s
  * l: Latency, Loss & Jitter
  * tr: TraceRoute
  * mtr: MyTraceRoute with Loss & Latency
* -p 指定协议
* -n 指定线程数
* -4 代表 ipv4(-6 代表 ipv6)
* -d 表示测试时间

```shell
ethr -c localhost -t c -p tcp -n 64 -4 -d 100s
```

![image-20220517220742597](https://resource.gocloudcoder.com/image-20220517220742597.png)

```shell
ethr -c localhost -t p -p udp -n 64 -6 -d 100s
```

![image-20220517220819834](https://resource.gocloudcoder.com/image-20220517220819834.png)

3. 对指定服务进行测试

* -x 指定服务端地址
* -n 使用 8 个线程
* -t c 测量每秒连接数

![image-20220517221248908](https://resource.gocloudcoder.com/image-20220517221248908.png)

**更多使用请使用 `ethr --help` 进行查看。**

## 参考链接

* [https://github.com/microsoft/ethr](https://github.com/microsoft/ethr)