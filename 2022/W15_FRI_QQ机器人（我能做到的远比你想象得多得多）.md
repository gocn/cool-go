# QQ机器人 go-cqhttp

## 什么是 go-cqhttp?

QQ机器人，可以做的事儿太多了，比如一个UP主需要群发多个QQ群，以便通知粉丝们开播；再比如可以检测群内或发给自己的消息，而通过代码直接回复做的简单回复。比如检测群内有加入或退出群时的消息而发送友好问候与告别白等。里面的功能还有很多，比如批量获取群成员，嘿嘿，具体能做什么，就自己发挥自己的想象力吧~



## 请不要用于商业或打扰他人的用途噢~



## 安装 go-cqhttp

```go
// 做为一个golang开发，直接源码编译，其实更方便。
https://github.com/Mrs4s/go-cqhttp
clone好了接着看。
或者https://github.com/Mrs4s/go-cqhttp/releases中下载编译好了的包也行
```

## 开始使用

1. 源码的直接编译即可，然后运行，首次运行后，会在当前运行目录下生成一个config.yml。

2. 打开config.yml，写入你的uin，也就是QQ号（数字类型），然后输入您的password（字符串类型），需要加英文单引号，然后保存当前文件。

3. 再次运行执行文件，会让你选择通信方式，这时我们选0 -> 回车

4. 首次登录需要用验证，就选1向手机发送短信验证码，之后输入验证码 -> 回车

5. 此时就已经登录成功，并开始监听5700端口

6. API文档一定要看：[API | go-cqhttp 帮助中心](https://docs.go-cqhttp.org/api)

## 入门给群里发送消息

```json
POST:127.0.0.1:5700/send_group_msg
Content-type : application/json

{
    "group_id":"538732374",
    "message":"大家好，我测试一下，可忽略本条消息"
}
```

![QQ图片20220415102411.png](https://cdn.gocn.vip/forum-user-images/20220415/f6033d070e5b4a8c931e59a44509c847.jpg)

## 多账号登录的配置：

打开配置文件config.yml

```shell
# 连接服务列表
servers:
  # 添加方式，同一连接方式可添加多个，具体配置说明请查看文档
  #- http: # http 通信
  #- ws:   # 正向 Websocket
  #- ws-reverse: # 反向 Websocket
  #- pprof: #性能分析服务器

  - http: # HTTP 通信设置
      host: 127.0.0.1 # 服务端监听地址
      port: 5700      # 服务端监听端口
      timeout: 5      # 反向 HTTP 超时时间, 单位秒，<5 时将被忽略
      long-polling:   # 长轮询拓展
        enabled: false       # 是否开启
        max-queue-size: 2000 # 消息队列大小，0 表示不限制队列大小，谨慎使用
      middlewares:
        <<: *default # 引用默认中间件
      post:           # 反向HTTP POST地址列表
      #- url: ''                # 地址
      #  secret: ''             # 密钥
      #  max-retries: 3         # 最大重试，0 时禁用
      #  retries-interval: 1500 # 重试时间，单位毫秒，0 时立即
      #- url: http://127.0.0.1:5701/ # 地址
      #  secret: ''                  # 密钥
      #  max-retries: 10             # 最大重试，0 时禁用
      #  retries-interval: 1000      # 重试时间，单位毫秒，0 时立即
```

我们需要更改的就是port，每个QQ账号都对应一个port，此后我们就可以运行多个cq了。

再比如，我们要获取群成员列表：

```json
POST:127.0.0.1:5700/get_group_member_list
Content-type : application/json

{
    "group_id":"538732374"
}

// 下面是返回内容

{
    "data": [
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1582388878,
            "last_sent_time": 1640106840,
            "level": "1",
            "nickname": "  S ｕｎ",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 371625
        },
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1565769192,
            "last_sent_time": 1648000653,
            "level": "3",
            "nickname": "  ωill",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 420984
        },
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1578443075,
            "last_sent_time": 1578443075,
            "level": "1",
            "nickname": "。",
            "role": "member",
            "sex": "unknown",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 3546393
        },
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1571772377,
            "last_sent_time": 1571994660,
            "level": "1",
            "nickname": "气运",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 4928040
        },
        {
            "age": 0,
            "area": "",
            "card": "suparna",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1555336437,
            "last_sent_time": 1643027065,
            "level": "1",
            "nickname": "6020940",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 6020940
        },
        {
            "age": 0,
            "area": "",
            "card": "上海-只是爱了童话",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1500864757,
            "last_sent_time": 1547811713,
            "level": "1",
            "nickname": "只是爱了童话",
            "role": "member",
            "sex": "female",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 6168731
        },
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1580104607,
            "last_sent_time": 1581510918,
            "level": "1",
            "nickname": "老矣",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 6202192
        },
        {
            "age": 0,
            "area": "",
            "card": "nickee",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1481884001,
            "last_sent_time": 1516517605,
            "level": "1",
            "nickname": "澎澎",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 7511950
        },
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1560697994,
            "last_sent_time": 1562396877,
            "level": "1",
            "nickname": "不忘初心",
            "role": "member",
            "sex": "unknown",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 9445656
        },
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1540626510,
            "last_sent_time": 1549110296,
            "level": "1",
            "nickname": "JerryGuan",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 10223102
        },
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1548510009,
            "last_sent_time": 1563723272,
            "level": "1",
            "nickname": "向前看",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 10419265
        },
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1563710990,
            "last_sent_time": 1565880929,
            "level": "1",
            "nickname": "メ宇哥哥メ/fw",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 10732275
        },
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1540209338,
            "last_sent_time": 1573990882,
            "level": "1",
            "nickname": "云飘～沉",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 10939443
        },
        {
            "age": 0,
            "area": "",
            "card": "",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1570347146,
            "last_sent_time": 1596110081,
            "level": "1",
            "nickname": "思念",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 11291183
        },
        {
            "age": 0,
            "area": "",
            "card": "广西-脑袋长草",
            "card_changeable": false,
            "group_id": 538732374,
            "join_time": 1499139520,
            "last_sent_time": 1649989856,
            "level": "3",
            "nickname": "脑袋长草",
            "role": "member",
            "sex": "male",
            "shut_up_timestamp": 0,
            "title": "",
            "title_expire_time": 0,
            "unfriendly": false,
            "user_id": 13978955
        }......
```

这就非常恐怖了，user_id就是用户的 QQ号了，自己解析一下，你可以比如。。。群发。。。邮件，当然作者肯定是不希望我们做违法的事情的。

记得，群发消息如果不合规，可能会出现风控，你需要打开go-cqhttp挂机几天才可以继续使用。

## 参考文献

* https://github.com/Mrs4s/go-cqhttp
* https://docs.go-cqhttp.org/