# 数据备份 reed-solomn库 的使用

## 1. reed-solomn 是什么？

[reed-solomn](github.com/klauspost/reedsolomon) 假如磁盘损坏了一部分，或者光盘一部分出现了污渍，那是不是我们的信息就丢了呢？

当然不是，有一种纠错算法，可以在数据明显缺失的前提下，依然可以恢复数据，它就是 Reed–Solomon 算法。当然，它事先需要冗余备份，在丢失数据在一定范围内，就可以恢复源文件。
    
## 2. Reed Solomon编码原理
把输入数据视为向量D=(D1，D2，..., Dn）, 编码后数据视为向量（D1, D2,..., Dn, C1, C2,.., Cm)，RS编码可视为如下图所示矩阵运算。

![alt 条形图](https://swarm-gateways.net/bzz:/88bb2d9c0fa71e028b9473eabbbd35b14907fd82a0a48eeecd9f963faf605e10/rs-1.png)

需要注意：编码矩阵B必须具有任意子矩阵可逆的特性。

## 3. Reed Solomon解码原理
RS最多能容忍m个数据块被删除，m包括实际数据和冗余数据。 数据恢复的过程如下：

（1）假设D1、D4、C2丢失，从编码矩阵中删掉丢失的数据块/编码块对应的行。

![alt 条形图](https://swarm-gateways.net/bzz:/702e43ea7227b3d872327693f43dd5ebd5f5291392d8ab989b88a01376fbd7b7/rs-2.png)

（2）由于B' 是可逆的，记B'的逆矩阵为 (B'^-1)，则B' * (B'^-1) = I 单位矩阵。两边左乘B' 逆矩阵。

![alt 条形图](https://swarm-gateways.net/bzz:/fe36405ededdf932b1b2efa0bc2636272251e3d3ecd4f6a565bd858007826961/rs-3.png)

（3）得到如下原始数据D的计算公式

![alt 条形图](https://swarm-gateways.net/bzz:/72f0aae20411e808d05b080909141b489249d776b240800f118fb401c7d5dd33/rs-4.png)


## 4. 代码实现及测试

github地址： ` "github.com/klauspost/reedsolomon"` 下载即可使用。

1.reed-solomon 编码测试
此时，输入in.txt，go run main.go

输出 out目录下，30个shards文件。
   
```go
package main
 
import (
    "fmt"
    "io"
    "os"
    "path/filepath"
 
    "github.com/klauspost/reedsolomon"
)
 
 
func main() {
    var DataShards = 10
    var ParShards = 20
    var OutDir = "./out"
 
    var dataShards = &DataShards
    var parShards = &ParShards
    var outDir = &OutDir
 
    fname := "in.txt"
 
    // 1.Create encoding matrix.
    enc, err := reedsolomon.NewStream(*dataShards, *parShards)
    checkErr2(err)
 
 
    fmt.Println("Opening", fname)
    f, err := os.Open(fname)
    checkErr2(err)
 
    instat, err := f.Stat()
    checkErr2(err)
 
    shards := *dataShards + *parShards
    out := make([]*os.File, shards)
 
    // 2.创建输入文件 30个shards
    dir, file := filepath.Split(fname)
    if *outDir != "" {
        dir = *outDir
    }
    for i := range out {
        outfn := fmt.Sprintf("%s.%d", file, i)
        fmt.Println("Creating", outfn)
        out[i], err = os.Create(filepath.Join(dir, outfn))
        checkErr2(err)
    }
 
    // Split into files.
    data := make([]io.Writer, *dataShards)
    for i := range data {
        data[i] = out[i]
    }
    // 3.原始文件拆分
    err = enc.Split(f, data, instat.Size())
    checkErr2(err)
 
    // Close and re-open the files.
    input := make([]io.Reader, *dataShards)
 
    for i := range data {
        out[i].Close()
        f, err := os.Open(out[i].Name())
        fmt.Println("Error ", err)
        input[i] = f
        defer f.Close()
    }
 
    // 4.封装 parity
    parity := make([]io.Writer, *parShards)
    for i := range parity {
        parity[i] = out[*dataShards+i]
        defer out[*dataShards+i].Close()
    }
 
    // 5.Encode 编码rs格式
    err = enc.Encode(input, parity)
    checkErr2(err)
 
    fmt.Printf("File split into %d data + %d parity shards.\n", *dataShards, *parShards)
 
}
 
func checkErr2(err error)  {
    if err != nil {
        os.Exit(1)
    }
}
```

```
直接运行，最后输出如下：
$ go run main.go
Opening in.txt
Creating in.txt.0
Creating in.txt.1
Creating in.txt.2
Creating in.txt.3
Creating in.txt.4
Creating in.txt.5
Creating in.txt.6
Creating in.txt.7
Creating in.txt.8
Creating in.txt.9
Creating in.txt.10
Creating in.txt.11
Creating in.txt.12
Creating in.txt.13
Creating in.txt.14
Creating in.txt.15
Creating in.txt.16
Creating in.txt.17
Creating in.txt.18
Creating in.txt.19
Creating in.txt.20
Creating in.txt.21
Creating in.txt.22
Creating in.txt.23
Creating in.txt.24
Creating in.txt.25
Creating in.txt.26
Creating in.txt.27
Creating in.txt.28
Creating in.txt.29
File split into 10 data + 20 parity shards.
经查看，30个文件已经生成，其中前10个拼接就是原始文件。


2.reed-solomon 恢复测试
在上面基础上，删掉out目录下面20个文件(编号6-24)，剩下10个，执行  go run recover_main.go 

开始恢复源文件，并恢复删掉的20个文件。
```

```go
package main
 
import (
    "fmt"
    "io"
    "os"
 
    "github.com/klauspost/reedsolomon"
)
 
var OutFile = "out2.txt"
var outFile = &OutFile
 
var DataShards = 10
var ParShards = 20
var OutDir = "./out"
 
var dataShards = &DataShards
var parShards = &ParShards
var outDir = &OutDir
 
 
 
func main() {
    fname := "out/in.txt"
 
    // 1.Create matrix
    enc, err := reedsolomon.NewStream(*dataShards, *parShards)
    checkErr(err)
 
    // 2.Open the inputs
    shards, size, err := openInput(*dataShards, *parShards, fname)
    checkErr(err)
 
    // 3.Verify the shards
    ok, err := enc.Verify(shards)
    if ok {
        fmt.Println("No reconstruction needed")
    } else {
        fmt.Println("Verification failed. Reconstructing data")
        shards, size, err = openInput(*dataShards, *parShards, fname)
        checkErr(err)
        // 3.1 重新创建删除的文件
        out := make([]io.Writer, len(shards))
        for i := range out {
            if shards[i] == nil {
                //dir, _ := filepath.Split(fname)
                outfn := fmt.Sprintf("%s.%d", fname, i)
                fmt.Println("Creating", outfn)
                out[i], err = os.Create(outfn)
                checkErr(err)
            }
        }
        fmt.Println("reconstruct")
        // 3.2 重建30个shards
        err = enc.Reconstruct(shards, out)
        if err != nil {
            fmt.Println("Reconstruct failed -", err)
            os.Exit(1)
        }
        // Close output.
        for i := range out {
            if out[i] != nil {
                err := out[i].(*os.File).Close()
                checkErr(err)
            }
        }
        shards, size, err = openInput(*dataShards, *parShards, fname)
        ok, err = enc.Verify(shards)
        if !ok {
            fmt.Println("Verification failed after reconstruction, data likely corrupted:", err)
            os.Exit(1)
        }
        checkErr(err)
    }
 
    // 4.Join the shards and write them
    outfn := *outFile
    if outfn == "" {
        outfn = fname
    }
 
    fmt.Println("Writing data to", outfn)
    f, err := os.Create(outfn)
    checkErr(err)
 
    shards, size, err = openInput(*dataShards, *parShards, fname)
    checkErr(err)
 
    // join恢复原文件 but We don't know the exact filesize.
    err = enc.Join(f, shards, int64(*dataShards)*size)
    checkErr(err)
}
 
func openInput(dataShards, parShards int, fname string) (r []io.Reader, size int64, err error) {
    // Create shards and load the data.
    shards := make([]io.Reader, dataShards+parShards)
    for i := range shards {
        infn := fmt.Sprintf("%s.%d", fname, i)
        fmt.Println("Opening", infn)
        f, err := os.Open(infn)
        if err != nil {
            fmt.Println("Error reading file", err)
            shards[i] = nil
            continue
        } else {
            shards[i] = f
        }
        stat, err := f.Stat()
        checkErr(err)
        if stat.Size() > 0 {
            size = stat.Size()
        } else {
            shards[i] = nil
        }
    }
    return shards, size, nil
}
 
func checkErr(err error)  {
    if err != nil {
        os.Exit(1)
    }
}
```

```
先删除0-5，25-29 这些文件，
然后运行recover逻辑，

$ go run recover_main.go
Verification failed. Reconstructing data
Opening out/in.txt.0
Opening out/in.txt.1
Opening out/in.txt.2
Opening out/in.txt.3
Opening out/in.txt.4
Opening out/in.txt.5
Opening out/in.txt.25
Opening out/in.txt.26
Opening out/in.txt.27
Opening out/in.txt.28
Opening out/in.txt.29


Creating out/in.txt.5
Creating out/in.txt.6
Creating out/in.txt.7
Creating out/in.txt.8
Creating out/in.txt.9
Creating out/in.txt.10
Creating out/in.txt.11
Creating out/in.txt.12
Creating out/in.txt.13
Creating out/in.txt.14
Creating out/in.txt.15
Creating out/in.txt.16
Creating out/in.txt.17
Creating out/in.txt.18
Creating out/in.txt.19
Creating out/in.txt.20
Creating out/in.txt.21
Creating out/in.txt.22
Creating out/in.txt.23
Creating out/in.txt.24


reconstruct ...

Writing data to out2.txt

最后可以看到0-5，25-29 这些文件已经恢复出来，并且源文件也恢复出来了 out2.txt.


```
## 总结
[reed solomon](github.com/klauspost/reedsolomon) 纠错码是一种特殊类型的纠错码。在事先冗余备份下，当丢失数据在一定范围内，调用恢复过程，就可以恢复源文件。
一些大型的分布式文件存储，都用它来保证文件的高可用性。当然，使用起来非常方便，大家可以多动手试试，希望你能喜欢哦！

以上所有内容均采用最新官方案例做示例

## 参考资料

- [reed solomn编码](https://blog.csdn.net/wangsiman/article/details/80101654)
- [ECC之Reed-Solomon算法](https://zhuanlan.zhihu.com/p/104306038)
- [图片上传及查看，使用swarm网关实现](https://swarm-gateways.net/)
