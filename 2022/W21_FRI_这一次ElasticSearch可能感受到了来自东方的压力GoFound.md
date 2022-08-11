# 单机亿级数据毫秒级查找的搜索引擎GoFound

## 什么是 GoFound?

`GoFound` 一个golang实现的全文检索引擎，支持持久化和单机亿级数据毫秒级查找。 接口可以通过http调用

## 为什么要用GoFound?

一个小巧精悍的全文检索引擎，支持持久化和单机亿级数据毫秒级查找。

`ElasticSearch`缺点就是配置繁琐、基于JVM对内存消耗比较大。

`gofound`是原生编译，会减少系统资源的消耗。而且对外无任何依赖。

## 特性与优势

- 二分法查找

- 快速排序法

- 倒排索引

- 正排索引

- 文件分片

- golang-jieba分词

- leveldb

### 编译与启动

```shell
git clone https://github.com/newpanjing/gofound.git
go get && go build

#作者是用的1.18版本，已经用到了1.17的新特性embed 和 fs，所以GO至少版本要大于1.17

./gofound --addr=:8080 --data=./data
```

1. 启动服务端后就可以访问到后台

```shell
http://127.0.0.1:8080/admin
#笔者发现，明显后台没有做完，但是接口基本可用，尽量别用后台就行。
```

![](https://cdn.gocn.vip/forum-user-images/20220520/b2665a1fdadc4f1f86da6ad79c19ae3d.jpg)

2. 使用接口进行新增和查询操作

```shell
#新增数据接口
curl -H "Content-Type:application/json" -X POST --data '{"id":1,"text":"深圳北站","document":{"title":"阿森松岛所445","number":223}}' 127.0.0.1:8080/api/index?database=testdb1

#响应结果
{
  "state": true,
  "message": "success"
}

#我定义了一个库名：testdb1
```

```shell
#查询一下
POST ： 127.0.0.1:8080/api/query?database=testdb1

{
  "query": "北站",
  "page": 1,
  "limit": 10,
  "order": "desc",
  "highlight": {}
}

#响应结果
{
    "state": true,
    "message": "success",
    "data": {
        "total": 3,
        "pageCount": 1,
        "page": 1,
        "limit": 10,
        "documents": [
            {
                "id": 1,
                "text": "深圳北站",
                "document": {
                    "number": 223,
                    "title": "阿森松岛所445"
                },
                "originalText": "深圳北站",
                "score": 1,
                "keys": [
                    "深圳",
                    "北站"
                ]
            },
            {
                "text": "深圳北站",
                "document": {
                    "number": 223,
                    "title": "阿森松岛所445"
                },
                "originalText": "深圳北站",
                "score": 1,
                "keys": [
                    "深圳",
                    "北站"
                ]
            }
        ],
        "words": [
            "北站"
        ]
    }
}
```

3. 其他操作，包括完整的CRUD还有分词接口都在下面的文档中可以找到，这里就不多作赘述，简单的带大家用用即可。

文档：[gofound · GitHub](https://github.com/newpanjing/gofound/blob/main/docs/api.md)

还有分词操作，CRUD所有操作，都可以看文档来对着写即可，记得尽量不要用他的可视化后台，会出现各种未知问题。

## 参考链接

* [GitHub - newpanjing/gofound: GoFound GoLang Full text search go语言全文检索引擎 基于平衡二叉树+正排索引、倒排索引实现 可支持亿级数据，毫秒级查询。 使用简单，使用http接口，任何系统都可以使用。](https://github.com/newpanjing/gofound)