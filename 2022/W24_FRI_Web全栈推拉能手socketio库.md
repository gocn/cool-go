# Web全栈推拉能手Socket.IO库
## Web前端与后台的通信
+ 短轮询：通过不断发送http请求达到即时通信的目的
+ 长轮询：访问无资源并不会立刻返回，保持较长时间的通讯，直到获得数据或超时返回。
+ WebSocket：基于TCP协议，兼容HTTP协议的基础上，做协议升级，在HTML5中提供client和server进行全双工通信。

## Socket.IO
Socket.IO 是一个封装了 Websocket、基于 Node 的 JavaScript 框架，包含 client 的 JavaScript 和 server 的 Node。其屏蔽了所有底层细节，让顶层调用非常简单。

## Socket.IO的协议
Socket.IO的协议基于engine.io的版本，目前最新版本是4。GO的socket.io库是github.com/graarh/golang-socketio，仅支持EIO=3的协议版本。

## 准备全栈Socket.IO库
### 前端(React typescript):
+ "socket.io-client": "2.3.0"
+ "@types/socket.io-client": "^1.4.33"
### 后端(golang)
```shell
go get github.com/graarh/golang-socketio
go get github.com/graarh/golang-socketio/transport
```

## 使用Socket.IO
### 后端
#### 创建对象
```go
server := gosocketio.NewServer(transport.GetDefaultWebsocketTransport())
```

Server类型实现了http.Handler接口,可以无缝衔接各web框架，例如: http.Handle("/socket.io",server)。

#### 响应事件
```go
//预设
server.on(gosocketio.OnConnection,func(c *gosocketio.Channel){})
server.on(gosocketio.OnDisConnection,func(c *gosocketio.Channel){})
server.on(gosocketio.OnError,func(c *gosocketio.Channel){})

//自定义事件
server.On("send", func(c *gosocketio.Channel, msg Message) string {})
```

#### 推送数据
```go
//回复消息
c.Emit("event","message")

//广播消息
c.BroadcastTo("room","event","message")
server.BroadcastTo("room","event","message")
```
Channel的BroadcastTo 函数在内部实现上调用了Server的BroadcastTo函数。

这里涉及到房间的概念，房间是对访问者的归类，用于局部分类广播消息，加入房间方法为
c.Join("roomName")。

### 前端(React typescript)
#### 创建对象
```typescript
import * as io from 'socket.io-client';
let socket:SocketIOClient.Socket
socket = io.connect("ws://", { transports: ['websocket'] })
```

#### 响应事件
```typescript
socket.on("events", (data: any) => {})
```

#### 发送消息
```typescript
socket.emit("events","message")
```

## 总结
Socket.IO不仅支持 WebSocket，还支持许多种轮询机制以及其他实时通信方式，并封装了通用的接口。这些方式包含 Adobe Flash Socket、Ajax 长轮询、Ajax multipart streaming 、持久 Iframe、JSONP 轮询等。换句话说，当 Socket.IO 检测到当前环境不支持 WebSocket 时，能够自动地选择最佳的方式来实现网络的实时通信。

PS: 目前编者还未找到支持Engine.IO 版本4协议的GO语言库，所以只能降低客户端socket.io-client的版本以完成前后端的适配。







