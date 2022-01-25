# Golang 图片处理 — image 库

在开发中，有时会遇到对图片的处理需求，在 Python中， PIL/Pillow  库非常强大和易用。

而 Golang 语言中，处理图片的标准库 `image`也可以实现一些基本操作。

`image` 库支持常见的 PNG、JPEG、GIF 等格式的图片处理， 可以对图片进行读取、裁剪、绘制、生成等操作。

### 读取、新建图片

#### 读取

图片的读取，和文件的读取类似，只需要使用 `os.Open()`函数，获取一个输入流，然后将数据流进行解码操作。

需要注意的是，在解码阶段，要将不同类型的图片的解码器先进行注册，这样才不会报` unknown format` 的错误。

```go
package main

import (
	"fmt"
	"image"
	_ "image/png"
	"os"
)

func main() {
	f, err := os.Open("./gopher.png")
	if err != nil {
		panic(err)
	}
	img, formatName, err := image.Decode(f)
	if err != nil {
		panic(err)
	}
	fmt.Println(formatName)
	fmt.Println(img.Bounds())
	fmt.Println(img.ColorModel())
}
```

`Decode` 方法返回的第一个值是一个 `image.Image`类型接口。不同的颜色模型的图片返回不同类型的值。该接口有三个方法：

```go
type Image interface {
  ColorModel() color.Model // 返回图片的颜色模型
  Bounds() Rectangle		   // 返回图片的长宽
  At(x, y int) color.Color // 返回(x,y)像素点的颜色
}
```

image 库中很多结构都实现了该接口，对于一些标准库中没有实现的功能，我们也可以自己实现该接口去满足。

#### 新建

如果是需要新建一个图片，可以使用`image.NewRGBA()`方法。

```go
img := image.NewRGBA(image.Rect(0, 0, 300, 300))
```

这里的 `NewRGBA`方法返回的是一个实现了 `image.Image`接口的 `image.RGBA`类型数据。 这里是一个300*300的透明背景的图片。

### 保存图片

保存图片和保存文件也类似，需要先将图片编码，然后以数据流的形式写入文件。

```go
img := image.NewRGBA(image.Rect(0, 0, 300, 300))

outFile, err := os.Create("gopher2.png")
defer outFile.Close()
if err != nil {
  panic(err)
}
b := bufio.NewWriter(outFile)
err = png.Encode(b, img)
if err != nil {
  panic(err)
}
err = b.Flush()
if err != nil {
  panic(err)
}
```

### 裁剪图片

图片的裁剪主要使用`SubImage()`方法，如下：

```go
img := image.NewRGBA(image.Rect(0, 0, 300, 300))
subImage := img.SubImage(image.Rect(0, 0, 20, 20))
```

该方法将从创建的300 * 300的图片裁剪出20 * 20 像素的子图片。

### 绘制图片

绘制图片主要使用到 `draw.Draw`和`draw.DrawMask`方法。

```go
func Draw(dst Image, r image.Rectangle, src image.Image, sp image.Point, op Op)

func DrawMask(dst Image, r image.Rectangle, src image.Image, sp image.Point, mask image.Image, mp image.Point, op Op)

```

#### Draw

`Draw`方法各个参数含义如下：

- ***dst***  绘图的背景图
- ***r***  背景图的绘图区域
- ***src***  要绘制的图
- ***sp***  src 对应的绘图开始点
- ***op*** 组合方式

以下代码是将一个 Gopher 的图案绘制到了一张黑色背景空白图的左上角。

```go
f, err := os.Open("./gopher.png")
if err != nil {
  panic(err)
}
gopherImg, _, err := image.Decode(f) // 打开图片

img := image.NewRGBA(image.Rect(0, 0, 300, 300))
for x := 0; x < img.Bounds().Dx(); x++ {    // 将背景图涂黑
  for y := 0; y < img.Bounds().Dy(); y++ {
    img.Set(x, y, color.Black)
  }
}
draw.Draw(img, img.Bounds(), gopherImg, image.Pt(0, 0), draw.Over) // 将gopherImg绘制到背景图上
```

![](https://gocn.oss-cn-shanghai.aliyuncs.com/photo/bazinga/aca901cb-d79a-4682-9443-5f82096eebd6.png?x-oss-process=image%2Fresize%2Cw_1920) 

#### DrawMask

`DrawMask`方法多了一个遮罩蒙层参数`mask`，以及蒙层的起始位置参数 `mp`。

`Draw`方法是 `DrawMask`的一种特殊形式，当 `DrawMask` 的 `mask` 参数为nil时，即为`Draw`方法。

`DrawMask`将背景图上的绘图区域起始点、要绘制图的起始点、遮罩蒙层的起始点进行对齐，然后对背景图上的绘图矩阵区域执行 [Porter-Duff](https://docs.microsoft.com/zh-tw/xamarin/xamarin-forms/user-interface/graphics/skiasharp/effects/blend-modes/porter-duff)合并操作。

下面是给图片加一个圆形遮罩的示例：

```go
func drawCirclePic() {
	f, err := os.Open("./gopher.png")
	if err != nil {
		panic(err)
	}
	gopherImg, _, err := image.Decode(f)


	d := gopherImg.Bounds().Dx()
	
	//将一个cicle作为蒙层遮罩，圆心为图案中点，半径为边长的一半
	c := circle{p: image.Point{X: d / 2, Y: d / 2}, r: d / 2} 
	circleImg := image.NewRGBA(image.Rect(0, 0, d, d))
	draw.DrawMask(circleImg, circleImg.Bounds(), gopherImg, image.Point{}, &c, image.Point{}, draw.Over)

	SavePng(circleImg)
}

type circle struct { // 这里需要自己实现一个圆形遮罩，实现接口里的三个方法
	p image.Point // 圆心位置
	r int
}

func (c *circle) ColorModel() color.Model {
	return color.AlphaModel
}

func (c *circle) Bounds() image.Rectangle {
	return image.Rect(c.p.X-c.r, c.p.Y-c.r, c.p.X+c.r, c.p.Y+c.r)
}

// 对每个像素点进行色值设置，在半径以内的图案设成完全不透明
func (c *circle) At(x, y int) color.Color {
	xx, yy, rr := float64(x-c.p.X)+0.5, float64(y-c.p.Y)+0.5, float64(c.r)
	if xx*xx+yy*yy < rr*rr {
		return color.Alpha{A: 255}
	}
	return color.Alpha{}
}
```

![](https://gocn.oss-cn-shanghai.aliyuncs.com/photo/bazinga/eebb6f99-7ba6-4d5f-a3cf-dbb3d405f86c.png?x-oss-process=image%2Fresize%2Cw_1920)

给图片加一个圆角遮罩的示例：

```go
func drawRadiusPic() {
	f, err := os.Open("./gopher.png")
	if err != nil {
		panic(err)
	}
	gopherImg, _, err := image.Decode(f)

	w := gopherImg.Bounds().Dx()
	h := gopherImg.Bounds().Dy()

	c := radius{p: image.Point{X: w, Y: h}, r: int(40)}
	radiusImg := image.NewRGBA(image.Rect(0, 0, w, h))
	draw.DrawMask(radiusImg, radiusImg.Bounds(), gopherImg, image.Point{}, &c, image.Point{}, draw.Over)

	SavePng(radiusImg)
}

type radius struct {
	p image.Point // 矩形右下角位置
	r int
}

func (c *radius) ColorModel() color.Model {
	return color.AlphaModel
}

func (c *radius) Bounds() image.Rectangle {
	return image.Rect(0, 0, c.p.X, c.p.Y)
}

// 对每个像素点进行色值设置，分别处理矩形的四个角，在四个角的内切圆的外侧，色值设置为全透明，其他区域不透明
func (c *radius) At(x, y int) color.Color {
	var xx, yy, rr float64
	var inArea bool
	// left up
	if x <= c.r && y <= c.r {
		xx, yy, rr = float64(c.r-x)+0.5, float64(y-c.r)+0.5, float64(c.r)
		inArea = true
	}
	// right up
	if x >= (c.p.X-c.r) && y <= c.r {
		xx, yy, rr = float64(x-(c.p.X-c.r))+0.5, float64(y-c.r)+0.5, float64(c.r)
		inArea = true
	}
	// left bottom
	if x <= c.r && y >= (c.p.Y-c.r) {
		xx, yy, rr = float64(c.r-x)+0.5, float64(y-(c.p.Y-c.r))+0.5, float64(c.r)
		inArea = true
	}
	// right bottom
	if x >= (c.p.X-c.r) && y >= (c.p.Y-c.r) {
		xx, yy, rr = float64(x-(c.p.X-c.r))+0.5, float64(y-(c.p.Y-c.r))+0.5, float64(c.r)
		inArea = true
	}

	if inArea && xx*xx+yy*yy >= rr*rr {
		return color.Alpha{}
	}
	return color.Alpha{A: 255}
}
```



![](https://gocn.oss-cn-shanghai.aliyuncs.com/photo/bazinga/c38eba93-2589-4e1f-9ab4-25eaccf0554e.png?x-oss-process=image%2Fresize%2Cw_1920)

在图案进行圆形、圆角绘制的过程中，因为最小单位是1px，所以可能会有锯齿边缘的问题，解决这个问题可以通过先将原图放大，遮罩后再缩小来解决。

## Reference

[The Go image/draw package - The Go Blog (golang.org)](https://blog.golang.org/image-draw)

[Porter-Duff blend 模式 - Xamarin | Microsoft Docs](https://docs.microsoft.com/zh-tw/xamarin/xamarin-forms/user-interface/graphics/skiasharp/effects/blend-modes/porter-duff)

