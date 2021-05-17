# 强大的模糊测试工具 go-fuzz

## 推荐 go-fuzz 的背景

我们在日常开发中经常会编写测试和对应的测试用例，大家是否常常会有以下疑惑：

* 现有的测试用例是否完全覆盖了各种边界场景？会不会有意料之外的 case？
* 代码测试覆盖率都达到 100% 了，代码上线时为啥还会战战兢兢？
* 写测试用例太费心费力了，有没有一款能自动生成测试用例的工具？

这次要推荐给大家的 **`go-fuzz`** 也许能让你的工程鲁棒性再上一个台阶，并或多或少缓解你的担忧。

[**`go-fuzz`**](https://github.com/dvyukov/go-fuzz)（[https://github.com/dvyukov/go-fuzz](https://github.com/dvyukov/go-fuzz)）是 [Dmitry Vyukov](https://www.1024cores.net/home/about-me) 大神早在 go1.5 时代开源（Apache License 2.0 开源许可）的一款 golang 模糊测试工具，为解析复杂输入（文本或二进制）的系统提供了强大的鲁棒性验证手段。迄今为止，`go-fuzz` 已经为 go 语言（你没看错，就是 golang 自身）和一些三方库检测出了[**几百个缺陷**](https://github.com/dvyukov/go-fuzz#trophies)，可谓居功至伟！

## go-fuzz 与模糊测试

维基百科对模糊测试的解释如下：

> 模糊测试 （fuzz testing, fuzzing）是一种软件测试技术。其核心思想是将自动或半自动生成的随机数据输入到一个程序中，并监视程序异常，如崩溃，断言（assertion）失败，以发现可能的程序错误，比如内存泄漏。模糊测试常常用于检测软件或计算机系统的安全漏洞。

`go-fuzz` 自动生成的测试用例可不是简单的随机，它强依赖于 `afl`（[American Fuzzy Lop](https://lcamtuf.coredump.cx/afl/)），在执行时会根据程序现有的用例和用例执行情况按照一定的算法规则进行迭代修改，从而无限“裂变”生成新测试。

目前 [gitlab.com](https://docs.gitlab.com/ee/user/application_security/coverage_fuzzing/) 和 [fuzzbuzz.io](https://docs.fuzzbuzz.io/getting-started/find-your-first-bug-in-go) 上均有基于 `go-fuzz` 的 ci 集成。

## go-fuzz 应用举例

下面我们就以 fuzzbuzz.io 上的小例子来看 go-fuzz 如何使用。如下代码中有一处比较隐晦的 bug，当输入的 `Data` 刚好是 `FUZ` 时会触发访问越界：

```golang
package tutorial

// BrokenMethod has a bug - it will try to read the 4th
// index of Data even when it only has a length of 3.
func BrokenMethod(Data string) bool {
    return len(Data) >= 3 &&
        Data[0] == 'F' &&
        Data[1] == 'U' &&
        Data[2] == 'Z' &&
        Data[3] == 'Z'
}
```

接下来我们尝试使用 `go-fuzz` 对漏洞进行探测

### *Step0:* 安装 `go-fuzz-build` 和 `go-fuzz`

```bash
go get -u github.com/dvyukov/go-fuzz/go-fuzz@latest github.com/dvyukov/go-fuzz/go-fuzz-build@latest
```

> 别忘了把 $GOPATH/bin 添加到 PATH 中

### *Step1:* 编写测试函数

在代码中添加 method_fuzz.go，注意 `// +build gofuzz` 是必须添加的，接下来的构建步骤会对其进行识别。

```golang
// +build gofuzz
package tutorial

func Fuzz(data []byte) int {
  BrokenMethod(string(data))
  return 0
}
```

> `Fuzz` 函数的返回码目前有 3 个可选值：返回 `1` 表示当前的输入权重增加，返回 `-1` 表示当前的输入不添加进语料库，否则返回 `0`。

### *Step2:* 设计几个初始语料

我们添加 `F` 和 `FU` 作为 `BrokenMethod` 的两个测试用例。当然，如果你的代码中有一些已经设计好的用例，也可以直接复制到 `workdir/corpus` 下。

```bash
mkdir -p workdir/corpus
echo -n "F"  >workdir/corpus/1
echo -n "FU" >workdir/corpus/2
```

> 添加初始语料不是必须的，但是 `go-fuzz` 作者建议初始语料越丰富越好，这对后续的模糊测试执行很有帮助！

### *Step3:* `go-fuzz-build` 生成测试工程

```bash
go get -d github.com/dvyukov/go-fuzz-corpus
go-fuzz-build
```

这一步可能需要花一些时间，这跟工程的复杂度有关系。执行成功后，会在当前目录里看到一个 `tutorial-fuzz.zip` 的压缩包。

> `go-fuzz` 是 go1.5 时期的老家伙了，当前对 go module 的支持还处于早期阶段。构建测试前执行 **`go get -d github.com/dvyukov/go-fuzz-corpus`** 会在 go.mod 里添加一行并不需要的依赖，模糊测试执行完毕后使用 go mod tidy 即可恢复。

### *Step4:* `go-fuzz` 执行模糊测试

```bash
go-fuzz -bin=tutorial-fuzz.zip -workdir=workdir
```

这时我们看到控制台有如下输出：

```plain
2021/05/16 13:56:45 workers: 4, corpus: 4 (2s ago), crashers: 1, restarts: 1/0, execs: 0 (0/sec), cover: 0, uptime: 3s
2021/05/16 13:56:48 workers: 4, corpus: 4 (5s ago), crashers: 1, restarts: 1/0, execs: 0 (0/sec), cover: 6, uptime: 6s
2021/05/16 13:56:51 workers: 4, corpus: 4 (8s ago), crashers: 1, restarts: 1/408, execs: 48969 (5440/sec), cover: 6, uptime: 9s
...
```

> `go-fuzz` 执行测试时不会自动终止，当我们发现 **crashers** 字段的值不为 0 时（有用例触发了程序异常），就可以终止测试并查看测试结果了，导致程序异常的用例会存放在 workdir/crashers/ 目录中

### *Step5:* 分析测试结果

```bash
$ tree workdir/crashers/
workdir/crashers
├── 0eb8e4ed029b774d80f2b66408203801cb982a60
├── 0eb8e4ed029b774d80f2b66408203801cb982a60.output
└── 0eb8e4ed029b774d80f2b66408203801cb982a60.quoted
```

可见，`workdir/crashers` 中多出了 3 个文件，它们的文件名均为输入用例的 sha1sum 值。

> * 不带后缀的文件存放用例的原始输入
> * 后缀 `.quoted` 的文件存放字符串形式的用例输入（方便贴入代码直接进行调试，设计太友好了）
> * 后缀为 `.output` 的文件存放异常时的错误输出

```bash
$ cat workdir/crashers/0eb8e4ed029b774d80f2b66408203801cb982a60.quoted
  "FUZ"
$ cat workdir/crashers/0eb8e4ed029b774d80f2b66408203801cb982a60.output
panic: runtime error: index out of range [3] with length 3

goroutine 1 [running]:
demo.BrokenMethod.func4(...)
  /Users/blanet/repos/tmp/tutorial-go/method.go:9
demo.BrokenMethod(0xc000092e80, 0x3, 0x3)
  /Users/blanet/repos/tmp/tutorial-go/method.go:10 +0x11d
demo.Fuzz(0x4810000, 0x3, 0x3, 0x3)
  /Users/blanet/repos/tmp/tutorial-go/fuzz.go:5 +0x6f
go-fuzz-dep.Main(0xc000092f70, 0x1, 0x1)
  go-fuzz-dep/main.go:36 +0x1b8
main.main()
  demo/go.fuzz.main/main.go:15 +0x52
exit status 2
```

至此，我们找到了程序中的漏洞以及复现漏洞的用例，稍加调试问题就迎刃而解了！漏洞修复后，我们还可以为找到的 bad case 设计新的单元测试，进一步提升代码质量。

## 总结

使用 `go-fuzz` 可以为程序集成模糊测试，对于检测复杂输入系统的鲁棒性、筛查各种深水区 `panic` 的场景非常有帮助。大家赶快试用吧！

## 参考资料

* [https://github.com/dvyukov/go-fuzz](https://github.com/dvyukov/go-fuzz)
* [https://go-talks.appspot.com/github.com/dvyukov/go-fuzz/slides/go-fuzz.slide](https://go-talks.appspot.com/github.com/dvyukov/go-fuzz/slides/go-fuzz.slide#1)
* [https://www.youtube.com/watch?v=a9xrxRsIbSU&t=459s](https://www.youtube.com/watch?v=a9xrxRsIbSU&t=459s)
* [https://github.com/google/gofuzz](https://github.com/google/gofuzz)
* [https://en.wikipedia.org/wiki/Fuzzing](https://en.wikipedia.org/wiki/Fuzzing)
* [https://docs.gitlab.com/ee/user/application_security/coverage_fuzzing/](https://docs.gitlab.com/ee/user/application_security/coverage_fuzzing/)
* [https://about.gitlab.com/blog/2020/12/03/how-to-fuzz-go/](https://about.gitlab.com/blog/2020/12/03/how-to-fuzz-go/)
* [https://docs.fuzzbuzz.io/getting-started/find-your-first-bug-in-go](https://docs.fuzzbuzz.io/getting-started/find-your-first-bug-in-go)
* [https://medium.com/@dgryski/go-fuzz-github-com-arolek-ase-3c74d5a3150c](https://medium.com/@dgryski/go-fuzz-github-com-arolek-ase-3c74d5a3150c)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
