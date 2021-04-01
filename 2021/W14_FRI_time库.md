# Golang | time库

##### 推荐编辑：laocheng

## 一、简介 

开发中会碰到各种各样的时间和日期问题，比如获取当前时间、格式转换、设置timer等等。本期，我们看看time包的使用方法

## 二、应用场景
> 1. 获取当前各种时间
> 2. 各种时间差计算
> 3. 时间戳格式转换
> 4. 各种时间格式化输出
> 5. 时区问题
> 6. 定时器设置


## 时间结构
```go
type Time struct {
	wall uint64
	ext  int64

	loc *Location
}
```
说明：
* wall: 距离公元1年1月1日 00:00:00UTC 的秒数
* ext: 纳秒
* loc: 时区，不同的时区，对应的时间不一样




## 1. 获取当前时间
可以获取当前时间，及对应的 年、月、日、时、分、秒，甚至星期几。
```go
package main
import (
	"fmt"
	"time"
)
func main() {
	now := time.Now() //获取当前时间
	fmt.Printf("当前时间：%v\n", now)

	fmt.Printf(
		"当前时间：%d-%02d-%02d %02d:%02d:%02d\n",
		now.Year(),
		now.Month(),
		now.Day(),
		now.Hour(),
		now.Minute(),
		now.Second())

	fmt.Printf("今天星期几：%s\n", now.Weekday().String())
}
```
输出如下：
```
当前时间：2021-04-01 16:22:58.678702 +0800 CST m=+0.000078825
当前时间：2021-04-01 16:22:58
今天星期几：Thursday
```

## 2. 各种时间差计算
包括对Add、Sub、Equel、After、Before几个时间差函数的使用。
```go
package main
import (
	"fmt"
	"time"
)
func main() {
	now := time.Now()
	fmt.Printf("当前时间：%v \n", now)

	later := now.Add(time.Hour)  // 1小时后
	fmt.Printf("一小时后：%v \n", later)

	diff := later.Sub(now)  // 计算时间差
	fmt.Printf("时间差值：%s \n", diff.String())

	bl := later.Equal(now)  // 判断时间相等
	fmt.Printf("later 等于 now 吗？%t \n", bl)

	bl = later.After(now)  // 判断在某时间之后
	fmt.Printf("later 在 now 之后吗？%t \n", bl)

	bl = later.Before(now)  // 判断在某时间之前
	fmt.Printf("later 在 now 之前吗？%t \n", bl)
}
```
输出如下：
```
当前时间：2021-04-01 16:36:22.774417 +0800 CST m=+0.000090199 
一小时后：2021-04-01 17:36:22.774417 +0800 CST m=+3600.000090199 
时间差值：1h0m0s 
later 等于 now 吗？false 
later 在 now 之后吗？true 
later 在 now 之前吗？false 
```


## 3. 时间戳、时间格式转换
包括获取时间戳，Time转字符串，字符串转Time。
```go
package main
import (
	"fmt"
	"time"
)
func main() {
	now := time.Now()

	sec := now.Unix()
	fmt.Printf("当前时间秒数：%v \n", sec)
	nano := now.UnixNano()
	fmt.Printf("当前时间纳秒数：%v \n", nano)

	nowString := now.Format("2006-01-02 15:04")
	fmt.Printf("time转字符串：%s \n", nowString)

	timeConst := "2006-01-02 15:04:05"
	stTime, _ := time.Parse(timeConst, "2013-10-05 18:30:50")
	fmt.Printf("字符串转time：%s \n", stTime)
}
```
输出如下：
```
当前时间秒数：1617267862 
当前时间纳秒数：1617267862497198000 
time转字符串：2021-04-01 17:04 
字符串转time：2013-10-05 18:30:50 +0000 UTC 
```


## 4. 各种时间格式化输出
包括获取时间戳，Time转字符串，字符串转Time。
```go
package main
import (
	"fmt"
	"time"
)
func main() {
	// 格式化的模板，是从Go的出生时间2006年1月2号15点04分 Mon Jan
	now := time.Now()
	fmt.Println(now.Format("2006-01-02 15:04:05.000"))  // 24小时制
	fmt.Println(now.Format("2006-01-02 03:04:05.000 PM"))  // 12小时制
	fmt.Println(now.Format("2006/01/02 15:04"))
	fmt.Println(now.Format("15:04 2006/01/02"))
	fmt.Println(now.Format("2006/01/02"))
}
```
输出如下：
```
2021-04-01 17:16:55.695
2021-04-01 05:16:55.695 PM
2021/04/01 17:16
17:16 2021/04/01
2021/04/01
```


## 5. 时区问题
包括获取时间戳，Time转字符串，字符串转Time。
```go
package main
import (
	"fmt"
	"time"
)
func main() {
	now := time.Now()

	loc := now.Location()
	fmt.Printf("时区：%v \n", loc)

	utcLoc := now.UTC().Location()
	fmt.Printf("utc 时区：%v \n", utcLoc)
}
```
输出如下：
```
时区：Local 
utc 时区：UTC 
```




## 6. 定时器设置
包括获取时间戳，Time转字符串，字符串转Time。
```go
package main
import (
	"fmt"
	"time"
)
func main() {
	now := time.Now()

	loc := now.Location()
	fmt.Printf("时区：%v \n", loc)

	utcLoc := now.UTC().Location()
	fmt.Printf("utc 时区：%v \n", utcLoc)
}
```
输出如下：
```
时区：Local 
utc 时区：UTC 
```























