

# 运行时替换 Golang 函数 — gohook

## Monkey Patching

Monkey Patch (猴子补丁) 是扩展、修改程序的一种比较 hack 的方式。在运行时对类或模块的属性进行动态修改，为现有的第三方代码打上补丁，以解决没有达到预期效果的问题或功能。 在下面几个场景中，我们可以通过 Monkey Patch 来实现：

1. 需要在运行时替换类、方法、属性等。如在单测中 mock 一个方法。
2. 修改、扩展第三方库的行为，而不直接修改其源代码。
3. 修复原来代码存在的安全问题或行为修正。

简单来说就是 Monkey Patch 可以修改当前运行时的变量的状态和行为，多用于动态语言，比如 Python 和 Ruby。

下面是一个Python中动态改变类方法的例子:

```python
class A:
    def origin_func(self):
        print("Hi, origin func")
    def monkey_func(self):
        print("Hi, monkey func")
a = A()
a.origin_func()
A.origin_func=A.monkey_func   # 运行时替换函数
a.origin_func()

''' 运行结果
Hi, origin func
Hi, monkey func
'''
```

## Monkey Patching in Golang

Python、Ruby 等动态语言中能够很容易地实现这个特性，很多人认为 Monkey Patch 只能在动态语言中实现，但静态语言如 Golang 也可以完成运行时的动态替换。Golang 的变量类型和函数体在编译的时候就会确定下来，其在语言级别上不能支持上述功能，但是我们可以通过在机器码层面做些 hack 行为，达到目的。

这里在 Golang 中劫持函数执行的思路可以简单描述为`修改原函数入口的指令，使其跳转到目标新函数继续执行`。详细原理可以参考 https://bou.ke/blog/monkey-patching-in-go

`github.com/bouk/monkey` 是一个基于以上原理实现的一个对 Golang 应用程序 Monkey Patch 的库。

以下为一个简单示例：

```go
func originFunc() {
	str := "Hi, origin func"
	fmt.Println(str)
}

func monkeyFunc() {
	str := "Hi, monkey func"
	fmt.Println(str)
}

func main() {
	originFunc()
	monkey.Patch(originFunc, monkeyFunc)
	originFunc()
}
/* 运行结果
Hi, origin func
Hi, monkey func
*/
```

## gohook

`bouk/monkey` 库已经比较好的实现了运行时动态替换的功能，[brahma-adshonor/gohook](https://github.com/brahma-adshonor/gohook) 是在其启发下实现的拥有更多选项的另外一个运行时动态替换库。  相比`bouk/monkey` ，**gohook** 具有以下优点：

- 跳转效率更高
- 更安全可靠
- 支持回调旧函数(最大优点)
- 不依赖 runtime 内部实现

**gohook** 实现了对函数的暴力拦截，无论是普通函数，还是成员函数都可以强行拦截替换，并支持回调原来的旧函数。

**gohook** 有以下几个方法 :

- `func Hook(target, replace, trampoline interface{}) error;`
- `func UnHook(target interface{}) error;`
- `func HookMethod(instance interface{}, method string, replace, trampoline interface{}) error;`
- `func UnHookMethod(instance interface{}, method string) error;`
- `func HookByIndirectJmp(target, replace, trampoline interface{});`

一般情况下我们使用前四个方法就可以满足日常需求。

对于 `Hook` 方法，其接受三个参数，第一个参数是要 hook 的目标原函数，第二个参数是用来替换的函数，第三个参数用来支持回调旧函数。当 hook 完成后，会调用 trampoline，其相当于调用旧的目标函数(target)，第三个参数可以传入 nil，此时表示不需要支持回调旧函数。

直接拦截替换的例子 ：

```go
func originFunc() {
	str := "Hi, origin func"
	fmt.Println(str)
}

func monkeyFunc() {
	str := "Hi, monkey func"
	fmt.Println(str)
}

func main() {
	originFunc()
	gohook.Hook(originFunc, monkeyFunc, nil)
	originFunc()
}

/* 运行结果
Hi, origin func
Hi, monkey func
*/

```
替换后回调原函数 :

```go
func originFunc() {
	str := "Hi, origin func"
	fmt.Println(str)
}

func monkeyFunc() {
	str := "Hi, monkey func"
	fmt.Println(str)
	trampolineFunc()
}

func trampolineFunc() {
}

func main() {
	originFunc()
	fmt.Println("-------")
	gohook.Hook(originFunc, monkeyFunc, trampolineFunc)
	originFunc()
}

/* 运行结果
Hi, origin func
-------
Hi, monkey func
Hi, origin func
*/
```

这里的 `trampoline` 函数内的内容是什么并不重要，只是为了给原函数申请空间。

除了hook普通过程函数外，还可以使用 `HookMethod`方法 hook 成员函数。

---

一个实用的场景：比如我们想替换 time.Now() 函数，使其在每次调用时，返回一个固定时间:

```go
func myTime() time.Time {
	return time.Date(2022, 1, 1, 0, 0, 0, 0, &time.Location{})
}

func main() {
	fmt.Println(time.Now())
	gohook.Hook(time.Now, myTime, nil)
	fmt.Println(time.Now())
}

/* 运行结果
2022-01-24 00:00:00 +0800 CST m=+0.000498280
2022-01-01 00:00:00 +0000 UTC
*/
```

---

使用 **gohook** 还需要注意以下几个事情:

- **gohook** 项目主要是用来辅助作测试，最好不要用于生产环境。
- 过小的函数有可能会变成内联函数，在编译期间被优化掉，这样在运行时就无法 hook了。这也是上面例子中将`fmt.Println(str) `和 `str := "Hi, origin func"`拆开的原因。（编译时加上`-gcflags='-m'`选项可以查看哪些函数被 inline，另外也可以通过 `// go:noline` 或`-gcflags=all='-l'`来告诉编译器不要对其进行 inline)。

- 跳转指令取决于硬件平台，该实现只支持 x86/x64 架构。

## 总结

**gohook** 实现了运行时动态拦截、修改函数，方便我们在单元测试时 mock 代码，也可以让我们方便的修改第三方库中没有达到预期效果的问题或功能。但是这种 hack 方案在带了便利的同时也充满着风险，破坏正常的代码逻辑，使不了解的同学感到困惑，我们在使用时应当保持小心和谨慎的态度。

## Reference

[python - What is monkey patching? - Stack Overflow](https://stackoverflow.com/questions/5626193/what-is-monkey-patching)

[Monkey Patching in Go (bou.ke)](https://bou.ke/blog/monkey-patching-in-go/)

[About Monkey Patch)](https://www.cnblogs.com/robert871126/p/10107258.html)

[bouk/monkey: Monkey patching in Go (github.com)](https://github.com/bouk/monkey)

[brahma-adshonor/gohook: a nice library to hook golang function at runtime (github.com)](https://github.com/brahma-adshonor/gohook)

[golang 函数拦截原理介绍.pptx](https://onedrive.live.com/View.aspx?resid=7804A3BDAEB13A9F!58083&authkey=!AKVlLS9s9KYh07s)

[gohook 一个支持运行时替换 golang 函数的库实现 - twoon](https://www.cnblogs.com/catch/p/10973611.html)

[Go语言中的内联函数 ](https://segmentfault.com/a/1190000040399875)

