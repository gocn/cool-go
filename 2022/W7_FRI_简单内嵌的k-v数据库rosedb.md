# 内嵌k-v数据库 rosedb

## 1.前言

通常，每当我们用到数据库时候，都会想起redis、memcache、mysql等等，这些数据库都是独立于我们的服务进程，需要单独的管理。

本文推荐一个内嵌的，简单的k-v 数据库rosedb，不需要单独管理数据库，直接import导入，就可以直接使用，非常方便。

rosedb 使用Golang实现，支持多种数据结构，包含 String、List、Hash、Set、Sorted Set，接口名称风格和 Redis 类似，如果你对 Redis 比较熟悉，那么使用起来会毫无违和感。

## 2.rosedb 特性
* 支持丰富的数据结构：字符串、列表、哈希表、集合、有序集合。
* 内嵌使用简单至极，无需任何安装部署（import "github.com/roseduan/rosedb"）。
* 低延迟、高吞吐（具体请见英文 README 的 Benchmark）。
* 不同数据类型的操作可以完全并行。
* 支持客户端命令行操作。
* 支持过期时间。
* String 数据类型支持前缀和范围扫描。
* 支持简单的事务操作，ACID 特性。
* 数据文件 merge 可手动停止。

## 3.rosedb 支持命令
String
* Set、SetNx、Get、GetSet、Append、StrLen、StrExists、StrRem、PrefixScan、RangeScan、Expire、Persist、TTL

List
* LPush、RPush、LPop、RPop、LIndex、LRem、LInsert、LSet、LTrim、LRange、LLen

Hash
* HSet、HSetNx、HGet、HGetAll、HDel、HExists、HLen、HKeys、HValues

Set
* SAdd、SPop、SIsMember、SRandMember、SRem、SMove、SCard、SMembers、SUnion、SDiff

Zset
* ZAdd、ZScore、ZCard、ZRank、ZRevRank、ZIncrBy、ZRange、ZRevRange、ZRem、ZGetByRank、ZRevGetByRank、ZScoreRange、ZRevScoreRange


## 4.命令行使用举例
1.按照protobuf及相关插件；并gen_pb_go.sh 生成相关pb文件
```
$ go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
$ sh gen_pb_go.sh 
```
2.切换目录到 rosedb/cmd/server
```
$ go run main.go 
  _____     ____     _____   ______   _____    ____
 |  __ \   / __ \   / ____| |  ____| |  __ \  |  _ \
 | |__) | | |  | | | (___   | |__    | |  | | | |_) |
 |  _  /  | |  | |  \___ \  |  __|   | |  | | |  _ <
 | | \ \  | |__| |  ____) | | |____  | |__| | | |_) |
 |_|  \_\  \____/  |_____/  |______| |_____/  |____/

2022/02/11 16:03:29 no config set, using the default config.
2022/02/11 16:03:29 no dir path set, using the os tmp dir.
2022/02/11 16:03:29 rosedb is running, ready to accept connections.
2022/02/11 16:03:29 grpc server serve in addr 127.0.0.1:5300

```

3.打开一个新的窗口，切换目录到 rosedb/cmd/cli
```
$ go run main.go 
127.0.0.1:5200>
127.0.0.1:5200>set name zhangsan
OK
127.0.0.1:5200>get name
zhangsan
127.0.0.1:5200>
127.0.0.1:5200>hset zhangsan math 100
(integer) 0 
127.0.0.1:5200>hset zhangsan english 80
(integer) 0 
127.0.0.1:5200>hkeys zhangsan
1) math
2) english
127.0.0.1:5200>hvals zhangsan
1) 100
2) 80
127.0.0.1:5200>

```

## 5.直接内嵌使用举例

```go
package main

import (
	"fmt"
	"log"

	"github.com/roseduan/rosedb"
)

func main() {
	config := rosedb.DefaultConfig()
	db, err := rosedb.Open(config)
	if err != nil {
		log.Fatal(err)
	}

	var s string
	db.Set("str", "hello world")
	db.Get("str", &s)
	fmt.Printf("str = %s \n\n", s)

	db.LPush("mylist", "apple", "pear", "banana")
	length := db.LLen("mylist")
	fmt.Printf("len(mylist) = %v \n", length)
	li, _ := db.LRange("mylist", 0, 3)
	fmt.Printf("mylist = %v \n\n", li)

	db.HSet("zhangsan", "math", 100)
	db.HSet("zhangsan", "english", 90)
	keys := db.HKeys("zhangsan")
	fmt.Printf("zhangsan.keys = %v \n", keys)
	vals := db.HVals("zhangsan")
	fmt.Printf("zhangsan.vals = %v \n\n", vals)

	defer db.Close()
}
```

执行，控制台输出如下：
```
$ go run test.go 
str = hello world 

len(mylist) = 21 
mylist = [[98 97 110 97 110 97] [112 101 97 114] [97 112 112 108 101] [98 97 110 97 110 97]] 

zhangsan.keys = [math english] 
zhangsan.vals = [[100] [90]] 

```


## 6.总结

`rosedb` 是一个内嵌的，简单的k-v 数据库rosedb，直接import导入就可以直接使用；同时它类似redis命令行，和api的使用方式，非常方便；如果你也想直接嵌入代码使用，不妨试试看，相信一定会喜欢上的！

## 参考资料

* [https://github.com/flower-corp/rosedb/blob/main/README-CN.md](https://github.com/flower-corp/rosedb/blob/main/README-CN.md)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)
