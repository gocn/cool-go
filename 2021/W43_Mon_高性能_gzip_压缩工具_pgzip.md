# 高性能 gzip 压缩工具 pgzip

## 背景

`gzip` 是当前应用非常广泛的文件压缩格式，golang 中也有内置的 `compress/gzip` 对其提供原生支持。今天我们要介绍的 `pgzip` 是一款完全兼容 gzip 的插件，并能提供相较于 `compress/gzip` 上百倍的性能提升！

## pgzip 简介

`pgzip`(https://github.com/klauspost/pgzip) 主要是通过分块并行压缩实现性能提升的，尤其适用于大量数据（>1MB）压缩的场景。由于 `pgzip` 的 API 设计完全兼容 `compress/gzip`，因此我们的工程升级到 `pgzip` 只需要特别小的改动：

```golang
import gzip "github.com/klauspost/pgzip"
```

另外，`pgzip` 还支持指定并发度，我们可以据此对压缩消耗的 CPU 资源进行管控；`pgzip` 还附带了一个线性时间压缩模式（Huffman only compression），该模式支持以每核每秒约 250MB 的速度进行数据压缩。

## 应用举例

随机生成一个 100MB 的大文件作为待压缩文件：

```bash
dd if=/dev/random of=data.bin bs=4m count=25
```

首先使用原生的 `compress/gzip` 进行压缩：

```golang
package main

import (
	"compress/gzip"
	"io"
	"log"
	"os"
)

func main() {
	fr, err := os.Open("data.bin")
	if err != nil {
		log.Fatalf("failed to open file to read: %v", err)
	}
	defer fr.Close()

	fw, err := os.Create("gzip_res.gz")
	if err != nil {
		log.Fatalf("failed to open file to write: %v", err)
	}
	defer fw.Close()

	w := gzip.NewWriter(fw)
	defer w.Close()
	_, err = io.Copy(w, fr)
	if err != nil {
		log.Fatalf("failed to gzip: %v", err)
	}
}
```

查看执行时间：

```bash
$ go build -o gzip_demo
$ time ./gzip_demo
./gzip_demo  1.86s user 0.23s system 99% cpu 2.111 total
```

现在我们切换到 `pgzip` 对同样的文件进行压缩：

```golang
package main

import (
	"io"
	"log"
	"os"

	gzip "github.com/klauspost/pgzip"
)

func main() {
	fr, err := os.Open("data.bin")
	if err != nil {
		log.Fatalf("failed to open file to read: %v", err)
	}
	defer fr.Close()

	fw, err := os.Create("pgzip_res.gz")
	if err != nil {
		log.Fatalf("failed to open file to write: %v", err)
	}
	defer fw.Close()

	w := gzip.NewWriter(fw)
	defer w.Close()
	// 1MB block with 4 concurrency
	w.SetConcurrency(1<<20, 4)
	_, err = io.Copy(w, fr)
	if err != nil {
		log.Fatalf("faield to gzip: %v", err)
	}
}
```

查看执行时间：

```bash
$ go build -o pgzip_demo
$ time ./pgzip_demo
./pgzip_demo  0.07s user 0.08s system 240% cpu 0.064 total
```

可见，使用 `pgzip` 压缩同样 100MB 大小的文件，使用 4 核 CPU 时压缩耗时有几十倍（1.86s -> 0.07s）的缩短！

## 总结

`pgzip` 实现了分块并行的 gzip 压缩能力，极大提升了压缩速率，能够充分利用多核 CPU 的计算资源。当然，`pgzip` 提升压缩速率的同时也会消耗更多的计算资源，能够达到的压缩比也会比原生 gzip 略低。

`pgzip` 非常适用于大文件压缩的场景，例如提升构建产物的打包速率、优化数据备份时效等，快来试试吧！

## 参考资料

* [https://github.com/klauspost/pgzip](https://github.com/klauspost/pgzip)
* [https://github.com/klauspost/compress](https://github.com/klauspost/compress)
* [http://blog.klauspost.com/constant-time-gzipzip-compression/](http://blog.klauspost.com/constant-time-gzipzip-compression/)
* [https://github.com/pierrec/lz4](https://github.com/pierrec/lz4)
* [https://zlib.net/pigz/](https://zlib.net/pigz/)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
