# 跨平台文件系统监控工具 — fsnotify

## 简介

在 linux 内核中，`Inotify` 是一种用于通知用户空间程序文件系统变化的机制。它监控文件系统的变化，如文件新建、修改、删除等，并可以将相应的事件通知给应用程序。`Inotify` 既可以监控文件，也可以监控目录。当监控目录时，它可以同时监控目录及目录中的各子目录及文件。Golang 的标准库 `syscall` 实现了该机制。为了进一步扩展和抽象， `github.com/fsnotify/fsnotify` 包实现了一个基于 channel 的、跨平台的实时监听接口。

## 如何使用：

`fsnotify`的使用非常简单：

- `NewWatcher`初始化一个 watcher
- 使用 watcher 的`Add`方法添加需要监听的文件或目录到监听队列中
- 创建新的 goroutine，等待管道中的事件或错误

```go
package main

import (
	"fmt"
	"log"

	"github.com/fsnotify/fsnotify"
)

func main() {
	// 1、NewWatcher 初始化一个 watcher
	watcher, err := fsnotify.NewWatcher()
	if err != nil {
		log.Fatal(err)
	}
	defer watcher.Close()

	//3、创建新的 goroutine，等待管道中的事件或错误
	done := make(chan bool)
	go func() {
		for {
			select {
			case e, ok := <-watcher.Events:
				if !ok {
					return
				}
				fmt.Printf("监听到文件 %s 变化| ", e.Name)
				switch e.Op {
				case fsnotify.Create:
					fmt.Println("创建事件", e.Op)
				case fsnotify.Write:
					fmt.Println("写入事件", e.Op)
				case fsnotify.Remove:
					fmt.Println("删除事件", e.Op)
				case fsnotify.Rename:
					fmt.Println("重命名事件", e.Op)
				case fsnotify.Chmod:
					fmt.Println("属性修改事件", e.Op)
				default:
					fmt.Println("some thing else")
				}
			case err, ok := <-watcher.Errors:
				if !ok {
					return
				}
				log.Println("error:", err)
			}
		}
	}()
	// 2、使用 watcher 的 Add 方法增加需要监听的文件或目录到监听队列中
	err = watcher.Add("./")
	if err != nil {
		log.Fatal(err)
	}
	<-done
}
```

运行以上代码，然后执行以下操作：

- 在当前目录创建一个`test.txt`
- 修改权限为`666`
- 重命名为`test_2.txt`文件
- 追加内容`test content`
- 删除该文件

观察程序输出，如图：

![](https://gocn.oss-cn-shanghai.aliyuncs.com/photo/bazinga/a4c847e3-c7ea-42bd-bf76-db7d26d70b1f.png?x-oss-process=image%2Fresize%2Cw_500)

![](https://gocn.oss-cn-shanghai.aliyuncs.com/photo/bazinga/7a7245f3-a04a-416c-8232-24d9b53c9263.png?x-oss-process=image%2Fresize%2Cw_500)

需要注意的是：

- 这里使用`touch`命令新建文件，实际会修改文件的时间属性，所以会有一个`CREATE`事件，一个`CHMOD`事件。

- 重命名时会产生两个事件，一个是原文件的`RENAME`事件，一个是新文件的`CREATE`事件。

## 事件

这里的事件的结构体`fsnotify.Event`如下：

```go
// Event represents a single file system notification.
type Event struct {
	Name string // Relative path to the file or directory.
	Op   Op     // File operation that triggered the event.
}
```

该结构只有两个属性，`Name`表示发生变化的文件或目录名，`Op`表示具体的变化。`Op`有 5 种枚举：

```go
// Op describes a set of file operations.
type Op uint32

// These are the generalized file operations that can trigger a notification.
const (
	Create Op = 1 << iota
	Write
	Remove
	Rename
	Chmod
)
```

## 总结

`fsnotify`的接口非常简洁，将系统相关的复杂性都封装在了内部。方便开发者监听文件变化然后执行后续自定义操作。`fsnotify` 不足的是目前它无法递归的监听子目录变更事件，需要我们自已去实现。

## Reference

[fsnotify/fsnotify: Cross-platform file system notifications for Go. (github.com)](https://github.com/fsnotify/fsnotify)

[golang 通过fsnotify监控文件，并通过文件变化重启程序 - 怀素真 - 博客园 (cnblogs.com)](https://www.cnblogs.com/jkko123/p/7256927.html)

[大俊的博客 (darjun.github.io)](https://darjun.github.io/2020/01/19/godailylib/fsnotify/)



