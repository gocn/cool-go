# 伪数据生成器 bxcodec/faker

## bxcodec/faker 是什么？

日常开发中通常需要大批量生成伪数据，用来测试程序。Go 开发中通常用 [`bxcodec/faker`](https://github.com/bxcodec/faker) 来批量生成测试数据。[`bxcodec/faker`](https://github.com/bxcodec/faker)  功能齐全，测试充分，支持多种语言（包含中文），足够满足日常开发需求，它支持的数据类型如下：
- int, int8, int16, int32 & int64
- []int, []int8, []int16, []int32 & []int64
- bool & []bool
- string & []string
- float32, float64, []float32 &[]float64
- time.Time & []time.Time
- 以及各种嵌套的结构体字段

> 注意📢：**生成数据的结构体字段必须是公开，否则会引起程序恐慌(异常)**。

为生成数据而提供的方法如下：

```go
package main

import "github.com/bxcodec/faker/v3"

// 单一的假函数可用于检索特定的值。
func Example_singleFakeData() {

	// 地址
	faker.Latitude()  // => 81.12195
	faker.Longitude() // => -84.38158

	// 时间
	faker.UnixTime()   // => 1197930901
	faker.Date()       // => 1982-02-27
	faker.TimeString() // => 03:10:25
	faker.MonthName()  // => February
	faker.YearString() // => 1994
	faker.DayOfWeek()  // => Sunday
	faker.DayOfMonth() // => 20
	faker.Timestamp()  // => 1973-06-21 14:50:46
	faker.Century()    // => IV
	faker.Timezone()   // => Asia/Jakarta
	faker.Timeperiod() // => PM

	// 网络
	faker.Email()      // => mJBJtbv@OSAaT.com
	faker.MacAddress() // => cd:65:e1:d4:76:c6
	faker.DomainName() // => FWZcaRE.org
	faker.URL()        // => https://www.oEuqqAY.org/QgqfOhd
	faker.Username()   // => lVxELHS
	faker.IPv4()       // => 99.23.42.63
	faker.IPv6()       // => 975c:fb2c:2133:fbdd:beda:282e:1e0a:ec7d
	faker.Password()   // => dfJdyHGuVkHBgnHLQQgpINApynzexnRpgIKBpiIjpTP

	// 词汇和句子
	faker.Word()      // => nesciunt
	faker.Sentence()  // => Consequatur perferendis voluptatem accusantium.
	faker.Paragraph() // => Aut consequatur sit perferendis accusantium voluptatem. Accusantium perferendis consequatur voluptatem sit aut. Aut sit accusantium consequatur voluptatem perferendis. Perferendis voluptatem aut accusantium consequatur sit.

	// 支付
	faker.CCType()             // => American Express
	faker.CCNumber()           // => 373641309057568
	faker.Currency()           // => USD
	faker.AmountWithCurrency() // => USD 49257.100

	// 人物
	faker.TitleMale()       // => Mr.
	faker.TitleFemale()     // => Mrs.
	faker.FirstName()       // => Whitney
	faker.FirstNameMale()   // => Kenny
	faker.FirstNameFemale() // => Jana
	faker.LastName()        // => Rohan
	faker.Name()            // => Mrs. Casandra Kiehn

	// 电话号
	faker.Phonenumber()         // -> 201-886-0269
	faker.TollFreePhoneNumber() // => (777) 831-964572
	faker.E164PhoneNumber()     // => +724891571063

	//  UUID
	faker.UUIDHyphenated() // => 8f8e4463-9560-4a38-9b0c-ef24481e4e27
	faker.UUIDDigit()      // => 90ea6479fd0e4940af741f0a87596b73

	// 唯一值
	faker.SetGenerateUniqueValues(true) // 在单一的假数据功能上实现唯一的数据生成
	faker.Word()
	faker.SetGenerateUniqueValues(false) // 禁用单一假数据功能上的唯一数据生成
	faker.ResetUnique()                  // 用来重置所有产生的唯一值

}
```

## 怎么使用 bxcodec/faker ?

第一步：在项目中安装

```go
go get github.com/bxcodec/faker/v3
```

第二步：项目中导入它并使用

- 不带标签的结构体示例：

```go
package main

import (
	"fmt"

	"github.com/bxcodec/faker/v3"
)

type FirstStruct struct {
	Image string
}

type SecondStruct struct {
	Number        int64
	Height        int64
	AnotherStruct FirstStruct
}

type SomeStruct struct {
	Int      int
	Int8     int8
	Int16    int16
	Int32    int32
	Int64    int64
	String   string
	Bool     bool
	SString  []string
	SInt     []int
	SInt8    []int8
	SInt16   []int16
	SInt32   []int32
	SInt64   []int64
	SFloat32 []float32
	SFloat64 []float64
	SBool    []bool
	Struct   FirstStruct
}

func main() {
	a := SomeStruct{}
	err := faker.FakeData(&a)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Printf("%+v", a)
}
```

- 带标签的结构体示例：

```go
package main

import (
	"fmt"

	"github.com/bxcodec/faker/v3"
)

type SomeStructWithTags struct {
	Latitude           float32 `faker:"lat"`
	Longitude          float32 `faker:"long"`
	CreditCardNumber   string  `faker:"cc_number"`
	CreditCardType     string  `faker:"cc_type"`
	Email              string  `faker:"email"`
	DomainName         string  `faker:"domain_name"`
	IPV4               string  `faker:"ipv4"`
	IPV6               string  `faker:"ipv6"`
	Password           string  `faker:"password"`
	Jwt                string  `faker:"jwt"`
	PhoneNumber        string  `faker:"phone_number"`
	MacAddress         string  `faker:"mac_address"`
	URL                string  `faker:"url"`
	UserName           string  `faker:"username"`
	TollFreeNumber     string  `faker:"toll_free_number"`
	E164PhoneNumber    string  `faker:"e_164_phone_number"`
	TitleMale          string  `faker:"title_male"`
	TitleFemale        string  `faker:"title_female"`
	FirstName          string  `faker:"first_name"`
	FirstNameMale      string  `faker:"first_name_male"`
	FirstNameFemale    string  `faker:"first_name_female"`
	LastName           string  `faker:"last_name"`
	Name               string  `faker:"name"`
	UnixTime           int64   `faker:"unix_time"`
	Date               string  `faker:"date"`
	Time               string  `faker:"time"`
	MonthName          string  `faker:"month_name"`
	Year               string  `faker:"year"`
	DayOfWeek          string  `faker:"day_of_week"`
	DayOfMonth         string  `faker:"day_of_month"`
	Timestamp          string  `faker:"timestamp"`
	Century            string  `faker:"century"`
	TimeZone           string  `faker:"timezone"`
	TimePeriod         string  `faker:"time_period"`
	Word               string  `faker:"word"`
	Sentence           string  `faker:"sentence"`
	Paragraph          string  `faker:"paragraph"`
	Currency           string  `faker:"currency"`
	Amount             float64 `faker:"amount"`
	AmountWithCurrency string  `faker:"amount_with_currency"`
	UUIDHypenated      string  `faker:"uuid_hyphenated"`
	UUID               string  `faker:"uuid_digit"`
	Skip               string  `faker:"-"`
	PaymentMethod      string  `faker:"oneof: cc, paypal, check, money order"` // oneof will randomly pick one of the comma-separated values supplied in the tag
	AccountID          int     `faker:"oneof: 15, 27, 61"`                     // use commas to separate the values for now. Future support for other separator characters may be added
	Price32            float32 `faker:"oneof: 4.95, 9.99, 31997.97"`
	Price64            float64 `faker:"oneof: 47463.9463525, 993747.95662529, 11131997.978767990"`
	NumS64             int64   `faker:"oneof: 1, 2"`
	NumS32             int32   `faker:"oneof: -3, 4"`
	NumS16             int16   `faker:"oneof: -5, 6"`
	NumS8              int8    `faker:"oneof: 7, -8"`
	NumU64             uint64  `faker:"oneof: 9, 10"`
	NumU32             uint32  `faker:"oneof: 11, 12"`
	NumU16             uint16  `faker:"oneof: 13, 14"`
	NumU8              uint8   `faker:"oneof: 15, 16"`
	NumU               uint    `faker:"oneof: 17, 18"`
}

func main() {

	a := SomeStructWithTags{}
	err := faker.FakeData(&a)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Printf("%+v", a)
}
```

- 带标签指定字段长度的结构体示例
：
```go
package faker_test

import (
	"fmt"

	"github.com/bxcodec/faker/v3"
)

// 你可以为你的随机字符串设置长度，也可以为你的整数设置边界。
func main() {
	type SomeStruct struct {
		Inta  int   `faker:"boundary_start=5, boundary_end=10"`
		Int8  int8  `faker:"boundary_start=100, boundary_end=1000"`
		Int16 int16 `faker:"boundary_start=123, boundary_end=1123"`
		Int32 int32 `faker:"boundary_start=-10, boundary_end=8123"`
		Int64 int64 `faker:"boundary_start=31, boundary_end=88"`

		UInta  uint   `faker:"boundary_start=35, boundary_end=152"`
		UInt8  uint8  `faker:"boundary_start=5, boundary_end=1425"`
		UInt16 uint16 `faker:"boundary_start=245, boundary_end=2125"`
		UInt32 uint32 `faker:"boundary_start=0, boundary_end=40"`
		UInt64 uint64 `faker:"boundary_start=14, boundary_end=50"`

		ASString []string          `faker:"len=50"`
		SString  string            `faker:"len=25"`
		MSString map[string]string `faker:"len=30"`
		MIint    map[int]int       `faker:"boundary_start=5, boundary_end=10"`
	}

	_ = faker.SetRandomMapAndSliceSize(20) // 随机生成的数据大小不会超过20个...
	a := SomeStruct{}
	_ = faker.FakeData(&a)
	fmt.Printf("%+v", a)
	}
```

执行程序打印出生成的数据。

[`bxcodec/faker`](https://github.com/bxcodec/faker)  提供的功能不止以上几种，还提供指定语言格式生成字符串数据，使用非常简单。

## 总结

[`bxcodec/faker`](https://github.com/bxcodec/faker)  基本能满足常见需求，也有一些可能满足不了，如它不支持 `map[interface{}]interface{}`,`map[any_type]interface{}`,`map[interface{}]any_type` 数据类型；不完全支持自定义类型，目前最稳妥的使用方式就是不使用任何自定义类型，以避免引起程序恐慌（异常）。

[`bxcodec/faker`](https://github.com/bxcodec/faker)  是很优秀的第三方伪造数据生成库，使用简单，功能齐全，测试充分，维护者活跃；能满足日常批量生成测试数据的需求，有效提升开发测试效率，推荐大家使用。

## 参考资料

- [https://github.com/bxcodec/faker](https://github.com/bxcodec/faker)
- [https://pkg.go.dev/github.com/bxcodec/faker/v3#section-documentation](https://pkg.go.dev/github.com/bxcodec/faker/v3#section-documentation)
