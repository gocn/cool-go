# W46_Mon_对标 linux tail 命令的 Golang tail 包

## tail 简单介绍

tail 命令用途是依照要求将指定的文件的最后部分输出到标准设备，通常是终端，通俗讲来，就是把某个档案文件的最后几行显示到终端上。

假设该档案有更新，tail 会自己主动刷新，确保你看到最新的档案内容 ，在日志收集中可以实时的监测日志的变化。

## 安装 tail

```shell
go get github.com/hpcloud/tail/...
```

## 快速使用 tail

使用场景：监听一个文件 info.log, 当 info.log 出现 `gopherchina` 字样时表明某项服务已经启动完毕

* 首先初始化配置结构体 config
* 调用 TailFile 函数，并传入文件路径和config，返回有个 tail 的结构体，tail 结构体的 Lines 字段封装了拿到的信息

* 遍历 tail.Lnes 字段，取出信息（注意这里要循环取数据，因为 tail 可以实现实时监控）

 ```go
 package main
 
 import (
 	"fmt"
 	"strings"
 	"time"
 
 	"github.com/hpcloud/tail"
 )
 
 func main() {
 	fileName := "./info.log"
 	config := tail.Config{
 		ReOpen:    true,                                 // 重新打开
 		Follow:    true,                                 // 是否跟随
 		Location:  &tail.SeekInfo{Offset: 0, Whence: 2}, // 从文件的哪个地方开始读
 		MustExist: false,                                // 文件不存在不报错
 		Poll:      true,
 	}
 	tails, err := tail.TailFile(fileName, config)
 	if err != nil {
 		fmt.Println("tail file failed, err:", err)
 		return
 	}
 	var (
 		line *tail.Line
 		ok   bool
 	)
 	for {
 		line, ok = <-tails.Lines//遍历chan，读取日志内容
 		if !ok {
 			fmt.Printf("tail file close reopen, filename:%s\n", tails.Filename)
 			time.Sleep(time.Second)
 			continue
 		}
 		if strings.Contains(line.Text, "gopherchina") {
 			fmt.Println("service has been started")
 			return
 		}
 	}
 }
 ```

## 参考文档

* https://github.com/hpcloud/tail
* https://www.cnblogs.com/wind-zhou/p/12840174.html