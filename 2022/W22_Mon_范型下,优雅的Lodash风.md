# 范型下，优雅的 Lodash 风

## 推荐理由

go 语言比较崇尚简单，所以在内嵌包中没有提供过多帮助性的函数，尤其在范型出来前想要写一个简单的对 slice 和 map 间互相转化的方法可能就需要很多行代码，并且这样的并不是很优雅。如今，go 范型已经得到官方的正式发布，samber/lo 又提供了优雅的 lodash 风格的工具函数，正是代码重构的好时机。

## 常见的使用场景

> 这里只是用 slice 和 map 两个常见结构举个栗子，lo 中支持超多的转化帮助函数供开发使用。
> 

### slice → map

在批量接口中，request 中携带 idList ，返回 map response

```go

struct Request {
	IDList []int64
}

struct Response {
	Metas map[int64]Meta
}

// 用于模拟数据库查询数据
struct Meta {
	ID int64
	Name string
}

func MGetMeta(req Request) Response{
	result := make([]Meta,0, len(req.IDList))
	// 执行 db query
	db.Raw("select * from meta where id in ?", req.IDList).Scan(&result)
	return Response{
		Metas: MetaResult2Map(result),
	}
}

// 旧代码: 针对不同的类型需要写不同的转化函数
func MetaResult2Map(metas []Meta)map[int64]Meta{
	res := make(map[int64]Meta)
	for _, m := range metas{
		res[m.ID] = m
	}
	return res
}

// 采用 lo:代码清晰，支持多种结构
func MGetMeta(req Request) Response{
	result := make([]Meta,0, len(req.IDList))
	// 执行 db query
	db.Raw("select * from meta where id in ?", req.IDList).Scan(&result)
	return Response{
		Metas: lo.KeyBy[int64,Meta](a, func(m Meta)int64{
			return m.ID
		}),
	}
}
```

### map → slice

```go
// 在对批量接口接口去重后，可能得到这样类型参数 map[int64]bool
// 然后在db查询时其实只用map 的 key 的 slice
// 旧代码实现
func Int64MapKeys(m map[int64]interface{}) []int64{
	res := make([]int64,0,len(m))
	for k := range m{
		res = append(res, k)
	}
	return res
}

// 使用 lo
lo.Keys[int64,bool](map[int64]bool{1: true,2:true,3:true})

// 批量的聚合接口
// 比如 定时任务中，扫表获取了一堆数据，分属于不同的业务方需要分别调用
// 此时参数转化为 []T -> map[K][]T ，安装 T 中的某个字段进行分组
func Group(m []T)map[K][]T{
	res := make(map[K][]T)
	for _, v := range m{
		if _, ok := res[v.Key];ok {
			res[v.Key] = append(res[v.Key],v)
		}else{
			res[v.Key] = []T{v}
		}
	}
	return res
}

// 使用 lo
lo.GroupBy([]Meta{{ID: 1,Name: "1"},{ID: 2, Name: "2"},{ID: 1,Name: "3"}}, func(t Meta) int64 {
		return t.ID
})
```

## 总结

samber/lo 使用的开源 MIT 协议，其中提供了很多转化数据结构的方法，让日常开发更加方便优雅，并且中的代码并不复杂，并且全部采用 go1.18 范型标注，对于初学者而言也是个学习范型的好地方。笔者就是一边自己实现，一边对照 samber/lo 实现学习完这个库的。

## 参考

文档：[https://pkg.go.dev/github.com/samber/lo#KeyBy](https://pkg.go.dev/github.com/samber/lo#KeyBy)

github： [https://github.com/samber/lo](https://github.com/samber/lo)‣