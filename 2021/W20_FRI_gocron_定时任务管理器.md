# Golang | gocron介绍

##### 推荐编辑：laocheng

## 一、简介 
很多时候，项目中需要用到定时任务，或者周期任务。首先我们想到了crontab，但是crontab使用会和项目分离，不那么方便。那怎么办呢？

这里，推荐使用`gocron` 这个库，可以非常方便的设置定时任务，无论年、月、日、时、分、秒的定时设置。同时，它还提供crontab字符串格式的设置功能。

## 二、快速使用
步骤如下：
> 1. 安装包"github.com/go-co-op/gocron"
> 2. 引入包，参照例子直接使用


### 1. 简单使用

> 1. 首先，初始化s对象；
> 2. 然后，直接配置定时任务，任务添加函数名+参数；
> 3. 最后，block当前进程，观察任务执行。

```go
package main

import (
	"fmt"
	"time"

	"github.com/go-co-op/gocron"
)

func task(s string){
	fmt.Printf("I'm running, about %s. \n", s)
}

func main() {
	s := gocron.NewScheduler(time.UTC)
	s.Every(1).Seconds().Do(task, "1s")
	s.StartBlocking()
}

```
输出如下：
```
I'm running, about 1s. 
I'm running, about 1s. 
I'm running, about 1s. 
```

### 2. 更多参考设置

针对定时任务，可以设置 时、分、秒、天、周、几点，也可以crontab字符串的格式设置。
```go
package main

import (
	"fmt"
	"time"
	
	"github.com/go-co-op/gocron"
)

func task(){
	fmt.Printf("I'm running.\n")
}

func main() {
	s := gocron.NewScheduler(time.UTC)

	// 每隔多久
	s.Every(1).Seconds().Do(task)
	s.Every(1).Minutes().Do(task)
	s.Every(1).Hours().Do(task)
	s.Every(1).Days().Do(task)
	s.Every(1).Weeks().Do(task)

	// 每周几
	s.Every(1).Monday().Do(task)
	s.Every(1).Thursday().Do(task)

	// 每天固定时间
	s.Every(1).Days().At("10:30").Do(task)
	s.Every(1).Monday().At("18:30").Do(task)
	s.Every(1).Tuesday().At("18:30:59").Do(task)

	// 设置crontab字符串格式
	s.Cron("*/1 * * * *").Do(task)

	s.StartBlocking()
}

```

## 总结

`gocron`库简单方便，支持方式多样，更多可自行挖掘一下；总之，当需要定时任务时，它会给予你非常大的帮助。

## 参考资料

* [https://github.com/go-co-op/gocron](https://github.com/go-co-op/gocrono)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)





















