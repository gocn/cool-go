# 有限状态机库 fsm

## 1.前言

开发中，我们经常会遇到这种情况，服务模块有多种状态，它们有一定的顺序，先后执行，逐步切换。这时，fsm这个库可以帮助我们更好的管理多个状态。

fsm库，它主要基于两个FSM实现，增加了golang版本的实现：

- Javascript Finite State Machine, https://github.com/jakesgordon/javascript-state-machine
- Fysom for Python, https://github.com/oxplot/fysom (forked at https://github.com/mriehl/fysom)


## 2.简单举例

```go
package main

import (
	"fmt"
	"github.com/looplab/fsm"
)

func enterState(e *fsm.Event) {
	fmt.Printf("event: %s, from:%s to %s\n", e.Event, e.Src, e.Dst)
}

func main() {
	f := fsm.NewFSM(
		"sleeping",
		fsm.Events{
			{Name: "eat", Src: []string{"sleeping"}, Dst: "eating"},
			{Name: "work", Src: []string{"eating"}, Dst: "working"},
			{Name: "sleep", Src: []string{"working"}, Dst: "sleeping"},
		},
		fsm.Callbacks{
			"enter_state": func(e *fsm.Event) { enterState(e) },
		},
	)

	err := f.Event("eat")
	if err != nil {
		fmt.Println(err)
	}

	err = f.Event("work")
	if err != nil {
		fmt.Println(err)
	}

	err = f.Event("sleep")
	if err != nil {
		fmt.Println(err)
	}

}
```

执行，控制台输出如下：
```
$ go run test.go 

event: eat, from:sleeping to eating
event: work, from:eating to working
event: sleep, from:working to sleeping
```

## 3.结构体举例

```go
package main

import (
	"fmt"
	"github.com/looplab/fsm"
)

type Door struct {
	To  string
	FSM *fsm.FSM
}

func NewDoor(to string) *Door {
	d := &Door{
		To: to,
	}

	d.FSM = fsm.NewFSM(
		"closed",
		fsm.Events{
			{Name: "open", Src: []string{"closed"}, Dst: "open"},
			{Name: "close", Src: []string{"open"}, Dst: "closed"},
		},
		fsm.Callbacks{
			"enter_state": func(e *fsm.Event) { d.enterState(e) },
		},
	)

	return d
}

func (d *Door) enterState(e *fsm.Event) {
	fmt.Printf("The door to %s, event: %s, from:%s to %s\n", d.To, e.Event, e.Src, e.Dst)
}

func main() {
	door := NewDoor("zhang san")

	err := door.FSM.Event("open")
	if err != nil {
		fmt.Println(err)
	}

	err = door.FSM.Event("close")
	if err != nil {
		fmt.Println(err)
	}
}
```

执行，控制台输出如下：
```
$ go run test.go 

The door to zhang san, event: open, from:closed to open
The door to zhang san, event: close, from:open to closed
```

## 4.总结

`fsm` 是一个非常简单，好用的状态机管理库。如果你各种状态切换的需求，不妨试试看，相信一定会喜欢上的！

## 参考资料

* [https://github.com/looplab/fsm](https://github.com/looplab/fsm)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
