# 全文检索库bluge

## 推荐理由

提到全文检索库，第一个想到的就是Java实现的lucene，今天介绍一款Golang实现的全文检索库bluge。bluge脱胎于Bleve，是当前Github比较火的搜索引擎项目zinc的底层索引检索库。

## 功能介绍

bluge索引存储支持内存，本地文件，以及扩展云存储等方式，文档字段类型支持Text, Numeric, Date, Geo Point等。

查询检索支持如下特性：

1. 支持多种查询方式：term/phrase/match等基本的全文检索，数字/时间范围查询；
2. 聚合函数：Min/Max/Count/Sum/Avg/Weighted Avg；
3. 匹配高亮。

## 使用指南

### 安装

```shell
go get github.com/blugelabs/bluge
```

### 代码示例

下面是一个简单的例子：

```golang
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"github.com/blugelabs/bluge"
)

func main() {
	// write index
	writeIndex("./data/bluge/")
	// batch insert
	batch("./data/bluge/")
	// search
	search("./data/bluge/")
}

// 创建索引
func writeIndex(indexPath string) {
	config := bluge.DefaultConfig(indexPath)
	writer, err := bluge.OpenWriter(config)
	if err != nil {
		log.Fatalf("error opening writer: %v", err)
	}
	defer writer.Close()

    // 新建文档
	doc := bluge.NewDocument("example").
		AddField(bluge.NewTextField("name", "bluge")).AddField(bluge.NewDateTimeField("created_at", time.Now()))

	err = writer.Update(doc.ID(), doc)
	if err != nil {
		log.Fatalf("error updating document: %v", err)
	}
}

// 批量创建
func batch(indexPath string) {
	writer, err := bluge.OpenWriter(bluge.DefaultConfig(indexPath))
	batch := bluge.NewBatch()
	for i := 0; i < 10; i++ {
		doc := bluge.NewDocument(fmt.Sprintf("example_%d", i)).
			AddField(bluge.NewTextField(fmt.Sprintf("field_%d", i), fmt.Sprintf("value_%d", i%2))).AddField(bluge.NewDateTimeField("created_at", time.Now()))
		batch.Insert(doc)
	}
	err = writer.Batch(batch)
	if err != nil {
		log.Fatalf("error executing batch: %v", err)
	}
	batch.Reset()
}

// 查询
func search(indexPath string) {
	config := bluge.DefaultConfig(indexPath)
	reader, err := bluge.OpenReader(config)

	if err != nil {
		log.Fatalf("error getting index reader: %v", err)
	}
	defer reader.Close()

	query := bluge.NewMatchQuery("value_1").SetField("field_1")
	request := bluge.NewTopNSearch(10, query).
		WithStandardAggregations()
	documentMatchIterator, err := reader.Search(context.Background(), request)
	if err != nil {
		log.Fatalf("error executing search: %v", err)
	}
	match, err := documentMatchIterator.Next()
	for err == nil && match != nil {
		err = match.VisitStoredFields(func(field string, value []byte) bool {
			fmt.Printf("match: %s:%s\n", field, string(value))
			return true
		})
		if err != nil {
			log.Fatalf("error loading stored fields: %v", err)
		}
		fmt.Println(match)
		match, err = documentMatchIterator.Next()
	}
	if err != nil {
		log.Fatalf("error iterator document matches: %v", err)
	}
}

```

## 总结

bulge是Golang实现的全文检索库，功能上类似lucene，性能上相比lucene还有些差距，如果对全文检索感兴趣可以把玩把玩。

## 参考资料

1. [https://github.com/blugelabs/bluge](https://github.com/blugelabs/bluge)
2. [https://blugelabs.com/bluge/](https://blugelabs.com/bluge/)