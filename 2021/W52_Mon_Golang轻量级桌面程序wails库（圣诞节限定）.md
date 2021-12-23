# Golang轻量级桌面程序wails2教学

## 推荐理由

不依赖cgo！ 不依赖cgo！ 不依赖cgo！真的不依赖cgo，且跨平台，原生渲染 无嵌入式浏览器，轻量级，生成的文件很小，而且只有一个可执行文件就可运行。

## 功能介绍

3. 后端使用标准 Go
2. 使用任意前端技术构建 UI 界面
3. 快速为您的 Go 应用生成 Vue、Vuetify、React 前端代码
4. 通过简单的绑定命令将 Go 方法暴露到前端
5. 使用原生渲染引擎 - 无嵌入式浏览器
6. 共享事件系统
7. 原生文件系统对话框
8. 强大的命令行工具
9. 跨多个平台

## 使用指南

本次教学在windows下进行。

### 安装

```shell
1、首先要安装三个必要的东西：
npm: https://nodejs.org/en/download/
webviews2: https://developer.microsoft.com/zh-cn/microsoft-edge/webview2/#download-section (下载常青版引导程序
，记得安装是一定用管理员安装)
*upx： https://github.com/upx/upx/releases/tag/v3.96 (下载后：upx-3.96-win64.zip，然后放入环境变量)

2、golang版本必须是1.17及其以上，安装wails工具：
go install github.com/wailsapp/wails/v2/cmd/wails@latest
3、wails doctor (用此命令查看是否已安装完整必要依赖) 如下图：

```

![1](E:\code\cool-go\2021\images\W52_Mon_Golang轻量级桌面程序wails库（圣诞节限定）\1.png)



### 目录结构概要：

```
/main.go - 主应用
/frontend/ - 前端项目文件
/build/ - 项目构建目录
    /build/appicon.png - 应用程序图标
    /build/darwin/ - Mac 特定的项目文件
    /build/windows/ - Windows 特定的项目文件
/wails.json - 项目配置
/go.mod - Go 模块文件
/go.sum - Go 模块校验文件

frontend目录没有特定于 Wails 的内容，可以是您选择的任何前端项目。
build目录在构建过程中使用。这些文件可以修改以自定义您的构建。如果文件从构建目录中删除，将重新生成默认版本。
go.mod中的默认模块名称是“changeme”。您应该将其更改为更合适的内容。
```



### 开始做我们的圣诞树

1、首先，利用wails工具创建一个项目：

```shell
wails init -n 项目名称
```

2、然后，我们开始写咱们的main： （看到go:embed注解就知道为什么要用go1.17及其以上的版本了吧）

```go
package main

import (
	"embed"
	"log"

	"github.com/wailsapp/wails/v2/pkg/options/mac"

	"github.com/wailsapp/wails/v2"
	"github.com/wailsapp/wails/v2/pkg/logger"
	"github.com/wailsapp/wails/v2/pkg/options"
	"github.com/wailsapp/wails/v2/pkg/options/windows"
)

//go:embed frontend/src
var assets embed.FS

//go:embed build/appicon.png
var icon []byte

func main() {
	// 创建一个APP结构体实例
	app := NewApp()

	// 给这个APP设置参数
	err := wails.Run(&options.App{
		Title:             "GoCN祝天下所有的Gopher圣诞节快乐",
		Width:             720,
		Height:            570,
		MinWidth:          720,
		MinHeight:         570,
		MaxWidth:          1280,
		MaxHeight:         740,
		DisableResize:     false,
		Fullscreen:        false,
		Frameless:         false,
		StartHidden:       false,
		HideWindowOnClose: false,
		RGBA:              &options.RGBA{R: 33, G: 37, B: 43, A: 255},
		Assets:            assets,
		LogLevel:          logger.DEBUG,
		OnStartup:         app.startup,
		OnDomReady:        app.domReady,
		OnShutdown:        app.shutdown,
		Bind: []interface{}{
			app,
		},
		// Windows平台特定选项
		Windows: &windows.Options{
			WebviewIsTransparent: false, // 背景是否半透明
			WindowIsTranslucent:  false, // 导航条是否半透明
			DisableWindowIcon:    false, // 是否关闭窗口上的图标
		},
		Mac: &mac.Options{
			TitleBar:             mac.TitleBarHiddenInset(),
			WebviewIsTransparent: true,
			WindowIsTranslucent:  true,
			About: &mac.AboutInfo{
				Title:   "Vanilla Template",
				Message: "Part of the Wails projects",
				Icon:    icon,
			},
		},
	})

	if err != nil {
		log.Fatal(err)
	}
}
```

3、写咱们的前端，在frontend/src/index.html下写入下面的代码（我也是抄[h阿泉](https://blog.csdn.net/weixin_51563198)的）：

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">
<html>
<head>
    <title>圣诞节快乐</title>
    <meta charset="utf-8">
    <style>
        html, body {
            width: 100%;
            height: 100%;
            margin: 0;
            padding: 0;
            border: 0;
        }

        canvas {
            width: 100%;
            height: 100%;
        }

        div {
            margin: 0;
            padding: 0;
            border: 0;
        }

        .nav {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 27px;
            background-color: white;
            color: black;
            text-align: center;
            line-height: 25px;
        }

        a {
            color: black;
            text-decoration: none;
            border-bottom: 1px dashed black;
        }

        a:hover {
            border-bottom: 1px solid red;
        }

        .previous {
            float: left;
            margin-left: 10px;
        }

        .next {
            float: right;
            margin-right: 10px;
        }

        .green {
            color: green;
        }

        .red {
            color: red;
        }

        textarea {
            width: 100%;
            height: 100%;
            border: 0;
            padding: 0;
            margin: 0;
            padding-bottom: 20px;
        }

        .block-outer {
            float: left;
            width: 22%;
            height: 100%;
            padding: 5px;
            border-left: 1px solid black;
            margin: 30px 3px 3px 3px;
        }

        .block-inner {
            height: 68%;
        }

        .one {
            border: 0;
        }
    </style>
</head>
<body marginwidth="0" marginheight="0">
<canvas id="c" height="800" width="300">
    <script>
        var collapsed = true;

        function toggle() {
            var fs = top.document.getElementsByTagName('frameset')[0];
            var f = fs.getElementsByTagName('frame');
            if (collapsed) {
                fs.rows = '250px,*';
                // enable resizing of frames in firefox/opera
                fs.noResize = false;
                f[0].noResize = false;
                f[1].noResize = false;
            } else {
                fs.rows = '30px,*';
                // disable resizing of frames in firefox/opera
                fs.noResize = true;
                f[0].noResize = true;
                f[1].noResize = true;
            }
            collapsed = !collapsed;
        }

    </script>
    <script>
        var b = document.body;
        var c = document.getElementsByTagName('canvas')[0];
        var a = c.getContext('2d');
        document.body.clientWidth; // fix bug in chrome.
    </script>

    <script>
        // start of submission //
        M = Math;
        Q = M.random;
        J = [];
        U = 16;
        T = M.sin;
        E = M.sqrt;
        for (O = k = 0; x = z = j = i = k < 200;) with (M[k] = k ? c.cloneNode(0) : c) {
            width = height = k ? 32 : W = 446;
            with (getContext('2d')) if (k > 10 | !k) for (font = '60px Impact', V = 'rgba('; I = i * U, fillStyle = k ? k == 13 ? V + '205,205,215,.15)' : V + (147 + I) + ',' + (k % 2 ? 128 + I : 0) + ',' + I + ',.5)' : '#cca', i < 7;) beginPath(fill(arc(U - i / 3, 24 - i / 2, k == 13 ? 4 - (i++) / 2 : 8 - i++, 0, M.PI * 2, 1))); else for (; x = T(i), y = Q() * 2 - 1, D = x * x + y * y, B = E(D - x / .9 - 1.5 * y + 1), R = 67 * (B + 1) * (L = k / 9 + .8) >> 1, i++ < W;) if (D < 1) beginPath(strokeStyle = V + R + ',' + (R + B * L >> 0) + ',40,.1)'), moveTo(U + x * 8, U + y * 8), lineTo(U + x * U, U + y * U), stroke();
            for (y = H = k + E(k++) * 25, R = Q() * W; P = 3, j < H;) J[O++] = [x += T(R) * P + Q() * 6 - 3, y += Q() * U - 8, z += T(R - 11) * P + Q() * 6 - 3, j / H * 20 + ((j += U) > H & Q() > .8 ? Q(P = 9) * 4 : 0) >> 1]
        }
        setInterval(function G(m, l) {
            A = T(D - 11);
            if (l) return (m[2] - l[2]) * A + (l[0] - m[0]) * T(D);
            a.clearRect(0, 0, W, W);
            J.sort(G);
            for (i = 0; L = J[i++]; a.drawImage(M[L[3] + 1], 207 + L[0] * A + L[2] * T(D) >> 0, L[1] >> 1)) {
                if (i == 2e3) a.fillText('Merry Christmas!', U, 412);
                if (!(i % 7)) a.drawImage(M[13], ((157 * (i * i) + T(D * 5 + i * i) * 5) % W) >> 0, ((113 * i + (D * i) / 60) % (290 + i / 99)) >> 0);
            }
            D += .02
        }, 1)
        // end of submission //
    </script>
</body>
</html>
```



4、在当前目录下用命令行开启开发者热加载工具查看效果：

```shell
wails dev
```

效果图来啦，提前祝各位2021圣诞节快乐~：

![1](E:\code\cool-go\2021\images\W52_Mon_Golang轻量级桌面程序wails库（圣诞节限定）\1.gif)

5、当然我们也可以打包成自己的可执行二进制文件哟：

```shell
wails build
```

生成的文件放在了build/bin下。

## 总结

其实之前我也知道有不少的go桌面应用库，可要么就是效率低，要么就是生产的二进制文件太大，但wails似乎让我看到了新希望，很多简单的桌面级小应用可能会因此而诞生更多。

## 参考资料

1. [wails中文文档](https://wails.io/zh-Hans/docs/reference/options)

2. [源码仓库](https://github.com/wailsapp/wails)

   