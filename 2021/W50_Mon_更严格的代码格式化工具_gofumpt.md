# 更严格的代码格式化工具 gofumpt

## 前言

`gofmt` 是 golang 自带的代码自动格式化工具，是保证 Go 代码风格一致的大杀器。我们这次要推荐的 `gofumpt` 在 `gofmt` 的基础上添加了一系列更加严格的格式化规则，并保证了对 `gofmt` 的兼容。

`gofumpt` 有助于进一步提升代码质量，它已经成为 `gopls`（Go 官方语言服务）中可选的格式化工具以及 `golangci-lint` 中支持的 linter，正在被越来越多优秀的开源项目（sourcegraph、gitlab 等）所使用。

## gofumpt 简介

`gofumpt`(https://github.com/mvdan/gofumpt) fork 自 `gofmt`，支持与 `gofmt` 几乎相同的命令行参数，因此可以作为 `gofmt` 的直接替代品使用。`gofumpt` 是 `gofmt` 的“超集”，经过 `gofumpt` 格式化的代码也符合 `gofmt` 的要求，其本身所扩展的格式化规则也可能在后续被集成进 `gofmt`。

`gofumpt` 有以下特点：

1. 更多的格式化规则(参见 https://github.com/mvdan/gofumpt#Added-rules)
2. 默认跳过对 vendor 的格式化
3. 不对自动生成的代码应用扩展规则
4. 不支持 -r 参数

以上第 2、3 个特性对于格式化整个 go 工程来说尤其友好，在格式化代码的同时可以避免对自动生成代码（mock、proto 等）的入侵！

## 使用举例

安装 `gofumpt`：

```bash
go install mvdan.cc/gofumpt@latest
```

`gofumpt` 的用法与 `gofmt` 用法非常类似，使用过 `gofmt` 的同学上手会非常容易。`gofumpt` 的格式化规则扩展很多，下面只取几个典型的例子进行介绍：

注：以下例子均不会被 `gofmt`(1.17.2) 格式化

### 赋值运算符后不能有空行

```go
package demo

func foo() {
	foo :=
		"bar"
}
```

格式化后：

```go
package demo

func foo() {
	foo := "bar"
}
```

### 简单的错误检查之前不能有空行

```go
package demo

foo, err := processFoo()

if err != nil {
	return err
}
```

格式化后：

```go
package demo

foo, err := processFoo()
if err != nil {
	return err
}
```

### 标准库的导入必须位于顶部的单独组中

```go
package demo

import (
	"foo.com/bar"

	"io"

	"io/ioutil"
)
```

格式化后：

```go
package demo

import (
	"io"
	"io/ioutil"

	"foo.com/bar"
)
```

### 代码简化标志（-s）默认启用

```go
package demo

var _ = [][]int{[]int{1}}
```

格式化后：

```go
package demo

var _ = [][]int{{1}}
```

> 更多的例子请大家参考 https://github.com/mvdan/gofumpt#Added-rules


### 编辑器集成

`gofumpt` 除了作为命令行工具直接使用之外，还可以集成到编辑器中供我们使用，例如在 vscode 中集成 `gofumpt` 只需要以下配置：

```json
"go.useLanguageServer": true,
"gopls": {
	"formatting.gofumpt": true,
},
```

> 更多的编辑器集成请参考 https://github.com/mvdan/gofumpt#installation

## 总结

`gofumpt` 是一款优秀的 go 代码格式化扩展工具，能够帮助我们进一步提升代码质量，来试试看吧！

## 参考资料

* [https://github.com/mvdan/gofumpt](https://github.com/mvdan/gofumpt)
* [https://golangci-lint.run/usage/linters/](https://golangci-lint.run/usage/linters/)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
