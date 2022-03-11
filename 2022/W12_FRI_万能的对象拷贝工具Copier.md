# 万能的对象拷贝工具Copier
## 推荐理由
在DDD的领域驱动的设计之下，数据流向由DTO到DO再到PO，在Golang语言中，不同类型的转换由独立的函数或者方法来实现。随着对象类型的增加，对象字段的增加以及复杂化，各种类型之间的转换方法无论是数量还是复杂度都会急剧增加。Copier库提供了对象之间的相互转换方法Copy(target,srouce interface{}) error用于解决数据在不同类型结构之间的传递问题。

## 功能介绍
对象这里指的是数据的聚合：数组，结构体，字典。Copier对对象之间的转换通过reflect提供的方法来完成，实现了从结构体到结构体，结构体到数组的转换方法，其中结构体到数组使用的是覆盖方式。
## 安装
```shell
go get github.com/jinzhu/copier
```

## 使用指南
### 基础功能之结构体对结构体赋值
```go
package main

import (
	"fmt"

	"github.com/jinzhu/copier"
)


type User struct{
	Name string 
	Age int 
	UserExtend string		
}

type Friend struct {
	Name string
	Age int 
	FriendExtend string
}

func main() {
	user := User{}
	friend := Friend{
		Name:"xiaoming",
		Age: 18,
		FriendExtend:"good friend",	
	}
	copier.Copy(&user,&friend). //前者是target，后者是source
	fmt.Println(user). // {xiaoming 18 }
}
```
这里可以看到结构体中相同字段名完成了赋值。光是如此，局限性还是很大的，所以结合Go语言结构体的特点，Copier库有如下特性：
1、可利用tag改变等价判定的条件,例如:`copier:"OtherName"`可以在判定时与字段OtherName匹配。
2、source可使用方法来改变等价判定的条件，例如func(xx * Friend)OtherName()string 可以在判定时与字段OtherName匹配。
3、target可使用方法来改变接受赋值的行为，例如func(xx * Friend)OtherName(string) 可以在接收OtherName字段时调用。
4、可利用tag设定忽略字段，例如 `copier:"-"`。
5、可以设置转换时的必填项等,例如:  `copier:"must,nopanic"`等。
6、支持嵌套结构体之间的对应转换。

### 结构体(数组)向数组切片的转换
在基础类型的转换之上，数组与数组之间的转换变得顺理成章。Copier提供了结构体向数组的转换，知乎上的文章有提到是追加操作，在阅读源码后，这里其实是覆盖操作。
1、当len(src) > len(target)时，会依次覆盖并扩展。
2、当len(src) < len(target)时，会依次覆盖保留未覆盖。

### 额外的转换函数
Go是强类型的语言，针对不同基础类型的转换在对象转换中也是不可避免的，Copier提供了CopyWithOption函数为转换过程提供额外的配置。
```go
// Option sets copy options

type Option struct {

	// setting this value to true will ignore copying zero values of all the fields, including bools, as well as a
	
	// struct having all it's fields set to their zero values respectively (see IsZero() in reflect/value.go)
	
	IgnoreEmpty bool
	
	DeepCopy bool
	
	Converters []TypeConverter

}
```
* IgnoreEmpty 会将所有的零值忽略
* DeepCopy会使用深拷贝
* Converters 设定特定的类型转换形式
```go
copier.TypeConverter{
{
	SrcType: time.Time{},
	DstType: copier.String,
	Fn: func(src interface{}) (interface{}, error) {
		s, ok := src.(time.Time)
		if !ok {
			return nil, errors.New("src type not matching")
		}
	return s.Format(time.RFC3339), nil
},
```
TypeConverter是一个结构体，其中是源类型，目标类型以及之间转换函数的实现。

## 总结
Copier是一个非常实用的第三方库，随着领域驱动思想的成熟与落地，数据结构的复杂，Go语言中复杂类型之间的转换愈加频繁，Copier库本身源码并不多，但是却能在工作中为你提供极大的便利性。虽然相比于直接编写转换函数，reflect的使用会在性能上有所损耗，但是随着Go语言的版本升级性能优化以及硬件设备配置的提高，这样损耗的影响将微乎其微，能覆盖大部分的使用场景需求。

## 参考资料
[ https://zhuanlan.zhihu.com/p/113301827](https://zhuanlan.zhihu.com/p/113301827)
[https://github.com/jinzhu/copier/blob/master/README.md](https://github.com/jinzhu/copier/blob/master/README.md)