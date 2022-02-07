# 强大的字符表格生成器 tablewriter

## 前言

日常工作中，我们有时需要以字符表格的形式向控制台输出一些结构化的数据，以实现类似 mysql 客户端在控制台输出查询结果的展示效果。那在 go 里如何实现这样的功能呢？也许大家可以试试 `tablewriter`。

## 简介

`tablewriter`(github.com/olekukonko/tablewriter) 是一款功能非常强大的字符表格生成工具，它不仅能够帮助我们实现单元格自动填充、边框绘制等通用能力，还支持设置对齐方式、单元格合并、定制分隔符等高级功能。

以下是 `tablewriter` 的主要特性，它甚至包含了我们日常操作 Excel 时常用的一些基础功能：

* 单元格自动填充和等宽对齐
* 单元格多行支持
* 支持对齐方式设定
* 支持自定义分隔符、表头、边框等
* 自动对齐数字和百分数
* 以 io.Writer 的形式直接输出内容
* 从 CSV 中直接加载数据
* 支持自定义脚注
* 支持单元格合并

## 使用举例

### 基本表格输出

```go
package main

import (
	"os"

	"github.com/olekukonko/tablewriter"
)

func main() {
	data := [][]string{
		{"A", "北京冬奥会 666", "100"},
		{"B", "Happy New Year 2022!", "200"},
	}

	table := tablewriter.NewWriter(os.Stdout)
	table.SetHeader([]string{"Name", "Sign", "Rating"})

	for _, v := range data {
		table.Append(v)
	}
	table.Render()
}
```

控制台（等宽字体）输出：

![简单表格输出](https://cdn.gocn.vip/forum-user-images/20220207/04955730997b47198cf9f9c9b30da9ac.jpg)

从以上输出中我们看到，`tablewriter` 不仅帮我们打印出了字符表格，还实现了中英文字符单元格的等宽对齐！

### 单元格合并举例

```go
package main

import (
	"os"

	"github.com/olekukonko/tablewriter"
)

func main() {
	data := [][]string{
		{"A", "北京冬奥会 666", "100"},
		{"A", "北京冬奥会真棒", "150"},
		{"B", "Happy New Year 2022!", "200"},
		{"B", "开工大吉！", "300"},
	}

	table := tablewriter.NewWriter(os.Stdout)
	table.SetHeader([]string{"Name", "Sign", "Rating"})
	// 合并第一列内容相同的单元格
	table.SetAutoMergeCellsByColumnIndex([]int{0})
	table.SetRowLine(true)

	table.AppendBulk(data)

	table.Render()
}
```

控制台（等宽字体）输出：

![合并单元格](https://cdn.gocn.vip/forum-user-images/20220207/804274c16abf46948a1046c811655ac6.jpg)

可见，第一列内容相同的相邻单元格成功实现了合并！

## 总结

`tablewriter` 是一款功能强大的字符表格绘制工具，熟练掌握 `tablewriter` 的使用技巧可以方便地绘制出漂亮的表格输出，很适用于集成到命令行工具中去，大家快试试吧！

## 参考资料

* [github.com/olekukonko/tablewriter](github.com/olekukonko/tablewriter)
* [https://pkg.go.dev/github.com/olekukonko/tablewriter](https://pkg.go.dev/github.com/olekukonko/tablewriter)
* [github.com/mattn/go-runewidth](github.com/mattn/go-runewidth)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
