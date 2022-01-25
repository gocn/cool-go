# 文本差异对比工具 go-diff

## 简介

纯文本差异对比在许多场景下都有应用，如语音识别技术对识别率的评估，需要将识别后的文本与预期文本之间做差异对比计算；又如我们使用 Git 进行代码提交时，通常会使用`git diff`来查看这次编辑发生了哪些改动。

这里我们先简单定义一下差异 diff：是指目标文本和源文本之间的区别，也就是将源文本变成目标文本所需要的操作。

以上问题的一个通常解决方案是 Eugene W.Myers 在1986年发表的一篇论文 [ An O(ND) Difference Algorithm and Its Variations ](http://xmailserver.org/diff2.pdf)中提出的 **Myers差分算法**，该算法是一个能在大部分情况产生「最短的直观的diff」的算法。

[google/diff-match-patch](https://github.com/google/diff-match-patch)  项目是Myers差分算法的一种实现。但是该项目缺少 Golang 语言的实现。

[go-diff](https://github.com/sergi/go-diff) 则是 [google/diff-match-patch](https://github.com/google/diff-match-patch)  的一个 Golang 版本的补充。

go-diff 主要提供三个功能：

- 比较两段文本并返回它们的差异
- 执行文本的模糊匹配
- 生成和应用补丁

go-diff 不仅能够简洁地输出字符串对比结果，还能够输出规范化的数据结构方便我们的二次开发。

## 如何使用

go-diff 使用方式非常简单，代码如下

```go
const (
	text1 = "gocn vip"
	text2 = "goCN cool"
)

func main() {
	dmp := diffmatchpatch.New()

	diffs := dmp.DiffMain(text1, text2, false)
	fmt.Println(diffs)
}
```

执行以上代码，输出

```
[{Equal go} {Insert CN } {Equal c} {Delete n vip} {Insert ool}]
```

以上输出结果表示从 text1 变成 text2 需要执行的步骤:

- `go` 不需要变动
- 插入 `CN`
- `c` 不需要变动
- 删除 `n vip`
- 插入语 `ool`

`DiffMain`  方法会查找两段文本的不同，并以数组形式返回 diff 差异。

> 这里的 diff 差异就是从左边 text1 的字符串变成右边 text2 的字符串所需要的最少的步骤，每个步骤只能做“保持不变”、“插入”或者“删除”操作。如果我们需要的是替换操作，那么只能是先“删除”后“插入”

工具提供了`DiffPrettyText` 和`DiffPrettyHtml` 等方法，可以将 diff 数组转换成更友好的有颜色高亮的文本或HTML格式报告。

`DiffTimeout`参数用以设置 diff 计算的超时时间，如果超过此时间，则该计算将被截断，并返回目前为止的最佳解决方案。 此时尽管结果是正确的，但可能并不是最佳方案。 该值为 0 时可进行无限时的计算。

## 总结

go-diff 库实现了高效、完备的文本差异对比算法，在类似需求时，如计算编辑距离、模糊匹配时，不需要我们再去手写复杂的算法，非常省心和方便。

## Reference

[git生成diff原理：Myers差分算法 | 大艺术家_SN (chenshinan.github.io)](https://chenshinan.github.io/2019/05/02/git生成diff原理：Myers差分算法/)

[Git 是怎样生成 diff 的：Myers 算法 - CJ Ting's Blog](https://cjting.me/2017/05/13/how-git-generate-diff/)

