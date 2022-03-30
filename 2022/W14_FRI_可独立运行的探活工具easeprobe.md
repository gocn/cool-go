# 探活工具easeprobe

## 推荐理由

服务探活在现实场景中应用广泛，比如：服务发现、服务负载均衡、服务调度、服务状态监控等。然而，“探活”往往是作为一个功能模块或者组件集成在各个平台系统中。本次要介绍的easeprobe是一款轻量级的，可独立运行的探活工具，利用easeprobe，无需其他系统支持，就可以对多种类型的服务/中间件等探活。
## 功能介绍

easeprobe 除基础的探活功能外，还支持消息通知和定时发送SLA报表。

探活目前支持以下类型：
- HTTP: 检查http状态码，并支持mTLS，HTTP基本身份认证，以及添加请求header/body
- TCP: 检查是否可以建立连接
- Shell：运行shell命令，并检查返回结果
- Client：支持Mysql、Redis、MongoDB、Kafka、PostgreSQL、Zookeeper等客户端

通知支持以下类型：
- 邮件：发送邮件通知
- Slack：Slack webhook通知
- Discord：Discord webhook通知
- Telegram：Telegram 机器人通知
- Log file：记录到日志文件

通知触发方式是“边缘触发”，即只有服务状态发生变化时才会触发通知。

SLA报表支持每日/周/月定时发送


## 使用指南

Go版本要求1.17+
### 安装

```shell
$ git clone git@github.com:megaease/easeprobe.git
$ make
```
### 配置

使用easeprobe需要配置yaml格式的配置文件，这里是一个简单的配置实例：
```yaml
# probe设置
http:
  - name: MegaEase Website
    url: http://megaease.com
# 通知设置
notify:
  email:
    - name: Mail List
      server: smtp.email.example.com:465
      username: user@example.com
      password: ********
      to: "user1@example.com;user2@example.com"
# 全局设置
settings:
  sla:
    schedule: "daily"
    time: "23:59"
  notify:
    retry: # 重试
      times: 5
      interval: 10s
  probe:
    timeout: 30s # 探测超时设置
    interval: 1m # 探测时间间隔1分钟
```
更详细的配置可以到github主页查看(https://github.com/megaease/easeprobe#3-configuration)。

### 运行

```shell
$ build/bin/easeprobe -f config.yaml

INFO[2022-03-30T18:04:06+08:00] Using Standard Output as the log output...
INFO[2022-03-30T18:04:06+08:00] Load the configuration file successfully!
INFO[2022-03-30T18:04:06+08:00] Ready to monitor(http): MegaEase Website - http://megaease.com
INFO[2022-03-30T18:04:06+08:00] [email] configuration: &{Name:Mail List Server:smtp.email.example.com:465 User:user@example.com Pass:aaaa To:user1@example.com;user2@example.com Dry:false Timeout:30s Retry:{Times:5 Interval:10s}}
INFO[2022-03-30T18:04:06+08:00] Successfully setup the notify channel: email
INFO[2022-03-30T18:04:06+08:00] Preparing to send the daily SLA report at 23:59 UTC time...
INFO[2022-03-30T18:04:06+08:00] Next Time to send the SLA Report - 2022-03-30 23:59:00 UTC
ERRO[2022-03-30T17:07:52+08:00] error making get request: Get "http://megaease.com": read tcp 10.111.10.17:54139->104.21.46.25:80: read: connection reset by peer
INFO[2022-03-30T17:07:52+08:00] MegaEase Website (Global) (http://megaease.com) - Status changed [up] ==> [down]
INFO[2022-03-30T17:08:56+08:00] MegaEase Website (Global) (http://megaease.com) - Status changed [down] ==> [up]
```

## 总结

easeProbe是一个基于Go的简单的探测工具，可以检查HTTP、TCP、Shell、Client等类型的服务状态，并可以发送通知和报表。如果有监控多种类型服务存活的需求，可以试试。
## 参考资料

- [https://github.com/megaease/easeprobe](https://github.com/megaease/easeprobe)