如何将 Markdown 文档解析成 html？如何对 Markdown 语法进行个性化扩展以实现特定功能？goldmark 也许是一个不错的选择！

## 简介

使用 Markdown 书写结构化的文档和评论已经相当流行了，Web 服务需要将用户编写的 Markdown 文本转换为 html 以便浏览器渲染，还常常需要对 Markdown 语法进行自定义扩展以实现个性化的功能。

本期要介绍的 goldmark 就是 Go 生态中的一款 Markdown 解析器和扩展器，与 GitHub 中被广泛应用的 GFM(GitHub Flavored Markdown) 一样，goldmark 也遵循 CommonMark 标准，且非常易于使用和扩展。

goldmark 有以下特点:

* 完全符合最新版(0.30)的 CommonMark 规范
* 易扩展，例如使用 goldmark 添加 @username 扩展会非常容易
* 具备与 cmark 相当的性能
* 鲁棒性好，goldmark 使用 go-fuzz 进行模糊测试
* 丰富的内置扩展，如表格、删除线、任务列表和定义列表等
* 工程仅依赖 Go 标准库

## 使用举例

使用 goldmark 将基本的 Markdown 转换成 html 非常简单:

```go
package main

import (
    "os"

    "github.com/yuin/goldmark"
)

func main() {
    const demo = `# h1 Heading
## h2 Heading

**This is bold text**

*This is italic text*

[Some link](https://www.example.com)

* unordered list 01
* unordered list 02

1. ordered list 01
2. ordered list 02
`
    if err := goldmark.Convert([]byte(demo), os.Stdout); err != nil {
        panic(err)
    }
}
```

转换得到的 html 如下，符合我们的预期:

```html
<h1>h1 Heading</h1>
<h2>h2 Heading</h2>
<p><strong>This is bold text</strong></p>
<p><em>This is italic text</em></p>
<p><a href="https://www.example.com">Some link</a></p>
<ul>
<li>unordered list 01</li>
<li>unordered list 02</li>
</ul>
<ol>
<li>ordered list 01</li>
<li>ordered list 02</li>
</ol>
```

我们再来试试带扩展文档的转换(本例中的文档使用了 GFM 与脚注)，注意这次需要额外引入一些 goldmark 的内置扩展:

```go
package main

import (
    "os"

    "github.com/yuin/goldmark"
    "github.com/yuin/goldmark/extension"
)

func main() {
    const demo = `# Demo with extensions

Footnote 1 link[^first].

Footnote 2 link[^second].

[^first]: Footnote with *markup*

[^second]: Footnote text.

~~Strikethrough~~

http://this.should.be.a.link.com
`
    md := goldmark.New(
        // 添加扩展
        goldmark.WithExtensions(extension.GFM, extension.Footnote),
    )

    if err := md.Convert([]byte(demo), os.Stdout); err != nil {
        panic(err)
    }
}
```

本次转换得到的 html 如下:

```html
<h1>Demo with extensions</h1>
<p>Footnote 1 link<sup id="fnref:1"><a href="#fn:1" class="footnote-ref" role="doc-noteref">1</a></sup>.</p>
<p>Footnote 2 link<sup id="fnref:2"><a href="#fn:2" class="footnote-ref" role="doc-noteref">2</a></sup>.</p>
<p><del>Strikethrough</del></p>
<p><a href="http://this.should.be.a.link.com">http://this.should.be.a.link.com</a></p>
<div class="footnotes" role="doc-endnotes">
<hr>
<ol>
<li id="fn:1">
<p>Footnote with <em>markup</em>&#160;<a href="#fnref:1" class="footnote-backref" role="doc-backlink">&#x21a9;&#xfe0e;</a></p>
</li>
<li id="fn:2">
<p>Footnote text.&#160;<a href="#fnref:2" class="footnote-backref" role="doc-backlink">&#x21a9;&#xfe0e;</a></p>
</li>
</ol>
</div>
```

html 不够直观，用浏览器渲染出来的样式大致如下图，可见脚注、删除线和超链接被正确解析啦！

![带扩展的 Markdown 渲染](https://cdn.gocn.vip/forum-user-images/20220504/5ac9cf24a2e646beb6e7fbba79ff183f.jpg)

## 总结

goldmark 不仅能够帮助我们轻松将 Markdown 文档转换成对应的 html，还内置了非常多的常用扩展支持，使用 goldmark 开发自定义扩展的成本也相对较低，快使用 goldmark 玩转服务端 Markdown 渲染吧！

## 参考资料

* [https://github.com/yuin/goldmark](https://github.com/yuin/goldmark)
* [https://commonmark.org/](https://commonmark.org/)
* [https://github.github.com/gfm/](https://github.github.com/gfm/)
* [https://michelf.ca/projects/php-markdown/extra/](https://michelf.ca/projects/php-markdown/extra/)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
