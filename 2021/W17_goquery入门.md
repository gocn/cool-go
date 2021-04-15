# goquery 入门

## 一、简介

### 什么是goquery？

goquery是由Go实现的基于Go的net/html包和CSS选择器库cascadia的HTML解析库。

`由于net/html解析器需要UTF-8编码，goquery也同样需要，所以需要确保提供的html是UTF-8编码。`

### 为什么用goquery？

由于net/html解析器返回的是节点，而不是功能齐全的DOM树，所以在使用的过程中goquery可以提供更便利的操作。

## 二、快速上手

我们先对微博热搜进行一个简单的解析，打印当日的热搜排名标题以及热度。

```go
package main

import (
	"fmt"
	"github.com/PuerkitoBio/goquery"
	"log"
	"net/http"
)

type Data struct {
	number string
	title  string
	heat   string
}

func main() {
	// 爬取微博热搜网页
	res, err := http.Get("https://s.weibo.com/top/summary")
	if err != nil {
		log.Fatal(err)
	}
	defer res.Body.Close()
	if res.StatusCode != 200 {
		log.Fatalf("status code error: %d %s", res.StatusCode, res.Status)
	}
	//将html生成goquery的Document
	dom, err := goquery.NewDocumentFromReader(res.Body)
	if err != nil {
		log.Fatalln(err)
	}
	var data []Data
	// 筛选class为td-01的元素
	dom.Find(".td-01").Each(func(i int, selection *goquery.Selection) {
		data = append(data, Data{number: selection.Text()})
	})
	// 筛选class为td-02的元素下的a元素
	dom.Find(".td-02>a").Each(func(i int, selection *goquery.Selection) {
		data[i].title = selection.Text()
	})
	// 筛选class为td-02的元素下的span元素
	dom.Find(".td-02>span").Each(func(i int, selection *goquery.Selection) {
		data[i].heat = selection.Text()
	})
	fmt.Println(data)
}


```

打印结果为：

```
[{ 网络安全有你有我 3026574} {1 任豪道歉 1635306} {2 刘鑫方称对江歌遇害不担责 1398449} {3 小米折叠屏开售 977369} {4 姜素拉产女 952863} {5 17元吃海底捞 718468} {6 那英 我只配给沈腾杨洋伴唱 634544} {7 日本船东要求埃及打一折 575670} {8 癌症早期几乎无症状 547113} {9 央行称要重视理工科教育 540756} {10 李宁回应天价鞋 540379} {11 任豪 外网 537050} {12 谢楠汪聪主持山河令演唱会 531075} {13 偶遇张小斐为李焕英扫墓 506813} {14 FBI
每10小时启动一项对中国的新调查 503616} {15 意大利海域遭水母入侵 495294} {16 山河令演唱会票价 486933} {17 刘以豪被娜扎亲出口红印 484077} {18 吴前 473968} {19 科学家成功捕获黑洞多波段指纹 468128} {20 宋佳袁咏仪发海报裁掉对方 462762} {21 郑州一中学通报女学生坠楼事件 453126} {22 72年前她身穿旗袍运送绝密情报 451299} {23 张雨绮李柄熹 娱乐圈姐弟恋甜宠文 414856} {24 多肉杨梅奶茶 397879} {25 创4总决赛绘画撑腰大赛 380987} {26 中
学小卖部承包3年拍出320万 347615} {27 北京沙尘暴 325600} {28 罗永浩再回应被强制执行 313570} {29 在营P2P网贷机构全部停业 311420} {30 多名教师殴打幼儿被园方辞退 294579} {31 我国连续4年多未发生暴恐案事件 285303} {32 蜘蛛侠3中文片名 282589} {33 娜扎护士装造型 278113} {34 王霜说在巴黎不被尊重 267191} {35 王佑硕鼻子 250443} {36 吴磊一镜到底哭戏 249606} {37 秦岭大熊猫上树折樱桃花 249071} {38 警方通报沈阳1死2伤持刀伤人案 248500
} {39 2020年中国群众安全感指数98.4% 246917} {40 你见过哪些海王操作 246148} {41 商务部回应日本处置福岛核废水 244992} {42 一组数据看平安中国 244345} {43 赵文瑄 243697} {44 内蒙古沙尘暴风沙夹杂雨雪 242044} {45 油价或迎年内首次搁浅 241157} {46 库里的进攻能力在NBA排第几 239944} {47 在家长群抢30个红包被拘 239808} {48 明白做老师不易的瞬间 207204} {49 张哲瀚OK周年刊封面 206645} {50 轻混血浓颜泰妆 }]
```

## 三、过滤器示例

### 基于HTML Element 元素的选择器

使用Element名称作为选择器，如dom.Find("div")。

```go
dom.Find("div").Each(func (i int, selection *goquery.Selection) {
fmt.Println(selection.Text())
})
```

### ID选择器

以#加id值作为选择器

```go
dom.Find("#id").Each(func (i int, selection *goquery.Selection) {
fmt.Println(selection.Text())
})
```

### Class选择器

以.加class值为选择器

```go
dom.Find(".class").Each(func (i int, selection *goquery.Selection) {
fmt.Println(selection.Text())
})
```

由上面的示例可以看出，goquery的选择器与jQuery的选择器用法无异，在这里就不继续赘述了，同学们可以自行探索。

## 四、常用节点属性值

### Html() 获取该节点的html

```go
dom.Find("table").Each(func (i int, selection *goquery.Selection) {
fmt.Println(selection.Html())
})
```

### Text() 获取该节点的文本值

```go
dom.Find(".td-02>a").Each(func (i int, selection *goquery.Selection) {
fmt.Println(selection.Text())
})
```

### Attr() 返回节点的属性值以及该属性是否存在的布尔值

```go
dom.Find("#execution").Each(func (i int, selection *goquery.Selection) {
value[i], ok = selection.Attr("value")
})
```

### Length() 返回该Selection的元素个数

```go
dom.Find("td").Length()
```

## 闲言

这个库用起来非常容易上手，尤其是对jQuery熟悉的同学，极大地提升了开发效率。

之前都是用Python写爬虫，使用BeautifulSoup进行解析，十分方便。第一次用go的net/html尝试解析的时候，花了很多不必要的时间，直到后来发现了goquery，才找回了用BeautifulSoup的感觉。

## 友情提示：爬虫有风险，请务必遵守法律法规。

## 参考文档：

- https://github.com/PuerkitoBio/goquery
- https://www.flysnow.org/2018/01/20/golang-goquery-examples-selector.html

欢迎加入我们GOLANG中国社区：https://gocn.vip/