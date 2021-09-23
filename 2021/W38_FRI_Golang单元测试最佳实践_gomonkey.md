# golang 单元测试最佳实践

## 为什么要进行单元测试？

在没工作之前，说实话没怎么写过单元测试，很多情况下就是一边写代码，一边运行，用 fmt.Println() 打印变量，再稍微复杂一点的也许会用 dlv 去 debug 代码，找出问题。

但是在做公司做大型项目时就会发现，你根本就没办法把项目跑起来，这个时候你只能通过写单元测试去看自己的逻辑对不对。

当然仍旧会出现一个问题，当你的功能中又调用了其他的接口，但这个接口在你当前的环境中是没办法正常调用的，比如数据库连接，文件 I/O。网络I/O 等。这个时候就需要一个 好用的 mock 库了，简单来说就是用 mock 对象模拟依赖项的行为，这里我推荐使用 gomonkey。

从另一个角度讲，为什么需要单元测试呢，因为我们一般项目都有覆盖率的要求，写单测当然是也是为了提高代码覆盖率咯，不然代码都无法提交到 gitlab 上。

## gomonkey 入门

### 安装 gomonkey

```shell
go get github.com/agiledragon/gomonkey
```

### gomonkey 常见用法

* mock 一个函数
* mock 一个成员方法

* 其他用法可参考官方文档

业务代码如下：

```go
package mock

import (
	"encoding/json"
	"io/ioutil"
	"net/http"
)

type Request struct {
	Url string
}

type Response struct {
	Result string
}

func exec(args []byte) (res *Response, err error) {
	var w Request
	if err = json.Unmarshal(args, &w); err != nil {
		return nil, err
	}
	if res, err = w.DoAction(w.Url); err != nil {
		return nil, err
	}
	return
}

func (r *Request) DoAction(action string) (resp *Response, err error) {
	var (
		res *http.Response
		b   []byte
	)
	if res, err = http.Get(action); err != nil {
		return nil, err
	}
	if b, err = ioutil.ReadAll(res.Body); err != nil {
		return nil, err
	}
	return &Response{Result: string(b)}, nil
}
```

test case 如下：

```go
package mock

import (
	"encoding/json"
	"reflect"
	"testing"

	"github.com/agiledragon/gomonkey"
	"github.com/stretchr/testify/assert"
)

func TestExec(t *testing.T) {
	var test = []struct {
		in   []byte
		want *Response
	}{
		{
			in:   []byte("https://gocn.vip/api/v1/count"),
			want: &Response{Result: "666"},
		},
	}
	var r Request
	f := func(t *testing.T) *gomonkey.Patches {
		patches := gomonkey.NewPatches()
		// mock json.Unmarshal()
		patches.ApplyFunc(json.Unmarshal, func(b []byte, d interface{}) error {
			data := d.(*Request)
      // 替换成任何你想要的数据
			(*data).Url = "127.0.0.1/api/v1/count"
			return nil
		})
		// mock 成员方法，注意，成员方法首字母要大写！
		patches.ApplyMethod(reflect.TypeOf(&r), "DoAction", func(_ *Request, _ string) (*Response, error) {
			return &Response{Result: "666"}, nil
		})
		return patches
	}
	t.Run("test", func(t *testing.T) {
		patches := f(t)
		defer patches.Reset()
		for _, v := range test {
			r, err := exec(v.in)
			if !assert.NotNil(t, r) {
				t.Log(err)
				continue
			}
			assert.Equal(t, "666", r.Result)
		}
	})
}
```

```shell
$ go test -gcflags=-l -v -run ./
=== RUN   TestExec
=== RUN   TestExec/test
--- PASS: TestExec (0.00s)
    --- PASS: TestExec/test (0.00s)
PASS
ok      mytest/add      0.018s
```

当然有时候也为了增加代码覆盖率，需要将`if err != nil` 等覆盖，可以写多个 f, 如 

`f1 := func(t *testing.T) *gomonkey.Patches {}`

`f2 := func(t *testing.T) *gomonkey.Patches {}`

最后 `t.Run("test1",xxx) `, `t.Run("test2", xxx)`即可

## 参考资料

* https://github.com/agiledragon/gomonkey/
* https://github.com/stretchr/testify









