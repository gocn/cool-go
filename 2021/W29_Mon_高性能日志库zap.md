# 高性能日志库 zap

## zap 简介

zap 是高效的、结构化的，分日志级别的 Go 日志库。

## 为什么选择使用 zap?

首先，在我们写项目中需要一个好的日志记录器，需要提供以下功能：

- 能够将事件记录到文件中，而不是应用程序控制台。
- 日志切割-能够根据文件大小、时间或间隔等来切割日志文件。
- 支持不同的日志级别。例如 INFO，DEBUG，ERROR 等。
- 能够打印基本信息，如调用文件/函数名和行号，日志时间等。

在 Go 中我们可选的日志库有下列几种：

- 标准库 log
- logrus
- zap

对于标准库 log 而言，它最大的优势是使用简单，但是它仅限一些基本的日志级别。对于错误日志，缺少 ERROR 日志级别（这个级别可以在不抛出 panic 或退出程序的情况下记录错误），缺乏日志格式化的能力，不提供日志切割的能力，所以实际使用起来体验并不好。

对于 logrus 而言，业界内评价也非常的好，易用性好，定制化很强。但是在高性能的场景下，logrus 就显得有些力不从心，比较消耗资源。

综合考量下，选择 zap 是一个比较好的选择。

zap 在整体设计上有精细的考量，不仅仅是在高性能上面的出色表现，更多的在于其设计和工程实践上：

- 合理的代码组织结构，结构清晰的抽象关系
- 写实复制，避免加锁
- 对象内存池，避免频繁创建销毁对象
- 避免使用 fmt，json/encode，使用字符编码方式对日志信息编码，使用 byte slice 的形式对日志内容进行拼接编码操作

## zap 入门使用

首先我们安装 zap

```shell
go get -u go.uber.org/zap
```

Zap 提供了两种类型的日志记录器：

- Sugared Logger

- Logger

在性能很好但不是很关键的上下文中，使用`SugaredLogger`。它比其他结构化日志记录包快 4-10 倍，并且支持结构化和 printf 风格的日志记录。

在每一微秒和每一次内存分配都很重要的上下文中，使用`Logger`。它甚至比`SugaredLogger`更快，内存分配次数也更少，但它只支持强类型的结构化日志记录。

### Logger

- 通过调用`zap.NewProduction()`/`zap.NewDevelopment()`或者`zap.Example()`创建一个 Logger。
- 上面的每一个函数都将创建一个 logger。唯一的区别在于它将记录的信息不同。例如 production logger 默认记录调用函数信息、日期和时间等。
- 通过 Logger 调用 Info/Error 等。
- 默认情况下日志都会打印到应用程序的 console 界面。

```go
package main

import (
  "time"

  "go.uber.org/zap"
)

func main() {
  logger := zap.NewExample()
  defer logger.Sync()
  url := "https://gocn.zip"
  logger.Info("failed to fetch URL",
    zap.String("url", url),
    zap.Int("attempt", 3),
    zap.Duration("backoff", time.Second),
  )
}
// {"level":"info","msg":"failed to fetch URL","url":"https://gocn.vip","attempt":3,"backoff":"1s"}
```

### Sugared Logger

- 通过调用主 logger 的`. Sugar()`方法来获取一个`SugaredLogger`。
- 使用`SugaredLogger`以`printf`格式记录语句

```go
package main

import (
  "time"

  "go.uber.org/zap"
)

func main() {
  logger := zap.NewExample()
  defer logger.Sync()
  url := "https://gocn.vip"
  sugar := logger.Sugar()
  sugar.Infow("failed to fetch URL",
    "url", url,
    "attempt", 3,
    "backoff", time.Second,
  )
  sugar.Infof("Failed to fetch URL: %s", url)
}
// {"level":"info","msg":"failed to fetch URL","url":"https://gocn.vip","attempt":3,"backoff":"1s"}
// {"level":"info","msg":"Failed to fetch URL: https://gocn.vip"}
```

## Gin 中嵌入 zap 日志库

我们知道，在 gin 中，默认创建的中间件为 Logger() 和 Recovery()。

我们需要封装 zap，实现这两个中间件。为了使用方便，我们使用 github 上已经封装好的。

```go
package main

import (
	"fmt"
	"time"

	"github.com/gin-contrib/zap"
	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
)

func main() {
	r := gin.New()
	logger, _ := zap.NewProduction()
	r.Use(ginzap.Ginzap(logger, time.RFC3339, true))
	r.Use(ginzap.RecoveryWithZap(logger, true))
	r.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong "+fmt.Sprint(time.Now().Unix()))
	})
	r.GET("/panic", func(c *gin.Context) {
		panic("An unexpected error happen!")
	})
	r.Run(":8080")
}
```

![image-20210718212706959](https://picture.nj-jay.com/image-20210718212706959.png)

若不使用 zap，默认打印的日志如下：

![image-20210718212937040](https://picture.nj-jay.com/image-20210718212937040.png)

## 参考资料

- https://github.com/uber-go/zap
- https://www.liwenzhou.com/posts/Go/zap/
- https://darjun.github.io/2020/04/23/godailylib/zap/
- https://github.com/gin-contrib/zap
