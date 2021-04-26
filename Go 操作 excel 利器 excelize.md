# Go 操作 excel 利器 excelize

## excelize 简介

Excelize是一个用Go语言编写的库，提供了一组允许您写入和读取XLSX / XLSM / XLTM文件的功能。 支持读写由Microsoft Excel™2007和更高版本生成的电子表格文档。 通过高度兼容性支持复杂的组件，并提供了流式API，用于从工作表中生成或读取包含大量数据的数据。 该库需要Go版本1.15或更高版本。

## 应用场景分析

这个库基本上不用多说什么了吧！

对于非专业人士来说，可用于处理 excel 数据，功能强大，解放双手！

对于专业程序员来说，如可以增加我们应用程序中的 excel 导入导出功能！

## 快速使用 excelize

### 安装 excelize

```shell
go get github.com/360EntSecGroup-Skylar/excelize
```

**如果你使用go module**，使用下面的安装命令。

```shell
go get github.com/360EntSecGroup-Skylar/excelize/v2
```

### 入门案例

首先需要明白的是 excel 中唯一确定单元格需要两个参数：

* sheet，默认是 sheet1
* 坐标

<img src="http://picture.nj-jay.com/image-20210422133727910.png" alt="image-20210422133727910" style="zoom:80%;" />

写入到excel

```go
package main

import (
    "fmt"

    "github.com/360EntSecGroup-Skylar/excelize/v2"
)

func main() {
    f := excelize.NewFile()
    // 创建一个 sheet
    index := f.NewSheet("Sheet2")
    // 向单元格中写入值
    f.SetCellValue("Sheet2", "A2", "gocn")
    f.SetCellValue("Sheet1", "B2", 100)
    // 设置文件打开后显示哪个 sheet， 0 表示 sheet1
    f.SetActiveSheet(index)
    // 保存到文件
    if err := f.SaveAs("Book1.xlsx"); err != nil {
        fmt.Println(err)
    }
}
```

读取 excel 中的内容

```go
package main

import (
	"fmt"

	"github.com/360EntSecGroup-Skylar/excelize/v2"
)

func main() {
	//打开文件
	f, err := excelize.OpenFile("Book1.xlsx")
	if err != nil {
		fmt.Println(err)
		return
	}
	// 获取单元格内容
	cell, err := f.GetCellValue("Sheet1", "B2")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(cell)
	// 获取sheet1中所有的行
	rows, err := f.GetRows("Sheet1")
	for _, row := range rows {
		for _, colCell := range row {
			fmt.Print(colCell, "\t")
		}
		fmt.Println()
	}
}
```

## 进阶使用 excelize

如果以上功能仍然不能满足于你，不用担心！

因为 excelize 远远不止以上功能，它还提供了：

* 添加图表
* 添加图片

### 向 excel 中添加图表

```go
package main

import (
    "fmt"

    "github.com/360EntSecGroup-Skylar/excelize/v2"
)

func main() {
    categories := map[string]string{
        "A2": "Small", "A3": "Normal", "A4": "Large",
        "B1": "Apple", "C1": "Orange", "D1": "Pear"}
    values := map[string]int{
        "B2": 2, "C2": 3, "D2": 3, "B3": 5, "C3": 2, "D3": 4, "B4": 6, "C4": 7, "D4": 8}
    f := excelize.NewFile()
    for k, v := range categories {
        f.SetCellValue("Sheet1", k, v)
    }
    for k, v := range values {
        f.SetCellValue("Sheet1", k, v)
    }
    //添加图表
    if err := f.AddChart("Sheet1", "E1", `{
        "type": "col3DClustered",
        "series": [
        {
            "name": "Sheet1!$A$2",
            "categories": "Sheet1!$B$1:$D$1",
            "values": "Sheet1!$B$2:$D$2"
        },
        {
            "name": "Sheet1!$A$3",
            "categories": "Sheet1!$B$1:$D$1",
            "values": "Sheet1!$B$3:$D$3"
        },
        {
            "name": "Sheet1!$A$4",
            "categories": "Sheet1!$B$1:$D$1",
            "values": "Sheet1!$B$4:$D$4"
        }],
        "title":
        {
            "name": "Fruit 3D Clustered Column Chart"
        }
    }`); err != nil {
        fmt.Println(err)
        return
    }
    
    if err := f.SaveAs("Book1.xlsx"); err != nil {
        fmt.Println(err)
    }
}
```

![image-20210422141005980](http://picture.nj-jay.com/image-20210422141005980.png)

### 向 excel 中添加图片

```go
package main

import (
	"fmt"
	_ "image/gif"
	_ "image/jpeg"
	_ "image/png"

	"github.com/360EntSecGroup-Skylar/excelize/v2"
)

func main() {
	f, err := excelize.OpenFile("Book1.xlsx")
	if err != nil {
		fmt.Println(err)
		return
	}
	// Insert a picture.
	if err := f.AddPicture("Sheet1", "A2", "logo.png", ""); err != nil {
		fmt.Println(err)
	}
	// Save the spreadsheet with the origin path.
	if err = f.Save(); err != nil {
		fmt.Println(err)
	}
}
```

![image-20210422141634536](http://picture.nj-jay.com/image-20210422141634536.png)

## 参考资料

* https://github.com/360EntSecGroup-Skylar/excelize