# chromedp一期 —— 截图小工具

## **推荐背景**

chromedp是谷歌官方推出的无头浏览器，类似Selenium，但是由于Selenium对golang的支持并不是很好，而且我们也不需要去配置繁琐的chrome driver，有助于咱们golang的跨平台使用。

## **快速使用**

### 准备环境

由于笔者只有windows和centos系统，就只介绍这两个系统的浏览器必要的安装方式：

```
Windows下直接安装chrome浏览器即可，不能用绿色版，必须是安装版
```

```
Centos下安装必要的驱动与文字库：
yum install https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
yum install mesa-libOSMesa-devel gnu-free-sans-fonts wqy-zenhei-fonts
```

### 安装

```go
// 安装无头浏览器库 
go get github.com/chromedp/chromedp
```

由于网上都喜欢用来做爬虫，我就不做爬虫了，容易面向监狱编程，做个截图工具吧，想起当年PHP还用了第三方的CutyCapt，现在解放了。

### 手写截图工具（就直接上代码，代码中解释）

笔者喜欢一次性写完，自己粘贴复制到代码中实操一次即可。

```go
package main

import (
	"context"
	"flag"
	"io/ioutil"
	"log"
	"os"
	"path/filepath"
	"regexp"
	"strings"
	"time"

	"github.com/chromedp/cdproto/page"
	"github.com/chromedp/chromedp"
)

type Chrome struct {
	Url         string
	UserAgent   string
	FullScreen  bool
	ResolutionX int
	ResolutionY int
	Delay       int64
}

var ChromeType *Chrome

// 初始化输入
func init() {
	ChromeType = new(Chrome)
	flag.StringVar(&ChromeType.Url, "url", "", "web http or https url (must)")
	flag.BoolVar(&ChromeType.FullScreen, "F", false, "full screenshots auto width and height")
	flag.StringVar(&ChromeType.UserAgent, "userAgent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/93.0.4577.82 Safari/537.36", "user agent string to use")
	flag.IntVar(&ChromeType.ResolutionX, "resolutionX", 1920, "screenshot resolution x")
	flag.IntVar(&ChromeType.ResolutionY, "resolutionY", 1080, "screenshot resolution y")
	flag.Int64Var(&ChromeType.Delay, "delay", 0, "delay in seconds between navigation and screenshot")
	flag.Parse()
}

// 网页截屏小工具
func main() {
	if ChromeType.Url == "" {
		panic("url must input!")
	}
	buf, err := ChromeType.Screenshot()
	if err != nil {
		panic(err.Error())
	}
	fp := ChromeType.SafeFileName()
	if err = ioutil.WriteFile(fp, buf, 0644); err != nil {
		panic(err.Error())
	}
	thisPath, _ := os.Getwd()
	log.Println("生成成功，文件名是：", filepath.Join(thisPath, fp))
}

func (chrome *Chrome) Screenshot() ([]byte, error) {
	// 组装 chromeDp的参数设置
	var options []chromedp.ExecAllocatorOption
	options = append(options, chromedp.DefaultExecAllocatorOptions[:]...)
	options = append(options, chromedp.UserAgent(chrome.UserAgent))
	options = append(options, chromedp.DisableGPU)
	options = append(options, chromedp.Flag("ignore-certificate-errors", true))
	options = append(options, chromedp.WindowSize(chrome.ResolutionX, chrome.ResolutionY)) // 分辨率生成

	actX, aCancel := chromedp.NewExecAllocator(context.Background(), options...)
	ctx, cancel := chromedp.NewContext(actX)
	defer aCancel()
	defer cancel()

	var buf []byte

	// 压缩js对话框 如：alert()
	chromedp.ListenTarget(ctx, func(ev interface{}) {
		if _, ok := ev.(*page.EventJavascriptDialogOpening); ok {
			go func() {
				if err := chromedp.Run(ctx,
					page.HandleJavaScriptDialog(true),
				); err != nil {
					panic(err)
				}
			}()
		}
	})

	// 是否全屏
	if chrome.FullScreen {
		if err := chromedp.Run(ctx, chromedp.Tasks{
			chromedp.Navigate(chrome.Url),
			chromedp.Sleep(time.Duration(chrome.Delay) * time.Second),
			chromedp.FullScreenshot(&buf, 100),
		}); err != nil {
			return nil, err
		}

	} else {
		// 不是全屏下就直接生成
		if err := chromedp.Run(ctx, chromedp.Tasks{
			chromedp.Navigate(chrome.Url),
			chromedp.Sleep(time.Duration(chrome.Delay) * time.Second),
			chromedp.CaptureScreenshot(&buf),
		}); err != nil {
			return nil, err
		}
	}
	return buf, nil
}

// SafeFileName 生成安全文件名，且生成png后缀的地址
func (chrome *Chrome) SafeFileName() string {
	name := strings.ToLower(chrome.Url)
	name = strings.Trim(name, " ")

	separators, err := regexp.Compile(`[ &_=+:/]`)
	if err == nil {
		name = separators.ReplaceAllString(name, "-")
	}

	legal, err := regexp.Compile(`[^[:alnum:]-.]`)
	if err == nil {
		name = legal.ReplaceAllString(name, "")
	}

	for strings.Contains(name, "--") {
		name = strings.Replace(name, "--", "-", -1)
	}

	return name + `.png`
}

```

上面几乎注释都有，运行cli：

```go
可以先-h 看看：
go run main.go -h
之后咱们直接来，存储下来的截图是在当前执行命令的位置：
go run main.go -url="https://game.lingwoyun.cn/"
如果要截全屏，加F咯：
go run main.go -url="https://game.lingwoyun.cn/" -F

结果：
PS E:\code\src\first\an_screenshots> go run main.go -url="https://game.lingwoyun.cn/"
2021/11/22 18:24:57 生成成功，文件名是： E:\code\src\first\an_screenshots\https-game.lingwoyun.cn-.png
```

## 总结

[github.com/chromedp/chromedp](github.com/chromedp/chromedp) 这个库中有很多的操作，在我的demo项目中只用了其中的很小一部分，其他更复杂的功能有需要可自行查阅。
