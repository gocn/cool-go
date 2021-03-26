# gogo/protobuf的使用

编辑推荐：Bazinga

## 从 JSON 开始

谈到序列化，大家最先想到的可能是 JSON 或者 XML，这两种序列化协议都是基于文本的编码方式进行数据传输。类似的还有 YAML 等。

JSON 拥有许多优点，使之成为最广泛使用的序列化协议之一。如 JSON 协议简单，人眼可读，序列化后十分简洁且解析速度快。此外，JSON 具备 JavaScript 的先天性支持，被广泛应用于 Web Browser 的应用场景中，并且是 Ajax 的事实标准协议。

JSON 的适用场景比较多，典型应用场景包括：

- 公司外部之间传输数据量相对较小，实时性要求相对低的服务
- 基于 Web browser 的 Ajax 请求
- 接口经常发生变化，并对可调式性要求较高的场景，例如移动 App 与服务端的通信

然而，由于 JSON 本身的设计的一些特点，在一些场景下使用 JSON 仍然不是最优解。如：

- 需要标准的 IDL ，增强参与各方业务约束的场景。由于 JSON 协议往往只能使用文档的方式来进行约定，这可能会给调试带来一些不便与不明确
- 对性能和简洁性有较高要求的场景。JSON 在一些语言中的序列化和反序列化需要采用反射机制，所以在性能要求特别高场景下可能不是最优解

- 对于大数据量服务或持久化场景。JSON 进行序列化的额外空间开销比较大，这也意味着较大的内存和磁盘开销

对于以上场景， 使用一些基于 IDL ，存储方案为二进制存储的序列化方案则更为合适， 如 ProtoBuf、Thrift、avro等。

> IDL: 参与通讯的各方需要对通讯的内容需要做相关的约定。为了建立一个与语言和平台无关的约定，这个约定需要采用与具体开发语言、平台无关的语言来进行描述。这种语言被称为接口描述语言（IDL），采用IDL撰写的协议约定称之为IDL文件。

## 什么是 Protobuf

ProtoBuf 是 [Protocol Buffers](https://link.zhihu.com/?target=https%3A//developers.google.com/protocol-buffers/docs/overview) 的简称 ，是 Google 公司开源的一种语言无关、平台无关、可扩展的序列化结构数据的方案，它可用于（数据）通信协议、数据存储等。

ProtoBuf 是上述场景中比较适用的序列化方案之一。 ProtoBuf 非常灵活，高效，我们可以通过定义 IDL （在这里是proto）文件，然后使用生成的源代码轻松的在各种数据流中使用各种语言进行编写和读取结构数据。甚至可以更新数据结构，而不破坏由旧数据结构编译的已部署程序。

上文提到，同类型的序列化方案还有 Thrift 和 Avro。其中 Thrift 并不仅仅是序列化协议，他被嵌入到 Thrift 框架中，这导致其很难和其他传输层协议共同使用。Avro 由于没有成熟的 JS 实现，不适合 Web 环境, 也 导致其使用场景也比较有限。

目前 gRPC 默认的序列化方式是 ProtoBuf。

ProtoBuf 包含序列化格式的定义、各种语言的库以及一个 IDL 编译器。正常情况下需要我们定义 proto 文件，然后使用IDL 编译器编译成需要的语言。

### 一个简单的 proto 例子

```protobuf
syntax = "proto3";                // proto 版本，建议使用 proto3
option go_package = "main/proto"; // 包名声明符

message SearchRequestParam {      // message 类型
  enum Type {                     // 枚举类型
    PC = 0;
    Mobile = 1;
  }
  string query_text = 1;          // 字符串类型 | 后面的「1」为数字标识符，在消息定义中需要唯一
  int32 limit = 3;                // 整型
  Type type = 4;                  // 枚举类型
}

message SearchResultPage {
  repeated string result = 1;     // 「repeated」表示字段可以重复任意多次（包括0次）
  int32 num_results = 2;
}
// test.proto
```

代码中的只是一些比较普通的字段定义，还有一些复杂的一些字段定义，如`Oneof`、`Map`、`Reserved`等可以参考官方文档。

### 生成 Go 代码

在 `.proto` 文件中定义好需要处理的结构化数据后，可以通过 `protoc` 工具，将 `.proto` 文件转换为 C、C++、Golang、Java、Python 等多种语言的代码。我们这里尝试一下生成 Golang 语言代码。

首先需要安装 `protoc` 工具

```shell
# 下载安装包 (Mac)
$ wget https://github.com/protocolbuffers/protobuf/releases/download/v3.15.6/protoc-3.15.6-osx-x86_64.zip
# 解压到 /usr/local 目录下
$ unzip protoc-3.15.6-osx-x86_64.zip -d protoc-3.15.6-osx-x86_64
$ mv protoc-3.5.0-osx-x86_64/bin/protoc /usr/local/bin/protoc
# 执行如下表示成功：
$ protoc --version
libprotoc 3.15.6
```

然后安装一个官方的生成 Golang 代码的插件 `protoc-gen-go`

```bash
$ go get -u github.com/golang/protobuf/protoc-gen-go
```

现在在 `proto`文件所在目录下执行以下命令以生成go文件

```bash
$ protoc --go_out=. test.proto
```

`protoc` 命令还可以使用`-I`参数指定搜索 import 的 proto 的文件夹。其他参数详情可以参考官方文档。

我们可以在目录下看到一个 `test.pb.go` 文件。其中主要结构体如下:

```go
type SearchRequestParam struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	QueryText string                  `protobuf:"bytes,1,opt,name=query_text..."`    
	Limit     int32                   `protobuf:"varint,3,opt,name=limit,proto3"...."`                           
	Type      SearchRequestParam_Type `protobuf:"varint,4,opt,name=type,proto3..."`
}
type SearchResultPage struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Result     []string `protobuf:"bytes,1,rep,name=result,proto3...."`
	NumResults int32    `protobuf:"varint,2,opt,name=num_results,json=numResults,proto3..."`

```

接下来，就可以在项目代码中直接使用了。

## gogo/protobuf 是什么

在上文中，我们安装了一个「生成 Golang 代码的插件 `protoc-gen-go`」，这个插件其实是 golang 官方提供的 一个Protobuf api 实现。而我们的主角`gogo/protobuf`是基于 `golang/protobuf` 的一个增强版实现。

gogo 库基于官方库开发，增加了很多的功能，包括：

- 快速的序列化和反序列化
- 更规范的Go数据结构
- goprotobuf 兼容
- 可选择的产生一些辅助方法，减少使用中的代码输入
- 可以选择产生测试代码和 benchmark 代码
- 其它序列化格式

目前很多知名的项目都在使用该库，如 etcd、k8s、tidb、docker swarmkit 等。

## gogo/protobuf 如何使用

https://github.com/gogo/protobuf  根目录下我们可以看到有很多文件夹，其中「protoc-gen」为前缀的为生成代码的插件，其他「proto」、「protobuf」、「gogoproto」等为库文件。

gogo 库目前有三种生成代码的方式

- `gofast`: 速度优先，但此方式不支持其它 gogoprotobuf 的扩展选项。

```bash
$ go get github.com/gogo/protobuf/protoc-gen-gofast
$ protoc --gofast_out=. myproto.proto
```

- `gogofast`、`gogofaster`、`gogoslick`: 更快的速度、会生成更多的代码。

  - `gogofast`类似`gofast`，但是会引入 gogoprotobuf 库。

  - `gogofaster`类似`gogofast`，但是不会产生`XXX_unrecognized`类的指针字段，可以减少垃圾回收时间。
  - `gogoslick`类似`gogofaster`，但是会增加一些额外的`string`、`gostring`和`equal method`等。

  ```bash
  $ go get github.com/gogo/protobuf/proto
  $ go get github.com/gogo/protobuf/{binary} //protoc-gen-gogofast、protoc-gen-gogofaster 、protoc-gen-gogoslick 
  $ go get github.com/gogo/protobuf/gogoproto
  $ protoc -I=. -I=$GOPATH/src -I=$GOPATH/src/github.com/gogo/protobuf/protobuf --{binary}_out=. myproto.proto // 这里的{binary}不包含「protoc-gen」前缀
  ```

- `protoc-gen-gogo`: 最快的速度，最多的可定制化

  - 可以通过扩展选项高度定制序列化。

  ```bash
  $ go get github.com/gogo/protobuf/proto
  $ go get github.com/gogo/protobuf/jsonpb
  $ go get github.com/gogo/protobuf/protoc-gen-gogo
  $ go get github.com/gogo/protobuf/gogoproto
  ```

gogo/protobuf 提供了非常多的扩展选项，以便在产生代码的时候进行更多的控制。上文提到的扩展选项这里有一个全面的介绍：[extensions](https://github.com/gogo/protobuf/blob/master/extensions.md)，扩展选项里主要包含一些生成快速序列化反序列化代码的可选项、生成更规范的Golang 数据结构的可选项、goprotobuf 兼容的可选项，一些产生辅助方法的可选项、产生测试代码和benchmark 的可选项，还可以增加 jsontag 等。

有同学对以上多个生成方式的序列化性能做了一些压测，在一般需求下，性能差距并不是很大，`protoc-gen-gofast`方式基本可以满足大多数场景。

最后，生成的 go 语言代码在项目中使用就非常简单了，一般只需要使用`proto.Marshal`,`proto.Unmarshal` 方法就可以了，下面是一个例子：

```go
package main

import (
	"fmt"
	"log"

	zaproto "git.xxxxx.com/data/za-proto/proto"
	"github.com/gogo/protobuf/proto"
)

func main() {
	req := &zaproto.SearchRequestParam{
		QueryText: "xxxxxx",
		Limit:     10,
		Type:      zaproto.SearchRequestParam_PC,
	}
	data, err := proto.Marshal(req)
	if err != nil {
		log.Fatal("Marshal err : err")
	}
	// send data
	fmt.Println(string(data))

	var respData []byte
	var result = zaproto.SearchResultPage{}
	if err = proto.Unmarshal(respData, &result); err == nil {
		fmt.Println(result)
	} else {
		log.Fatal("Unmarshal err : err")

	}
}
```



## 参考

[alecthomas/go_serialization_benchmarks: Benchmarks of Go serialization methods (github.com)](https://github.com/alecthomas/go_serialization_benchmarks)

[So you want to use GoGo Protobuf (jbrandhorst.com)](https://jbrandhorst.com/post/gogoproto/)

[Schema evolution in Avro, Protocol Buffers and Thrift — Martin Kleppmann’s blog](https://martin.kleppmann.com/2012/12/05/schema-evolution-in-avro-protocol-buffers-thrift.html)

[Language Guide  | Protocol Buffers  | Google Developers](https://developers.google.com/protocol-buffers/docs/proto)

[序列化和反序列化 - 美团技术团队 (meituan.com)](https://tech.meituan.com/2015/02/26/serialization-vs-deserialization.html)

[Protobuf 有没有比 JSON 快 5 倍？-InfoQ](https://www.infoq.cn/article/json-is-5-times-faster-than-protobuf)

[几种Go序列化库的性能比较 | 鸟窝 (colobu.com)](https://colobu.com/2015/09/28/Golang-Serializer-Benchmark-Comparison/)

[思考gRPC ：为什么是protobuf | 横云断岭的专栏 (hengyunabc.github.io)](http://hengyunabc.github.io/thinking-about-grpc-protobuf/)

[几种Go序列化库的性能比较 | 鸟窝 (colobu.com)](https://colobu.com/2015/09/28/Golang-Serializer-Benchmark-Comparison/)

