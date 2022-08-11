# Go高性能多语言NLP和分词库gse

## gse是什么？
Go高性能多语言NLP和分词库, 支持英文、中文、日文等, 支持接入 elasticsearch 和 bleve
Gse是结巴分词(jieba)的golang实现,并尝试添加NLP功能和更多属性

## 特征
- 支持普通、搜索引擎、全模式、精确模式和 HMM 模式多种分词模式
- 支持自定义词典、embed 词典、词性标注、停用词、整理分析分词
- 多语言支持: 英文, 中文, 日文等
- 支持繁体字
- NLP 和 TensorFlow 支持 (进行中)
- 命名实体识别 (进行中)
- 支持接入 Elasticsearch 和 bleve
- 可运行 JSON RPC 服务

## 算法
- 词典用双数组 trie（Double-Array Trie）实现，
- 分词器算法为基于词频的最短路径加动态规划, 以及 DAG 和 HMM 算法分词.
- 支持 HMM 分词, 使用 viterbi 算法.

## 分词速度:
- 单线程 9.2MB/s
- goroutines 并发 26.8MB/s.
- HMM 模式单线程分词速度 3.2MB/s.（双核 4 线程 Macbook Pro）。


## 快速入门
```go
package main

import (
	"fmt"
	"regexp"

	"github.com/go-ego/gse"
	"github.com/go-ego/gse/hmm/pos"
)

var (
	seg gse.Segmenter
	posSeg pos.Segmenter

	new, _ = gse.New("zh,testdata/test_dict3.txt", "alpha")

	text = "你好世界, Hello world, Helloworld."
)

func main() {
	// 加载默认词典
	seg.LoadDict()
	// 加载默认 embed 词典
	// seg.LoadDictEmbed()
	//
	// 加载简体中文词典
	// seg.LoadDict("zh_s")
	// seg.LoadDictEmbed("zh_s")
	//
	// 加载繁体中文词典
	// seg.LoadDict("zh_t")
	//
	// 加载日文词典
	// seg.LoadDict("jp")
	//
	// 载入词典
	// seg.LoadDict("your gopath"+"/src/github.com/go-ego/gse/data/dict/dictionary.txt")

	cut()

	segCut()
}


func cut() {
	hmm := new.Cut(text, true)
	fmt.Println("cut use hmm: ", hmm)

	hmm = new.CutSearch(text, true)
	fmt.Println("cut search use hmm: ", hmm)
	fmt.Println("analyze: ", new.Analyze(hmm, text))

	hmm = new.CutAll(text)
	fmt.Println("cut all: ", hmm)

	reg := regexp.MustCompile(`(\d+年|\d+月|\d+日|[\p{Latin}]+|[\p{Hangul}]+|\d+\.\d+|[a-zA-Z0-9]+)`)
	text1 := `헬로월드 헬로 서울, 2021年09月10日, 3.14`
	hmm = seg.CutDAG(text1, reg)
	fmt.Println("Cut with hmm and regexp: ", hmm, hmm[0], hmm[6])
}

func analyzeAndTrim(cut []string) {
	a := seg.Analyze(cut, "")
	fmt.Println("analyze the segment: ", a)

	cut = seg.Trim(cut)
	fmt.Println("cut all: ", cut)

	fmt.Println(seg.String(text, true))
	fmt.Println(seg.Slice(text, true))
}

func cutPos() {
	po := seg.Pos(text, true)
	fmt.Println("pos: ", po)
	po = seg.TrimPos(po)
	fmt.Println("trim pos: ", po)

	posSeg.WithGse(seg)
	po = posSeg.Cut(text, true)
	fmt.Println("pos: ", po)

	po = posSeg.TrimWithPos(po, "zg")
	fmt.Println("trim pos: ", po)
}

func segCut() {
	// 分词文本
	tb := []byte("山达尔星联邦共和国联邦政府")

	// 处理分词结果
	fmt.Println("输出分词结果, 类型为字符串, 使用搜索模式: ", seg.String(string(tb), true))
	fmt.Println("输出分词结果, 类型为 slice: ", seg.Slice(string(tb)))

	segments := seg.Segment(tb)
	// 处理分词结果, 普通模式
	fmt.Println(gse.ToString(segments))

	segments1 := seg.Segment([]byte(text))
	// 搜索模式
	fmt.Println(gse.ToString(segments1, true))
}

```
输出结果：
```
cut use hmm:  [你好 世界 ,  hello   world ,  helloworld .]
cut search use hmm:  [你好 世界 ,  hello   world ,  helloworld .]
analyze:  [{0 6 0 0  你好 725 l} {6 12 1 0  世界 34387 n} {25 27 2 0  ,  0 } {27 32 3 0  hello 0 } {26 27 4 0    0 } {32 37 5 0  world 0 } {12 14 6 0  ,  0 } {27 37 7 0  helloworld 0 } {37 38 8 0  . 0 }]
cut all:  [你好 世界 ,   h e l l o   w o r l d ,   h e l l o w o r l d .]
Cut with hmm and regexp:  [헬로월드   헬로   서울 ,  2021年 09月 10日 ,  3.14] 헬로월드 2021年
输出分词结果, 类型为字符串, 使用搜索模式:  山/n 达尔/nrt 星/n 联邦/n 共和/nz 国/zg 共和国/ns 联邦/n 政府/n 联邦政府/nt 
输出分词结果, 类型为 slice:  [山 达尔 星 联邦 共和国 联邦政府]
山/n 达尔/nrt 星/n 联邦/n 共和国/ns 联邦政府/nt 
你好/l 世界/n ,/x  /x hello/x  /x world/x ,/x  /x helloworld/x ./x 
```

更多用法可参考github上官方用例

## 参考资料
* https://github.com/go-ego/gse/blob/master/README_zh.md
