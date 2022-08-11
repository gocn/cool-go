# Golang的Elastic链接库

## 背景介绍
Elasticsearch是一个分布式、高扩展、高实时的搜索与数据分析引擎，用于海量文档的搜索。有些项目会将Elasticsearch当做存储海量数据的数据库使用，可见其查询性能之高效。作为面向文档的搜索引擎，Elasticsearch比起传统数据库更偏向于结构化数据的高效查询，其独特的倒排索引更能将查询性能提升至极致。在大数据微服务时代，Elasticsearch在海量数据搜索、数据挖掘、人工智能领域都起到了关键作用。

## 安装
```shell
go get "github.com/olivere/elastic/v7"
```
Elasticsearch的数据来源通常来自于Logstash等数据采集中间件，作为golang项目来说，其查询功能的使用更加普遍。
此文章以V7版本为例来介绍如何使用golang对Elasticsearch进行查询。

## 开源库的使用
### 连接客户端构建
```go
import elasticv7 "github.com/olivere/elastic/v7"

address := []string{"http://127.0.0.1:9200"}
cli, err := elasticv7.NewClient(
	elasticv7.SetURL(address...),
	elasticv7.SetBasicAuth("elastic", "123456"),
	elasticv7.SetSniff(false),
)
```
* address 为集群的地址
* SetBaseicAuth 接受UserName和Password作为参数完成校验
* Sniff参数为true,创建的客户端会去嗅探整个集群，此动作会使用内网IP通信，导致无法连接到ES服务器，这里设置为false。

### 创建查询语句
#### 精确查询
```go
// 单值查询
elasticv7.NewTermQuery("key","value")

// 多值查询
elasticv7.NewTermsQuery("key", []string{"value1","value2"}...)
```
精确查询要注意字符串类型的匹配，若为text字段，将匹配失败。可以尝试对"{字段}.keyword"来进行Term查询
#### 通配符查询
```go
elasticv7.NewWildcardQuery(key, word)
```
通配符查询通常用于模糊查询，例如"\*xxxx\*",等价于mysql中的like "%xxxx%"。
#### 与查询
```go
query := elasticv7.NewBoolQuery()
query.Must(queries ...)
```
与查询使用BoolQuery的Must函数来完成，其参数是类型为query的不定参数。当所有query均为真时此条件为真，可嵌套。
#### 或查询
```go
elasticv7.NewWildcardQuery("", "")
query.Should(queries ...)
```
与Must相似。
#### 创建查询服务
```go
search := cli.Search().Index(index_name).Query(query)
```
index_name是ES中的索引，类比Mysql相当于表Table的概念。
query为查询对象，以上各种查询可相互嵌套形成最终的查询对象。
#### 分页
```go
search = search.From(10)
search = search.Size(10)
```
这里search中的函数都是链式的，可分行写亦可整行写。
#### 排序
```go
search = search.Sort(key,true)
```
排序的第一参数为排序字段，第二参数为是否正序。
#### 跳过评分计算
```go
constantQuery := elasticv7.NewConstantScoreQuery(query)
```
评分会降低查询的效率，当不需要时可以跳过。

## 与Sql的转换
*select * from t where a = 1 and b = 2;*
```go
elasticv7.NewBoolQuery().Must(elasticv7.NewTermQuery("a", 1), elasticv7.NewTermQuery("b", 2))
```

*select * from t where a = 1 or a = 2;*
```go
elasticv7.NewBoolQuery().Must(elasticv7.NewTermsQuery("a", 1, 2))
```

*select * from t where a = 1 or b = 2;*
```go
elasticv7.NewBoolQuery().Should(elasticv7.NewTermQuery("a", 1), elasticv7.NewTermQuery("b", 2))
```

*select * from t where a like '%m%' and b like '%n%';*
```go
elasticv7.NewBoolQuery().Must(elasticv7.NewWildcardQuery("a", "*m*"), elasticv7.NewWildcardQuery("b", "*n*"))
```

*select name,age from t;*
```go
cli.Search().Index("t").Source([]string{"name","age"})
```

*select * from t limit 2 offset 1;*
```go
cli.Search().Index("t").From(1).Size(2)
```

## 获取查询结果
使用Each获得结果，类型为[]interface{}，再使用断言得到所需的结构体。
```go
var user User
result := cli.Search().Index("t").Do(context.Background())
users := result.Each(reflect.TypeOf(user))
for _,v := range users {
	u , ok := v.(User)
}
```

## 总结
[官方库](https://github.com/elastic/go-elasticsearch)需要自己去构造查询的json结构，使用起来较为混乱，不易理解。相较而言，此开源库采用链式可嵌套的形式来构造查询对象，使用起来更加清晰便捷。其源码库中亦有相当多的各类函数和对象用于各种条件查询，此次只是摘取本人使用时设计过的些许功能加以介绍。

## 参考资料
1、[https://segmentfault.com/a/1190000039140870](https://segmentfault.com/a/1190000039140870)

2、[https://pkg.go.dev/github.com/elastic/go-elasticsearch/v6@v6.8.5/esapi](https://pkg.go.dev/github.com/elastic/go-elasticsearch/v6@v6.8.5/esapi)
