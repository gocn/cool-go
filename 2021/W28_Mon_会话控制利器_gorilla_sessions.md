# 会话控制利器 gorilla/sessions

## 推荐 [gorilla/sessions](https://github.com/gorilla/sessions) 的背景

在日常 Web 应用开发过程中，需要对用户登录状态进行判断，而 HTTP 是无状态的，即不记录用户登录状态，想要得到用户登录状态得把登录状态保存下来，通常用户状态数据加密后用数据库或者缓存等手段把存储；原生的会话数据存储的 HTTP cookie 对数据大小有限制，不能写入太多太大的数据，又不能对存储的数据进行有效管理，而 [gorilla/sessions](https://github.com/gorilla/sessions) 恰恰补充了原生回话控制的不足。

## [gorilla/sessions](https://github.com/gorilla/sessions) 简介

[gorilla/sessions](https://github.com/gorilla/sessions) 支持原生的 HTTP cookie 会话数据存储和存储介质系统会话以及自定义会话等基础设施（Memcache/Redis/SQLite/MySQL ）的支持应有尽有。

[gorilla/sessions](https://github.com/gorilla/sessions) 提供了会话数据的加密解密，以及底层的 cookie 管理，提供的接口比较简单易用，[gorilla/sessions](https://github.com/gorilla/sessions) 主要特点如下：

- 简单的API：使用它作为设置签名（和可选的加密）cookie的简单方法。
- 内置的后端可以在 cookie 或文件系统中存储会话。
- Flash 信息：会话值持续到读取为止。
- 切换会话持久性（又称 "记住我"）和设置其他属性的方便方法。
- 轮换认证和加密密钥的机制。
- 每个请求有多个会话，甚至使用不同的后端。
- 自定义会话后端的接口和基础设施：来自不同端的会话可以使用一个共同的API进行检索和批量保存。

## [gorilla/sessions](https://github.com/gorilla/sessions) 应用

下面是集成 `gorilla/sessions` 的简单会话控制业务代码段，如下：

```go
import (
	"github.com/gorilla/sessions"
	"net/http"
	"os"
)

// Store gorilla sessions 的存储库
var Store = sessions.NewCookieStore([]byte(os.Getenv("SESSION_KEY")))

// Session 当前会话
var Session *sessions.Session

// Request 用以获取会话
var Request *http.Request

// Response 用以写入会话
var Response http.ResponseWriter

// StartSession 初始化会话
func StartSession(w http.ResponseWriter, r *http.Request) {
	Store, _ := Store.Get(r, "session-name")
	// Set some session values.
	session.Values["foo"] = "bar"
	session.Values[42] = 43
	//外部可以这么设置
	//Set("foo","bar")
	//Set("age", 22)
	// Save it before we write to the response/return from the handler.
	session.Save(r, w)
	
	Request = r
	Response = w
	//Save(Request, Response)
}

// Set 写入键值对应的会话数据
func Set(key interface{}, value interface{}) {
	Session.Values[key] = value
	Save()
}

// Get 获取会话数据，获取数据时请做类型检测
func Get(key string) interface{} {
	return Session.Values[key]
}

// Destroy 删除某个会话项
func Destroy(key string) {
	delete(Session.Values, key)
	Save()
}

// Save 保持会话
func Save() {
	Session.Save(Request, Response)
}
```

注意：非 HTTPS 的链接无法使用 Secure 和 HttpOnly，浏览器会报错，因此非 HTTPS 要设置

```go
Session.Options.Secure = true
Session.Options.HttpOnly = true
```

首先，初始化一个会话存储，调用 NewCookieStore() 并传递一个用于验证会话的秘密密钥。其次，在处理器中调用store.Get() 来检索一个现有的会话或创建一个新的会话，在 session.Values 中设置一些会话值，它是一个 map[interface{}]interface{} 。最后，我们调用session.Save()来保存响应中的会话。

值得注意的是，
- 在生产代码中，应该在调用 session.Save(r, w) 时检查是否有错误，并显示错误信息或以其他方式处理。
- 会话数据被存储在map[string]interface{}中，所以在检索数据时需要对其进行类型验证。
- Save 必须在写入响应之前调用，否则会话 cookie 将不会被发送到客户端。

默认情况下，会话 cookies 持续一个月。这可能对有些情况来说可能太长了，但在运行时很容易改变cookies 持续时间和其他相关属性。会话可以被单独配置，也可以服务提供者哪里配置，然后所有使用它保存的会话都将使用该配置。具体地调用 session.Options 或 store.Options 来设置一个新的配置。

## 总结

[gorilla/sessions](https://github.com/gorilla/sessions) 支持原生的 cookie 会话数据存储和文件系统会话以及自定义会话功能，可以在主流的 Go Web 框架或者自创的框架及应用上可以直接拿来就可以使用。

## 参考资料

* [https://github.com/gorilla/sessions](https://github.com/gorilla/sessions)
* [https://pkg.go.dev/github.com/gorilla/sessions#section-readme](https://pkg.go.dev/github.com/gorilla/sessions#section-readme)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)