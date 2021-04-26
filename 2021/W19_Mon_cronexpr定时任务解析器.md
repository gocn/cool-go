# Golang | cronexpr库

简介 什么是cronexpr：

cronexpr是一个高级的crontab解析表达式的库，用于解析比原生crontab更好强大的定时任务解析器，当然，这个包是不包含定时任务功能的

## 安装

GO MOD模式下，执行：

```go
$ go get github.com/gorhill/cronexpr 
```



## 案例

这个库实际上非常简单，我这里拿一个小的例子来展示一下

```go
package main

import (
	"fmt"
	"github.com/gorhill/cronexpr"
	"log"
	"testing"
	"time"
)

func TestCron(t *testing.T) {
	_,nextTimeArr := CheckCrontabExpr("* * 10 * * * *")
	for _,v := range nextTimeArr{
		fmt.Println(v)
	}
}

func CheckCrontabExpr(crontabs string) (err error, nextTimeArr []string) {
	var nextTime []time.Time
	if _, err = cronexpr.Parse(crontabs); err != nil {
		log.Fatal(err)
		return err, nil
	}
	//返回当前crontab后的5次执行,n为次数
	nextTime = cronexpr.MustParse(crontabs).NextN(time.Now(), 5)
	for _, v := range nextTime {
		nextTimeArr = append(nextTimeArr, v.String())
	}
	return nil, nextTimeArr
}
```

上面的例子是打印这个crontab表达式“* * 10 * * * *”的后五次执行时间，结果如下：

````go
=== RUN   TestCron
2021-04-27 10:00:00 +0800 CST
2021-04-27 10:00:01 +0800 CST
2021-04-27 10:00:02 +0800 CST
2021-04-27 10:00:03 +0800 CST
2021-04-27 10:00:04 +0800 CST
--- PASS: TestCron (0.01s)
PASS
````



由于这个crontab表达式的最小粒度是秒，所以我们如果要自己写一个定时任务的话，可以用：

````go
for{
    var now = time.now()
    nextDoCron := cronexpr.MustParse("* * 10 * * * *").Next(now)
	if cronJob.nextTime.Before(now) || cronJob.nextTime.Equal(now) 
    ......
    ......
	select {
		case <-time.NewTimer(1 * time.Second).C:
	}
}

````

这样就只需要每秒检查一次，nextDoCron变量就是根据当前时间time.now()获取到的下一次执行时间

判断过期了或者刚好过期时，则执行if中的内容即可做到定时任务的功效

他的crontab表达式支持非常多的定义，这里我写个自己的总结：

````go
/**
	创建时间：2020-07-08 11:18:24
	根据传入的表达式，检查表达式是否正确：

      字段名       是否强制      支持的格式    		支持的特殊格式
	----------     ----------   --------------    --------------------
	Seconds        No           0-59              * / , -
	Minutes        Yes          0-59              * / , -
	Hours          Yes          0-23              * / , -
	Day of month   Yes          1-31              * / , - L W (L指最后last，若用L，表示月底,W指的是最近的工作日)
	Month          Yes          1-12 or JAN-DEC   * / , -
	Day of week    Yes          0-6 or SUN-SAT    * / , - L # (L指最后last，若用1L，表示本月最后一周的周一)
	Year           No           1970–2099         * / , -
	
	00 */10 08-19 * * * * * 从08点到19点，每10分钟执行一次
	2021-04-27 08:00:00 +0800 CST
	2021-04-27 08:10:00 +0800 CST
	2021-04-27 08:20:00 +0800 CST
	2021-04-27 08:30:00 +0800 CST
	2021-04-27 08:40:00 +0800 CST
	
	00 00 17 * * 1-5 * 我们公司点加班餐的定时任务推送消息 周一到周五，每天的17点推送一次
	2021-04-27 17:00:00 +0800 CST
	2021-04-28 17:00:00 +0800 CST
	2021-04-29 17:00:00 +0800 CST
	2021-04-30 17:00:00 +0800 CST
	2021-05-03 17:00:00 +0800 CST
	
*/

````



## 总结

cronexpr库实际上解决了我们自己去解析表达式的苦，我记得之前我用PHP写过一版crontab表达式解析，那叫一个心惊胆战，生怕自己写错造成生产事故，定时任务有很多种实现方式，这个库只是为我们解决了最基本也是最关键的表达式解析，推荐给大家，可以看看源码哟~



## 还想了解更多吗？

更多请查看：https://github.com/gorhill/cronexpr

欢迎加入我们GOLANG中国社区：https://gocn.vip/

