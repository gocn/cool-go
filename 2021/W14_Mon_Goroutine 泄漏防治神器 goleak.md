# Goroutine 泄漏防治神器 goleak

## 推荐 goleak 的背景

goroutine 作为 golang 并发实现的核心组成部分，非常容易上手使用，但却很难驾驭得好。我们经常会遭遇各种形式的 goroutine 泄漏，这些泄漏的 goroutine 会一直存活直到进程终结。它们的占用的栈内存一直无法释放、关联的堆内存也不能被 GC 清理，系统的可用内存会随泄漏 goroutine 的增多越来越少，直至崩溃！

![goroutine leaks](https://gocn.oss-cn-shanghai.aliyuncs.com/photo/bupt_xingxin/afae586b-655e-42a8-849b-20eb6531c9d6.jpg?x-oss-process=image)

goroutine 的泄漏通常伴随着复杂的协程间通信，代码评审和常规的单元测试通常更专注于业务逻辑正确，很难完全覆盖 goroutine 泄漏的场景；而 `pprof` 等性能分析工具更多是作用于监控报警/故障之后的复盘。我们需要一款能在编译部署前识别 goroutine 泄漏的工具，从更上游把控工程质量。

[`goleak`](https://pkg.go.dev/go.uber.org/goleak)([https://github.com/uber-go/goleak](https://github.com/uber-go/goleak) MIT 许可协议) 是 Uber 团队开源的一款 goroutine 泄漏检测工具，它可以非常轻量地集成到**测试中**，对于 goroutine 泄漏的防治和工程鲁棒性的提升很有帮助。

> **防范**胜于救灾

## goroutine 泄漏举例

先举个 goroutine 泄漏的例子；如下所示，`leak` 方法中的 `ch` 永远没有写操作且不会关闭，读取 `ch` 的 goroutine 一直处于阻塞状态，这是一种很典型的 goroutine 泄漏。

```go
func leak() {
	ch := make(chan struct{})
	go func() {
		ch <- struct{}{}
	}()
}
```

通常我们会为 `leak` 方法写类似下面的测试：

```go
func TestLeak(t *testing.T) {
	leak()
}
```

用 `go test` 执行测试看看结果：

```
$ go test -v -run ^TestLeak$
=== RUN   TestLeak
--- PASS: TestLeak (0.00s)
PASS
ok      cool-go.gocn.vip/goleak 0.007s
```

测试不出意外地顺利通过了，go 内置的测试显然无法帮我们识别 `leak` 中的 goroutine 泄漏。

## 集成 goleak 测试

`goleak` 暴露的方法特别精简，通常我们只需关注 `VerifyNone` 和 `VerifyTestMain` 两个方法，它们也对应了 `goleak` 的两种集成方式：

### 逐用例集成

在现有测试的首行添加 `defer goleak.VerifyNone(t)`，即可集成 goleak 泄漏检测：

```go
func TestLeakWithGoleak(t *testing.T) {
	defer goleak.VerifyNone(t)
	leak()
}
```

这次的 `go test` 失败了：

```
$ go test -v -run ^TestLeakWithGoleak$
=== RUN   TestLeakWithGoleak
    leaks.go:78: found unexpected goroutines:
        [Goroutine 19 in state chan send, with cool-go.gocn.vip/goleak.leak.func1 on top of the stack:
        goroutine 19 [chan send]:
        cool-go.gocn.vip/goleak.leak.func1(0xc00008c420)
                /Users/blanet/gocn/goleak/main.go:24 +0x35
        created by cool-go.gocn.vip/goleak.leak
                /Users/blanet/gocn/goleak/main.go:23 +0x4e
        ]
--- FAIL: TestLeakWithGoleak (0.45s)
FAIL
exit status 1
FAIL    cool-go.gocn.vip/goleak 0.459s
```

测试报告显示名为 `leak.func1` 的 goroutine 发生了泄漏（`leak.func1` 在这里指的是 `leak` 方法中的第一个匿名方法），并将测试结果置为失败。我们成功通过 `goleak` 找到了 goroutine 泄漏。

### 通过 TestMain 集成

如果觉得逐用例集成 `goleak` 的方式太过繁琐或“入侵”性太强，不妨试试完全不改变原有测试用例，通过在 `TestMain` 中添加 `goleak.VerifyTestMain(m)` 的方式集成 `goleak`：

```go
func TestMain(m *testing.M) {
	goleak.VerifyTestMain(m)
}
```

这次的 `go test` 输出如下：

```
$ go test -v -run ^TestLeak$
=== RUN   TestLeak
--- PASS: TestLeak (0.00s)
PASS
goleak: Errors on successful test run: found unexpected goroutines:
[Goroutine 19 in state chan send, with cool-go.gocn.vip/goleak.leak.func1 on top of the stack:
goroutine 19 [chan send]:
cool-go.gocn.vip/goleak.leak.func1(0xc00008c2a0)
        /Users/blanet/gocn/goleak/main.go:24 +0x35
created by cool-go.gocn.vip/goleak.leak
        /Users/blanet/gocn/goleak/main.go:23 +0x4e
]
exit status 1
FAIL    cool-go.gocn.vip/goleak 0.455s
```

可见，`goleak` 再次成功检测到了 goroutine 泄漏，但与逐用例集成不同的是，`goleak.VerifyTestMain` 会先报告用例执行的结果，然后再进行泄漏分析。如果单次测试执行了多个用例且最终发生泄漏，那么以 `TestMain` 方式集成的 `goleak` 并不能精准定位发生 goroutine 泄漏的用例，还需进一步分析。

> `goleak` 提供了[如下脚本](https://github.com/uber-go/goleak#determine-source-of-package-leaks)用于进一步推断具体发生 goroutine 泄漏的用例，其本质是逐一执行所有用例进行分析：

```sh
# Create a test binary which will be used to run each test individually
$ go test -c -o tests

# Run each test individually, printing "." for successful tests, or the test name
# for failing tests.
$ for test in $(go test -list . | grep -E "^(Test|Example)"); do
	./tests -test.run "^$test\$" &>/dev/null && echo -n "." || echo "\n$test failed"
done
```

## 总结

`goleak` 通过对运行时的栈分析获取 goroutine 状态，并设计了非常简洁易用的接口与测试框架进行对接，是一款小巧强悍的 goroutine 泄漏防治利器。

最后，完备的测试用例支持是 `goleak` 发挥作用的基础，大家还是要老老实实写测试，稳稳当当搞生产！

## 参考资料

* https://github.com/uber-go/goleak
* https://pkg.go.dev/go.uber.org/goleak
* https://rakyll.org/leakingctx/
* https://github.com/golang/go/issues/6705
* https://medium.com/golangspec/goroutine-leak-400063aef468
* https://dave.cheney.net/2016/12/22/never-start-a-goroutine-without-knowing-how-it-will-stop

---

欢迎加入 GOLANG 中国社区：https://gocn.vip