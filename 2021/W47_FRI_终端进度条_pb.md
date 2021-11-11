# 终端进度条-pb

## 什么是 pb？

pb是一个Go语言的终端进度条库。

## 什么时候需要pb?

终端显示的工具进行定时等待、IO传输等操作时，都可以用pb来显示当前进度。

## pb入门
### 安装pb

```shell
go get github.com/cheggaaa/pb/v3
```

### 快速上手

```go
package main

import (
	"time"

	"github.com/cheggaaa/pb/v3"
)

func main() {
	count := 100000

	// 创建进度条并开始
	bar := pb.StartNew(count)

	for i := 0; i < count; i++ {
		bar.Increment()
		time.Sleep(50 * time.Microsecond)
	}

	// 结束进度条
	bar.Finish()
}

```
我们可以看到如下的效果
```
47921 / 100000 [--------------------->______________________] 47.92% 13945 p/s
```

### 自定义进度条的方法
```go
// 创建进度条
bar := pb.New(count)

// 设置刷新速度（时间间隔）（默认为200毫秒）
bar.SetRefreshRate(time.Second)

// 强制设置io.Writer，默认为os.Stderr
bar.SetWriter(os.Stdout)

// 进度条将数字格式化为字节（B、KiB、MiB等）
bar.Set(pb.Bytes, true)

// 进度条使用SI字节前缀名称（B，kB）代替IEC（B，KiB）
bar.Set(pb.SIBytesPrefix, true)

// 设置自定义的进度条模板
bar.SetTemplateString(myTemplate)

// 设置模板后检查错误
if err := bar.Err(); err != nil {
return
}

// 进度条开始
bar.Start()

```
### IO操作的进度条

```go
package main

import (
	"crypto/rand"
	"io"
	"io/ioutil"

	"github.com/cheggaaa/pb/v3"
)

func main() {
	var limit int64 = 1024 * 1024 * 500

	// 我们将把500MiB从/dev/rand复制到/dev/null
	reader := io.LimitReader(rand.Reader, limit)
	writer := ioutil.Discard

	// 开始进度条
	bar := pb.Full.Start64(limit)

	// 创建代理读取器
	barReader := bar.NewProxyReader(reader)

	// 从代理读取器复制
	io.Copy(writer, barReader)

	// 结束进度条
	bar.Finish()
}

```

运行效果

```
258.07 MiB / 500.00 MiB [---------------->_______________] 51.61% 180.84 MiB p/s ETA 1s
```

### 自定义进度条模板

基于内置文本/模板包的渲染。您可以使用现有pb的元素，也可以创建自己的元素。

[element.go](https://github.com/cheggaaa/pb/blob/master/v3/element.go) 文件中描述了所有可用元素。

我们可以将下面这个例子带入到上面IO操作的进度条的DEMO中，这里就不展示效果了。

```go
tmpl := `{{ red "With funcs:" }} {{ bar . "<" "-" (cycle . "↖" "↗" "↘" "↙" ) "." ">"}} {{speed . | rndcolor }} {{percent .}} {{string . "my_green_string" | green}} {{string . "my_blue_string" | blue}}`

// 开始基于我们的模板的进度条
bar := pb.ProgressBarTemplate(tmpl).Start64(limit)

// 设置字符串元素的值
bar.Set("my_green_string", "green").Set("my_blue_string", "blue")

```


## 总结

pb可以帮助我们丰富Golang编写的终端工具，希望大家可以借此开发出更多实用且有趣的东西。


## 参考链接
https://github.com/cheggaaa/pb/

