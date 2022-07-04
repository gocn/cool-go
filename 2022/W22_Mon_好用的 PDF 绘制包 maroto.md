# 好用的 PDF 绘制包 maroto

## 前言

根据 API 数据绘制生成 PDF 文件并不容易实现，但 [maroto](https://github.com/johnfercher/maroto)  库解决了这并不容易的问题，现在用 [maroto](https://github.com/johnfercher/maroto) 包绘制 PDF 文件，就跟用 CSS 写网页样式一样容易；用来绘制简历，发票，证书非常方便。

下面我们就用 [maroto](https://github.com/johnfercher/maroto) 绘制 [GoCN 每日新闻](https://v.gd/qdUKrD)的 PDF 版本，最终效果如下：
![](media/16547776668123/16569433035570.jpg)
![16569433035570.jpg](https://cdn.gocn.vip/forum-user-images/20220704/fd1dc2c591d7467280d752d11faca36b.jpg)

## 创建项目

创建一个新文件夹，把它命名为 gocn-news:
```shell
# 创建项目目录
mkdir gocn-news
# 进入项目根目录
cd gocn-news
# 创建保存图片素材的文件夹
mkdir images
# 创建保存字体的文件夹
mkdir fonts
# 创建保存 PDF 的文件夹
mkdir pdfs
# 初始化项目
go mod init gocn-news
```

> logo 图片来着  [GoCN 每日新闻](https://v.gd/qdUKrD) 中。

然后安装 [maroto](https://github.com/johnfercher/maroto)  包：
```shell
go get -u github.com/johnfercher/maroto
go get github.com/johnfercher/maroto/internal
```

## 编写代码

在 gocn-news 目录中创建 `main.go` 文件，开始编写相关代码：

```go
package main

import (
	"fmt"
	"os"
	"time"

	"github.com/johnfercher/maroto/pkg/color"
	"github.com/johnfercher/maroto/pkg/consts"
	"github.com/johnfercher/maroto/pkg/pdf"
	"github.com/johnfercher/maroto/pkg/props"
)

func main() {
	// 创建一个文件实例，对它进行绘制
	m := pdf.NewMaroto(consts.Portrait, consts.A4)
	// 设置页面布局，left，top，right
	m.SetPageMargins(20, 10, 20)
	// 设置中文字体，默认是英文字体无法输出中文，要替换成支持中文的字体
	m.AddUTF8Font("nst", "bold", "/fonts/nst.ttf")

	// 页面设计，
	// 设计头部
	buildHeader(m)
	// 设计主体内容部分
	buildNewsList(m)
	// 设计附加信息
	buildEditors(m, "张继瑀")
	// 设计底部
	buildFooter(m)

	// PDF 文件保存并关闭资源
	err := m.OutputFileAndClose("pdfs/gocn.pdf")
	if err != nil {
		fmt.Println("PDF 文件保存失败: ", err)
		os.Exit(1)
	}
	fmt.Println("PDF 文件保存成功!")
}
```

> 提示：实际业务开发中数据可以数据库中获取，在此为了简化 Demo 数据就地获取了。

页面构成线框已构成，开始实现它们：
```go
func buildHeader(m pdf.Maroto) {
   // 注册头部容器
	m.RegisterHeader(func() {
	   // 设置水平，垂直布局大小
		m.Row(20, func() {
			m.Col(12, func() {
				err := m.FileImage("images/top.jpeg", props.Rect{
				  // 是否居中，显示占比 80%
					Center:  true,
					Percent: 80,
				})

				if err != nil {
					fmt.Println("图片加载失败！")
				}
			})
		})
	})

	m.Row(10, func() {
		title := "GoCN 每日新闻(" + string(time.Now().Format("2006-01-02")) + ")"
		m.Col(12, func() {
			m.Text(title, props.Text{
				Top:    3,
				Family: "nst",
				Style:  "bold",
				Size:   15,
				Align:  consts.Center,
			})
		})
	})
	
}

type News struct {
	ID    string
	Title string
	URL   string
}

func generateNewsList() [][]string {
	list := []News{
		{ID: "1", Title: "微服务追踪SQL(支持Isto管控下的gorm查询追踪)", URL: "https://v.gd/a71aHJ"},
		{ID: "2", Title: "代码优化的三重境界", URL: "https://mp.weixin.qq.com/s/QByaR4dsq7HzdyU7g9tp3w"},
		{ID: "3", Title: "Golang 数据库 Bolt 碎片整理", URL: "https://v.gd/HFZnAs"},
		{ID: "4", Title: "一图读懂k8s informer client-g", URL: "https://v.gd/aniENY"},
		{ID: "5", Title: "如何用Docker容器化Golang应用程序的开发和生产", URL: "https://v.gd/SM2w9t"},
	}
	
	var newsList [][]string
	for _, v := range list {
		news := make([]string, 0)
		news = append(news, v.ID)
		news = append(news, v.Title)
		news = append(news, v.URL)
		newsList = append(newsList, news)
	}

	return newsList
}

func buildNewsList(m pdf.Maroto) {
	tableHeadings := []string{"编号", "题目", "链接"}
	contents := generateNewsList()
	lightYellowColor := getLightYellowColor()

	m.SetBackgroundColor(getLightYellowColor())
	subTitle := "今日精彩新闻列表"
	// 新闻列表的布局
	m.Row(13, func() {
		m.Col(12, func() {
			m.Text(subTitle, props.Text{
				Top:    2,
				Size:   11,
				Color:  color.NewBlack(),
				Family: "nst",
				Style:  "bold",
				Align:  consts.Center,
			})
		})
	})

	m.SetBackgroundColor(color.NewWhite())
	m.TableList(tableHeadings, contents, props.TableList{
		HeaderProp: props.TableListContent{
			Size:      9,
			GridSizes: []uint{1, 5, 6}, // 表格列宽大小比列
			Color:     color.NewBlack(),
			Family:    "nst",
			Style:     "bold",
		},
		ContentProp: props.TableListContent{
			Size:      8,
			GridSizes: []uint{1, 5, 6},
			Color:     color.NewBlack(),
			Family:    "nst",
			Style:     "bold",
		},
		Align:                consts.Left,
		AlternatedBackground: &lightYellowColor,
		HeaderContentSpace:   1,
		Line:                 false,
	})

}

func buildEditors(m pdf.Maroto, name string) {
	editor := "编辑：" + name
	m.Row(12, func() {
		m.Col(12, func() {
			m.Text(editor, props.Text{
				Top:    2,
				Size:   9,
				Color:  color.NewBlack(),
				Family: "nst",
				Style:  "bold",
				Align:  consts.Left,
			})
		})
	})

	newsURL := "订阅新闻: http://tinyletter.com/gocn"
	m.Row(10, func() {
		m.Col(12, func() {
			m.Text(newsURL, props.Text{
				Top:    2,
				Size:   9,
				Color:  color.NewBlack(),
				Family: "nst",
				Style:  "bold",
				Align:  consts.Left,
			})
		})
	})

	jobsURL := "招聘专区: https://gocn.vip/jobs"
	m.Row(10, func() {
		m.Col(12, func() {
			m.Text(jobsURL, props.Text{
				Top:    2,
				Size:   9,
				Color:  color.NewBlack(),
				Family: "nst",
				Style:  "bold",
				Align:  consts.Left,
			})
		})
	})

}

func getLightYellowColor() color.Color {
	return color.Color{
		Red:   239,
		Green: 173,
		Blue:  60,
	}
}

func buildFooter(m pdf.Maroto) {
	m.RegisterFooter(func() {
		m.Row(165, func() {
			m.Col(12, func() {
				err := m.FileImage("images/footer.jpeg", props.Rect{
					Center:  true,
					Percent: 100,
				})
				if err != nil {
					fmt.Println("图片加载失败！")
				}
			})
		})
	})
}
```

## 运行代码

执行 `go run main.go` 运行成功的话在 pdfs 目录中生成 PDF 文件，运行失败在命令行给出提示。

用一个简单 Demo 展示是 maroto 的基本使用，若要觉得有趣访问 [官网](https://github.com/johnfercher/maroto) 了解更多。

## 总结

[maroto](https://github.com/johnfercher/maroto) 是很好用的 PDF 绘制工具，如果你有 PDF 绘制相关应用开发需求不妨试一试它。

## 相关资料

- [maroto](https://github.com/johnfercher/maroto)
- [go.dev](https://v.gd/0ZD3GM)



