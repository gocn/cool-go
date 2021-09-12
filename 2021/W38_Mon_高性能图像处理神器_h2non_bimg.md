# 高性能图像处理神器 h2non/bimg

## 推荐背景

日常业务开发中常会遇到各种图像处理需求，如，图片大小调整、翻转、旋转、提取大小、加水印、图片模糊化，格式转换，修剪等等；图像处理根据业务需求一部分图像处理需求在前端完成,如，用户裁剪编辑，打马赛克，美化，色彩调整;用户传完图片后的逻辑在后端处理，此时就会用到图像处理相关的方法；Go 语言标准库提供的 [image](https://pkg.go.dev/image) 能够处理最基本的图像处理任务，但个性化的图像处理任务还是得自己实现。

当标准库无法满足日常开发的需求时候，推荐使用 [h2non/bimg](https://github.com/h2non/bimg) ，它能够快速高效完成各种高级图像处理任务。

## h2non/bimg 是什么

[h2non/bimg](https://github.com/h2non/bimg) 是基于 C 语言的 [libvips](https://github.com/libvips/libvips) 库封装的高级图像处理的 Go 包；它的性能极高，所占内存也很小，通常比 [ImageMagick](https://github.com/ImageMagick/ImageMagick)， [GraphicsMagick](https://github.com/aheckmann/gm)，[Go 标准库快 4 倍，在某些情况下，处理 JPEG 图像的速度甚至比它们快 8 倍](https://pkg.go.dev/github.com/h2non/bimg#section-readme)。

> bimg uses internally libvips, a powerful library written in C for image processing which requires a low memory footprint and it's typically 4x faster than using the quickest ImageMagick and GraphicsMagick settings or Go native image package, and in some cases it's even 8x faster processing JPEG images.


[h2non/bimg](https://github.com/h2non/bimg) 提供以下出片处理 API：
- 调整大小
- 放大
- 裁剪（包括智能裁剪支持，libvips 8.5+）
- 旋转（根据 EXIF 方向自动旋转）
- 翻转（具有基于EXIF元数据的自动翻转）
- 翻转
- 缩略图
- 获取大小
- 水印（使用文本或图像）
- 高斯模糊效果
- 自定义输出颜色空间（RGB，灰度...）
- 格式转换以及压缩处理
- EXIF元数据（大小，Alpha通道，配置文件，方向...）修改
- 修剪（libvips 8.6+）

[h2non/bimg](https://github.com/h2non/bimg) 能够将图像输出为 JPEG、PNG 和 WEBP 格式，包括在它们之间进行格式转换。

## 怎么使用

[h2non/bimg](https://github.com/h2non/bimg) 因为基于 C 语言的 libvips 库，因此使用要满足以下几个条件：
- [libvips](https://github.com/libvips/libvips) 8.3+ (8.8+ recommended)
- C compatible compiler such as gcc 4.6+ or clang 3.0+
- Go 1.3+

提示：
- GIF、PDF 和 SVG 支持需要 [libvips](https://github.com/libvips/libvips) v8.3+。
- AVIF 支持需要 [libvips](https://github.com/libvips/libvips) v8.9+。 使用 AVIF 编码器/解码器编译的 libheif 也需要存在。 

### 安装

1. 创建一个测试项目

```shell
# 创建项目目录
$ mkdir bimg
# 切换到项目根目录
$ cd bimg
# 初始化项目
$ go mod init bimg

```

2. 获取 h2non/bimg 包

```shell
$ go get -u github.com/h2non/bimg
```

通常会报错，提示需要依赖 libvips ；因此，先安装 libvips，再获取 [h2non/bimg](https://github.com/h2non/bimg) 包。

mac 安装执行 `$ brew install vips` 即可自动加载安装配置；其他系统版本请参考 [libvips document](https://libvips.github.io/libvips/install.html) 。

再次尝试获取 [h2non/bimg](https://github.com/h2non/bimg) 包，通常会提示 `invalid flag in pkg-config --cflags: -Xpreprocessor` ;此时 CGO_FLAGS 授权，即执行一下命令：

```shell
$ export CGO_CFLAGS_ALLOW=-Xpreprocessor
``` 

再次尝试获取 [h2non/bimg](https://github.com/h2non/bimg) 包时，会成功获取包。

### 开始使用

在项目根目录创建 `main.go` 文件，并准备两张测试图片；分别是原始图片 `example.png` ：

![](https://gocn.oss-cn-shanghai.aliyuncs.com/photo/Aklman/449c7b2d-6baf-46d6-be83-b6e65edb21a4.png?x-oss-process=image%2Fresize%2Cw_1920)

以及用作水印的图片 `logo.png` ：

![](https://gocn.oss-cn-shanghai.aliyuncs.com/photo/Aklman/5390a237-acbe-4dfd-94ac-fa1204fa1024.png?x-oss-process=image%2Fresize%2Cw_1920)

开始编写示例：

```go
package main

import (
	"fmt"
	"os"

	"github.com/h2non/bimg"
)

func main() {
	imgPath := "example.png"
	// 打印图像的元数据
	meta_data := metaData(imgPath)
	fmt.Printf("图像类型：%v\n图像尺寸：%v x %v\n", meta_data.Type, meta_data.Size.Width, meta_data.Size.Height)

	// 调整图像大小
	resize(imgPath, 1200, 800)

	// 旋转图像,传旋转角度
	rotate(imgPath, 180)

	// 自动(随机)图像,根据 EXIF 方向元数据(autoRotateOnly)自动旋转
	autoRotate(imgPath)

	// 格式转换
	convert(imgPath, bimg.JPEG)

	// 添加文字水印
	watermarkText(imgPath)

	// 添加图片水印
	logoPath := "logo.png"
	watermarkImage(imgPath, logoPath)

	// 高斯模糊,模糊程度参数
	gaussianBlur(imgPath, 20, 5)

	// 按长宽式缩略
	smartCrop(imgPath, 400, 300)

	// 缩略图,缩略图像素 200 * 200
	thumbnail(imgPath, 200)

	// 获取图像类型
	imgType := getType(imgPath)
	fmt.Printf("图像类型：%v\n", imgType)
}

```

### 打印图像的元数据

```go
func metaData(path string) bimg.ImageMetadata {
	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	metaData, err := bimg.Metadata(buffer)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	return metaData
}
```


### 调整图像大小

```go
func resize(path string, width, height int) {
	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	newImage, err := bimg.NewImage(buffer).Resize(width, height)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	_, err = bimg.NewImage(newImage).Size()

	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	bimg.Write("001-resize.jpg", newImage)
}

```

### 指定角度旋转

```go
func rotate(path string, angle bimg.Angle) {
	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	// 传旋转角度
	newImage, err := bimg.NewImage(buffer).Rotate(angle)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	bimg.Write("002-rotate.png", newImage)
}
```

### 自动旋转

```go
func autoRotate(path string) {
	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	newImage, err := bimg.NewImage(buffer).AutoRotate()
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	bimg.Write("003-auto-rotate.jpg", newImage)
}
```

### 格式转换

```go
func convert(path string, imageType bimg.ImageType) {
	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	newImage, err := bimg.NewImage(buffer).Convert(imageType)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	bimg.Write("004-convert."+bimg.ImageTypes[imageType], newImage)

}
```

### 文字水印

```go
func watermarkText(path string) {
	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}
	// 水印的相关数据
	watermark := bimg.Watermark{
		Text:       "GoCN 社区",
		DPI:        150,
		Margin:     300,
		Opacity:    0.25, // 不透明度
		Width:      500,
		Font:       "sans bold 14",
		Background: bimg.Color{255, 255, 255},
	}

	newImage, err := bimg.NewImage(buffer).Watermark(watermark)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	bimg.Write("005-watermark.jpg", newImage)
}
```

### 图片水印

```go
func watermarkImage(path, logo string) {
	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}
	logoBuffer, err := bimg.Read(logo)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}
	// 水印的相关数据
	watermark := bimg.WatermarkImage{
		Left:    100,
		Top:     200,
		Buf:     logoBuffer,
		Opacity: 1, // 不透明度 0~1 之间的浮点数
	}

	newImage, err := bimg.NewImage(buffer).WatermarkImage(watermark)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	bimg.Write("006-watermark-image.jpg", newImage)
}
```

### 高斯模糊

```go
func gaussianBlur(path string, sigma, minAmpl float64) {

	options := bimg.Options{
		GaussianBlur: bimg.GaussianBlur{sigma, minAmpl},
	}

	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	newImage, err := bimg.NewImage(buffer).Process(options)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	bimg.Write("007-gaussianblur.jpg", newImage)
}
```

### 指定长宽缩略图像

```go
func smartCrop(path string, width, height int) {
	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	newImage, err := bimg.NewImage(buffer).SmartCrop(width, height)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	bimg.Write("008-smartCrop.jpg", newImage)
}
```

### 缩略图像

```go
func thumbnail(path string, pixels int) {
	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	newImage, err := bimg.NewImage(buffer).Thumbnail(pixels)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	bimg.Write("009-thumbnail.jpg", newImage)
}
```


### 获取类型

```go
func getType(path string) string {
	buffer, err := bimg.Read(path)
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
	}

	typeName := bimg.NewImage(buffer).Type()

	return typeName
}
```

### 运行项目

执行以下命令：

```shell
$ go run main.go
```

执行成功，项目目录下生成图像处理结果，效果如下：
![](https://gocn.oss-cn-shanghai.aliyuncs.com/photo/Aklman/06930d86-4dd4-4dc9-afca-24ea1611ae3d.png?x-oss-process=image%2Fresize%2Cw_1920)


想了解[更多](https://pkg.go.dev/github.com/h2non/bimg#section-documentation)。

 ## 总结
 
[h2non/bimg](https://github.com/h2non/bimg) 目前 Github Go 图像处理类包中 Star 数量最多，高性能的图像处理包； [h2non/bimg](https://github.com/h2non/bimg) 足够应付日常图像处理类业务需求，使用很方便，也可以在改包基础上功能扩展开发满足更多个性化需求。

## 参考资料

1. [https://github.com/h2non/bimg](https://github.com/h2non/bimg)
2. [https://pkg.go.dev/github.com/h2non/bimg](https://pkg.go.dev/github.com/h2non/bimg)
 
 
