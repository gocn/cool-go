# Go Web 路由处理利器 gorilla/mux 库

## 1. gorilla/mux 是什么？

[gorilla/mux](https://github.com/gorilla/mux) 是用来处理路由请求和请求对应的方法的映射关系的包，简单地说，[gorilla/mux](https://github.com/gorilla/mux) 就是把收到的请求与一组预先定义的 URL 路径列表做对比，然后在匹配到路径的时候调用关联的处理器（Handler）。

## 2. 为什么那么受欢迎？

[gorilla/mux](https://github.com/gorilla/mux)  在标准库 [net/http](https://golang.org/pkg/net/http/) 之上开发的，因此 [gorilla/mux](https://github.com/gorilla/mux) 比较能干，标准能干的也能干，不能干的也能干；标准库在运行效率上占优，[gorilla/mux](https://github.com/gorilla/mux) 在开发效率上占优；，gorilla/mux 是目前功能最为强大的路由包之一，它提供功能全面而强大,这也是它连续多年成为使用率最高的 Go 第三方包的原因。

## 3. 什么时候用到它？

[gorilla/mux](https://github.com/gorilla/mux) 的路由解析所采用的是精准匹配规则，标准库 net/http 采用的是长度优先匹配规则，精准匹配是只会匹配准确指定的路由，而长度优先匹配不支持动态元素，也就是不支持正则以及 URL 路径参数，只能匹配字符数较多的路由；开发 Web，API 时路由一般时动态的，路由参数会跟着访问对象不同而不同，这是 [gorilla/mux](https://github.com/gorilla/mux) 恰好满足了这种精准路由匹配需求。

在知名的路由包中 [gorilla/mux](https://github.com/gorilla/mux) 的功能比较强大，使用相对简单且很实，历史也非常悠久，可以放心去使用。

## 3. 怎么使用

使用前要安装它，安装要在当前项目所在目录中执行 `go get -u github.com/gorilla/mux` 即可。

安装完写一个 Web 应用示例：

```go
package main

import (
	"fmt"
	"net/http"

	"github.com/gorilla/mux"
)

func homeHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w, "<h1>This is homePage!</h1>")
}

func articlesIndexHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w, "<h1>This is articlesIndexPage!</h1>")
}

func notFoundHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w, "<h1>This is notFoundPage</h1>")
}

func articlesShowHandler(w http.ResponseWriter, r *http.Request) {
	//mux 提供 mux.Vars(r) 的方法会将 URL 路径参数解析为 Map， key 是路由参数，value 是参数对应的值，
	vars := mux.Vars(r)
	id := vars["id"]
	fmt.Fprint(w, "<h1>This articles‘s ID："+id+"</h1>")
}

func articlesStoreHandler(w http.ResponseWriter, r *http.Request) {
	//获取表单中传输的数据方式有 r.ParseForm, ParseFormValue 等，具体如下：
	//r.ParseForm() 是由标准库 http 包提供的，从请求中解析请求参数，必须是执行完这段代码，
	//再用 r.PostForm 和 r.Form 来获取到数据，不事先 r.ParseForm() 来解析时获取不到数据。
	err := r.ParseForm()
	if err != nil {
		fmt.Fprint(w, "请在表单中填写数据再提交！")
		return
	}

	title := r.PostForm.Get("title")
	content := r.PostForm.Get("content")
	author := r.PostForm.Get("author")
	fmt.Fprintf(w, "title 的值为: %v", title)
	fmt.Fprintf(w, "content 的值为: %v", content)
	fmt.Fprintf(w, "author 的值为: %v", author)

	//PostForm 只能能读取 post、put 方式传输的参数，在使用之前需要调用 ParseForm 方法。
	fmt.Fprintf(w, "POST PostForm: %v <br>", r.PostForm)
	//Form 能读取 post、put 和 get 方式传输的参数，在使用之前必须调用 ParseForm 方法来解析路由
	fmt.Fprintf(w, "POST Form: %v <br>", r.Form)
	//  直接获取想要的参数，则直接使用 r.PostFormValue() 方法即可,无需调用 ParseForm 方法来解析路由
	fmt.Fprintf(w, "r.Form() 中 title 的值为: %v <br>", r.FormValue("title"))
	fmt.Fprintf(w, "r.PostForm() 中 title 的值为: %v <br>", r.PostFormValue("title"))
}


func main() {
	//注册路由
	router := mux.NewRouter()

	//精准匹配的
	router.HandleFunc("/", homeHandler).Methods("GET").Name("home")
	router.HandleFunc("/articles/{id:[0-9]+}", articlesShowHandler).Methods("GET").Name("articles.show")
	router.HandleFunc("/articles", articlesIndexHandler).Methods("GET").Name("articles.index")
	router.HandleFunc("/articles/{id:[0-9]+}", articlesStoreHandler).Methods("GET").Name("articles.store")

	//未匹配到路由时匹配到 notFoundHandler
	router.NotFoundHandler = http.HandlerFunc(notFoundHandler)

	//监听路由
	http.ListenAndServe(":6060", router)
}
```

写完示例，执行 `go run main.go` 启动 Web 服务；再用 Postman 来访问一下 url 即可看到返回结果。

- GET 方式访问 `localhost:6060/` 
- GET 方式访问 `localhost:6060/articls` 
- GET 方式访问 `localhost:6060/articles/2` 
- POST 方式访问 `localhost:6060/articles` 

以上的所以路由中用到了路由别名 `router.HandleFunc(_, _).Methods("GET")..Name("别名")' ，除此之外在 `articlesIndexHandler).Methods("GET").Name("articles.index")
	router.HandleFunc("/articles/{id:[0-9]+}", ` 中用到了动态路由，也就是 id 不一样展示的对象也不一样。

除了提供 Web 开发常用功能之外，[gorilla/mux](https://github.com/gorilla/mux) 包还提供其他一些功能，比如，域名绑定，路由分组，路由绑定前缀，路由中间件等，看具体业务需求选择性使用即可。

## 总结

[gorilla/mux](https://github.com/gorilla/mux) 库是历史悠久，从使用文档齐全、其质量和社区活跃度都很 nice，多年 Go 趋势报告中指出 [gorilla/mux](https://github.com/gorilla/mux) 是整个社区使用率最高的第三方库，占据 36% 的市场份额。

[gorilla/mux](https://github.com/gorilla/mux) 使用方式简单，扩展性好，功能强大且全面，也可以根据自身业务需要对它进行定制化的二次开发。

[gorilla/mux](https://github.com/gorilla/mux) 能大大提升 Web 应用开发效率，使用简单不粗暴。

## 参考资料

- [https://github.com/gorilla/mux](https://github.com/gorilla/mux)
- [2021 Go 趋势报告](https://mp.weixin.qq.com/s/p7ex8RPg4Fhi51_kB9JPxg)
- [Go 语言构建 RESTful Web 服务](https://mp.weixin.qq.com/s/0f7Tzfm8kdPh3j49GfKpzQ)