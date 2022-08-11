# Go实现的Mock server openmock

## 推荐理由

当项目依赖比较多的其他服务，在测试时，通常连接是这些服务的test环境，但是这样做第一无法保证被依赖服务用例的完备性，第二在实际测试过程中可能会遇到比较多费时费力的联调问题。本次推荐的openmock工具，通过mock的手段，可以简化此类项目的测试过程。

## 功能介绍

openmock支持通过yaml文件，配置HTTP、gRPC、Kafka、AMQP (e.g. RabbitMQ) 等协议的mock服务。

## 使用指南

### 安装

下载代码，并进入目录：

```shell
$ git clone git@github.com:checkr/openmock.git
$ cd openmock
```
使用docker安装

```shell
$ docker run -it -p 9999:9999 -v $(pwd)/demo_templates:/data/templates checkr/openmock 
```

或

```shell
$ docker-compose up
```

检查是否安装成功

```shell
$ curl localhost:9999/ping
```

### 配置示例

openmock的配置文件可以分成4大部分：
1. Schema，openmock可以配置多个行为，每个行为通过key和kind来表示；
2. Except，用来定义协议类型（HTTP、gRPC、Kafka、AMQP），以及触发当前行为的条件；
3. Action，用来定义当前行为的执行结果；
4. Template，用来定义和组装响应payload。

下面是HTTP mock server的配置示例

```yaml
# 这里是 Template
- key: http-request-template
  kind: Template
  template: >
    { "http_path": "{{.HTTPPath}}", "http_headers": "{{.HTTPHeader}}" }
# 这里也是Template
- key: color-template
  kind: Template
  template: >
    { "color": "{{.color}}" }

# 这里是Schema
- key: teapot
  kind: AbstractBehavior
# 这里Except
  expect:
    http:
      method: GET
      path: /teapot
# 这里Action
  actions:
    - reply_http:
        status_code: 418
        body: >
		# 这里使用了Template
          {
            "request-info": {{ template "http-request-template" . }},
            "teapot-info": {{ template "color-template" .Values }}
          }

- key: purple-teapot
  kind: Behavior
  extend: teapot
  values:
    color: purple

```

测试一下，可以看到如下结果

```shell
$ curl localhost:9999/teapot
{"request-info":{"http_path":"/teapot","http_headers":""},"teapot-info":{"color":"purple"}}
```

更多示例请参考：[https://github.com/checkr/openmock/tree/master/demo_templates](https://github.com/checkr/openmock/tree/master/demo_templates)

## 总结

[openmock](https://github.com/checkr/openmock) 通过yaml文件可以实现快速mock服务，并且可以支持多种协议，比如HTTP、gRPC、Kafka、AMQP等。在项目测试时，可以试试使用openmock，看看能否提高项目的测试效率。

## 参考资料

* [https://github.com/checkr/openmock](https://github.com/checkr/openmock)
