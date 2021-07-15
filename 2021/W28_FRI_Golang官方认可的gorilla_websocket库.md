# W28_FRI_Golang官方认可的websocket库-gorilla/websocket

## 推荐理由

Golang官方标准库实现的websocket在功能上有些欠缺，本次介绍的gorilla/websocket库，是[Gorilla](https://www.gorillatoolkit.org)出品的速度快、质量高，并且被广泛使用的websocket库，很好的弥补了标准库功能上的欠缺。另外Gorilla Web toolkit包含多个实用的HTTP应用相关的工具库，感兴趣可以到官网主页<https://www.gorillatoolkit.org>自取。

## 功能介绍

gorilla/websocket库是 RFC 6455 定义的websocket协议的一种实现，在数据收发方面，提供Data Messages、Control Messages两类message粒度的读写API；性能方面，提供Buffers和Compression的相关配置选项；安全方面，可通过CheckOrigin来控制是否支持跨域。

### gorilla/websocket库和官方实现的对比

> *摘自 gorilla GitHub 主页*
<table>
<tr>
<th></th>
<th><a href="http://godoc.org/github.com/gorilla/websocket">github.com/gorilla</a></th>
<th><a href="http://godoc.org/golang.org/x/net/websocket">golang.org/x/net</a></th>
</tr>
<tr>
<tr><td colspan="3"><a href="http://tools.ietf.org/html/rfc6455">RFC 6455</a> Features</td></tr>
<tr><td>Passes <a href="https://github.com/crossbario/autobahn-testsuite">Autobahn Test Suite</a></td><td><a href="https://github.com/gorilla/websocket/tree/master/examples/autobahn">Yes</a></td><td>No</td></tr>
<tr><td>Receive <a href="https://tools.ietf.org/html/rfc6455#section-5.4">fragmented</a> message<td>Yes</td><td><a href="https://code.google.com/p/go/issues/detail?id=7632">No</a>, see note 1</td></tr>
<tr><td>Send <a href="https://tools.ietf.org/html/rfc6455#section-5.5.1">close</a> message</td><td><a href="http://godoc.org/github.com/gorilla/websocket#hdr-Control_Messages">Yes</a></td><td><a href="https://code.google.com/p/go/issues/detail?id=4588">No</a></td></tr>
<tr><td>Send <a href="https://tools.ietf.org/html/rfc6455#section-5.5.2">pings</a> and receive <a href="https://tools.ietf.org/html/rfc6455#section-5.5.3">pongs</a></td><td><a href="http://godoc.org/github.com/gorilla/websocket#hdr-Control_Messages">Yes</a></td><td>No</td></tr>
<tr><td>Get the <a href="https://tools.ietf.org/html/rfc6455#section-5.6">type</a> of a received data message</td><td>Yes</td><td>Yes, see note 2</td></tr>
<tr><td colspan="3">Other Features</tr></td>
<tr><td><a href="https://tools.ietf.org/html/rfc7692">Compression Extensions</a></td><td>Experimental</td><td>No</td></tr>
<tr><td>Read message using io.Reader</td><td><a href="http://godoc.org/github.com/gorilla/websocket#Conn.NextReader">Yes</a></td><td>No, see note 3</td></tr>
<tr><td>Write message using io.WriteCloser</td><td><a href="http://godoc.org/github.com/gorilla/websocket#Conn.NextWriter">Yes</a></td><td>No, see note 3</td></tr>
</table>

> *Notes:*
> 1. Large messages are fragmented in [Chrome's new WebSocket implementation](http://www.ietf.org/mail-archive/web/hybi/current/msg10503.html).
> 2. The application can get the type of a received data message by implementing
>   a [Codec marshal](http://godoc.org/golang.org/x/net/websocket#Codec.Marshal)
>   function.
> 3. The go.net io.Reader and io.Writer operate across WebSocket frame boundaries.
>  Read returns when the input buffer is full or a frame boundary is
>  encountered. Each call to Write sends a single frame message. The Gorilla
>  io.Reader and io.WriteCloser operate on a single WebSocket message.

## 使用指南

### 安装

```shell
go get github.com/gorilla/websocket
```

### 基础示例

下面以一个简单的echo来说明gorilla/websocket库基本使用。
client代码：

```go
package main

import (
	"flag"
	"log"
	"net/url"
	"os"
	"os/signal"
	"time"

	"github.com/gorilla/websocket"
)

var addr = flag.String("addr", "localhost:8080", "http service address")

func main() {
	flag.Parse()
	log.SetFlags(0)

    // 用来接收命令行的终止信号
	interrupt := make(chan os.Signal, 1)
	signal.Notify(interrupt, os.Interrupt)

    // 和服务端建立连接
	u := url.URL{Scheme: "ws", Host: *addr, Path: "/echo"}
	log.Printf("connecting to %s", u.String())

	c, _, err := websocket.DefaultDialer.Dial(u.String(), nil)
	if err != nil {
		log.Fatal("dial:", err)
	}
	defer c.Close()

	done := make(chan struct{})

	go func() {
		defer close(done)
		for {
            // 从接收服务端message
			_, message, err := c.ReadMessage()
			if err != nil {
				log.Println("read:", err)
				return
			}
			log.Printf("recv: %s", message)
		}
	}()

	ticker := time.NewTicker(time.Second)
	defer ticker.Stop()

	for {
		select {
		case <-done:
			return
        case t := <-ticker.C:
            // 向服务端发送message
			err := c.WriteMessage(websocket.TextMessage, []byte(t.String()))
			if err != nil {
				log.Println("write:", err)
				return
			}
		case <-interrupt:
			log.Println("interrupt")

			// 收到命令行终止信号，通过发送close message关闭连接。
			err := c.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseNormalClosure, ""))
			if err != nil {
				log.Println("write close:", err)
				return
            }
            // 收到接收协程完成的信号或者超时，退出
			select {
			case <-done:
			case <-time.After(time.Second):
			}
			return
		}
	}
}
```

server代码：
```go
package main

import (
	"flag"
	"html/template"
	"log"
	"net/http"

	"github.com/gorilla/websocket"
)

var addr = flag.String("addr", "localhost:8080", "http service address")

var upgrader = websocket.Upgrader{}

func echo(w http.ResponseWriter, r *http.Request) {
    // 完成和Client HTTP >>> WebSocket的协议升级
	c, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Print("upgrade:", err)
		return
	}
	defer c.Close()
	for {
        // 接收客户端message
		mt, message, err := c.ReadMessage()
		if err != nil {
			log.Println("read:", err)
			break
		}
        log.Printf("recv: %s", message)
        // 向客户端发送message
		err = c.WriteMessage(mt, message)
		if err != nil {
			log.Println("write:", err)
			break
		}
	}
}

func main() {
	flag.Parse()
	log.SetFlags(0)
	http.HandleFunc("/echo", echo)
	log.Fatal(http.ListenAndServe(*addr, nil))
}
```

更多示例可以参考 [https://github.com/gorilla/websocket/tree/master/examples](https://github.com/gorilla/websocket/tree/master/examples)

## 总结

gorilla/websocket库是websocket协议的一种实现，相比标准库的实现，封装了协议细节，使用者关注message粒度的API即可，但需要注意message的读写API非并发安全，使用上要注意不要多个协程并发调用。

## 参考资料

1. [https://github.com/gorilla/websocket](https://github.com/gorilla/websocket)
