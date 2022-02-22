# 一、gopacket简介

## 1、gopacket是什么？

gopacket是google出品的golang三方库，质量还是靠的住，项目地址为：github.com/google/gopacket

gopacket到底是什么呢？是个抓取网络数据包的库，这么说可能还有点抽象，但是抓包工具大家可能都使用过。

Windows平台下有Wireshark抓包工具，其底层抓包库是npcap（以前是winpcap）；

Linux平台下有Tcpdump，其抓包库是libpcap；

而gopacket库可以说是libpcap和npcap的go封装，提供了更方便的go语言操作接口。



对于抓包库来说，常规功能就是抓包，而网络抓包有以下几个步骤：

1、枚举主机上网络设备的接口

2、针对某一网口进行抓包

3、解析数据包的mac层、ip层、tcp/udp层字段等

4、ip分片重组，或tcp分段重组成上层协议如http协议的数据

5、对上层协议进行头部解析和负载部分解析

## 2、应用场景有哪些？

场景1：网络流量分析

对网络设备流量进行实时采集以及数据包分析。

场景2：伪造数据包

不少网络安全工具，需要伪造网络数据包，填充上必要的协议字段后发送给对端设备，从而达到一些目的。

场景3：离线pcap文件的读取和写入



# 二、安装部署

## 2、1 安装libpcap或npcap三方库

在使用gopacket包时，首先要确保在windows平台下安装了npcap或winpcap，或者是在linux平台下安装了libpcap库。

npcap下载地址：https://nmap.org/npcap/

libpcap下载地址:https://www.tcpdump.org/

下载自己电脑对应的操作系统版本的库

如果不想从官网下载libpcap库的话，也可以采用centos的yum命令或ubuntu的apt get命令来进行安装。

## 2、2 安装gopacket库

go get github.com/google/gopacket



# 三、使用方法

## 3、1 枚举网络设备

```go
package main
import (
    "fmt"
    "log"
    "github.com/google/gopacket/pcap"
)
func main() {
    // 得到所有的(网络)设备
    devices, err := pcap.FindAllDevs()
    if err != nil {
        log.Fatal(err)
    }
    // 打印设备信息
    fmt.Println("Devices found:")
    for _, device := range devices {
        fmt.Println("\nName: ", device.Name)
        fmt.Println("Description: ", device.Description)
        fmt.Println("Devices addresses: ", device.Description)
        for _, address := range device.Addresses {
            fmt.Println("- IP address: ", address.IP)
            fmt.Println("- Subnet mask: ", address.Netmask)
        }
    }
}
```

先调用pcap.FindAllDevs()获取当前主机所有的网络设备，网络设备有哪些属性呢？

```go
// Interface describes a single network interface on a machine.
type Interface struct {
	Name        string //设备名称
	Description string //设备描述信息
	Flags       uint32 
	Addresses   []InterfaceAddress //网口的地址信息列表
}
// InterfaceAddress describes an address associated with an Interface.
// Currently, it's IPv4/6 specific.
type InterfaceAddress struct {
	IP        net.IP
	Netmask   net.IPMask // Netmask may be nil if we were unable to retrieve it.
	Broadaddr net.IP     // Broadcast address for this IP may be nil
	P2P       net.IP     // P2P destination address for this IP may be nil
}
```



## 3、2 打开一个设备进行抓包

```go
package main
import (
    "fmt"
    "github.com/google/gopacket"
    "github.com/google/gopacket/pcap"
    "log"
    "time"
)
var (
    device       string = "eth0"
    snapshot_len int32  = 1024
    promiscuous  bool   = false
    err          error
    timeout      time.Duration = 30 * time.Second
    handle       *pcap.Handle
)
func main() {
    // 打开某一网络设备
    handle, err = pcap.OpenLive(device, snapshot_len, promiscuous, timeout)
    if err != nil {log.Fatal(err) }
    defer handle.Close()
    // Use the handle as a packet source to process all packets
    packetSource := gopacket.NewPacketSource(handle, handle.LinkType())
    for packet := range packetSource.Packets() {
        // Process packet here
        fmt.Println(packet)
    }
}
```

### 1） 实时捕获

2、1 节中我们枚举了当前主机的所有网络设备，现在需要打开网络设备并进行实时捕获数据包，需调用pcap.OpenLive来打开网络设备，其函数原型如下：

```go
func OpenLive(device string, snaplen int32, promisc bool, timeout time.Duration) (handle *Handle, _ error)
```

device:网络设备的名称，如eth0,也可以填充pcap.FindAllDevs()返回的设备的Name

snaplen: 每个数据包读取的最大长度 the maximum size to read for each packet

promisc:是否将网口设置为混杂模式,即是否接收目的地址不为本机的包

timeout:设置抓到包返回的超时。如果设置成30s，那么每30s才会刷新一次数据包；设置成负数，会立刻刷新数据包，即不做等待



函数返回值：是一个*Handle类型的返回值，可能作为gopacket其他函数调用时作为函数参数来传递。

注意事项：

一定要记得释放掉handle，如文中的defer handle.Close()。

### 2） 创建数据包源

packetSource := gopacket.NewPacketSource(handle, handle.LinkType())

第一个参数为OpenLive的返回值，指向Handle类型的指针变量handle。

第二个参数为handle.LinkType()此参数默认是以太网链路，一般我们抓包，也是从2层以太网链路上抓取。



### 3）读取数据包

```go
//packetSource.Packets()是个channel类型，此处是从channel类型的数据通道中持续的读取网络数据包
for packet := range packetSource.Packets() {
		// Process packet here
		fmt.Println(packet)
	}
```

## 3、3 解码数据包的各层

我们可以获取原始数据包，并尝试将其强制转换为已知格式。如ethernet、IP和TCP层。

Layers包是gopacket的Go库中的新功能，在底层libpcap库中不存在。它是gopacket库的非常有用的一部分。它允许我们轻松地识别数据包是否包含特定类型的层。这个代码示例将演示如何使用layers包来查看包是否是ethernet、IP和TCP，以及如何轻松访问这些头中的字段。

```go
package main
import (
    "fmt"
    "github.com/google/gopacket"
    "github.com/google/gopacket/layers"
    "github.com/google/gopacket/pcap"
    "log"
    "strings"
    "time"
)
var (
    device      string = "eth0"
    snapshotLen int32  = 1024
    promiscuous bool   = false
    err         error
    timeout     time.Duration = 30 * time.Second
    handle      *pcap.Handle
)
func main() {
    // Open device
    handle, err = pcap.OpenLive(device, snapshotLen, promiscuous, timeout)
    if err != nil {log.Fatal(err) }
    defer handle.Close()
    packetSource := gopacket.NewPacketSource(handle, handle.LinkType())
    for packet := range packetSource.Packets() {
        printPacketInfo(packet)
    }
}
func printPacketInfo(packet gopacket.Packet) {
    // Let's see if the packet is an ethernet packet
    // 判断数据包是否为以太网数据包，可解析出源mac地址、目的mac地址、以太网类型（如ip类型）等
    ethernetLayer := packet.Layer(layers.LayerTypeEthernet)
    if ethernetLayer != nil {
        fmt.Println("Ethernet layer detected.")
        ethernetPacket, _ := ethernetLayer.(*layers.Ethernet)
        fmt.Println("Source MAC: ", ethernetPacket.SrcMAC)
        fmt.Println("Destination MAC: ", ethernetPacket.DstMAC)
        // Ethernet type is typically IPv4 but could be ARP or other
        fmt.Println("Ethernet type: ", ethernetPacket.EthernetType)
        fmt.Println()
    }
    // Let's see if the packet is IP (even though the ether type told us)
    // 判断数据包是否为IP数据包，可解析出源ip、目的ip、协议号等
    ipLayer := packet.Layer(layers.LayerTypeIPv4)
    if ipLayer != nil {
        fmt.Println("IPv4 layer detected.")
        ip, _ := ipLayer.(*layers.IPv4)
        // IP layer variables:
        // Version (Either 4 or 6)
        // IHL (IP Header Length in 32-bit words)
        // TOS, Length, Id, Flags, FragOffset, TTL, Protocol (TCP?),
        // Checksum, SrcIP, DstIP
        fmt.Printf("From %s to %s\n", ip.SrcIP, ip.DstIP)
        fmt.Println("Protocol: ", ip.Protocol)
        fmt.Println()
    }
    // Let's see if the packet is TCP
    // 判断数据包是否为TCP数据包，可解析源端口、目的端口、seq序列号、tcp标志位等
    tcpLayer := packet.Layer(layers.LayerTypeTCP)
    if tcpLayer != nil {
        fmt.Println("TCP layer detected.")
        tcp, _ := tcpLayer.(*layers.TCP)
        // TCP layer variables:
        // SrcPort, DstPort, Seq, Ack, DataOffset, Window, Checksum, Urgent
        // Bool flags: FIN, SYN, RST, PSH, ACK, URG, ECE, CWR, NS
        fmt.Printf("From port %d to %d\n", tcp.SrcPort, tcp.DstPort)
        fmt.Println("Sequence number: ", tcp.Seq)
        fmt.Println()
    }
    // Iterate over all layers, printing out each layer type
    fmt.Println("All packet layers:")
    for _, layer := range packet.Layers() {
        fmt.Println("- ", layer.LayerType())
    }
    ///.......................................................
    // Check for errors
    // 判断layer是否存在错误
    if err := packet.ErrorLayer(); err != nil {
        fmt.Println("Error decoding some part of the packet:", err)
    }
}
```

仅仅以此处tcp部分的代码详细解析下

```go
// 判断数据包是否为TCP数据包，可解析源端口、目的端口、seq序列号、tcp标志位等
    tcpLayer := packet.Layer(layers.LayerTypeTCP)
    if tcpLayer != nil {
        fmt.Println("TCP layer detected.")
        tcp, _ := tcpLayer.(*layers.TCP)
        fmt.Printf("From port %d to %d\n", tcp.SrcPort, tcp.DstPort)
    }
```

此处需要研究下源代码中数据结构，以防理解错误

```go
type Packet interface {
	// Layer returns the first layer in this packet of the given type, or nil
	Layer(LayerType) Layer   //根据给定的类型，在数据包中寻找其第一个层
}
//看看Layer的结构
type Layer interface {
	// LayerType is the gopacket type for this layer.
	LayerType() LayerType
	// LayerContents returns the set of bytes that make up this layer.
	LayerContents() []byte
	// LayerPayload returns the set of bytes contained within this layer, not
	// including the layer itself.
	LayerPayload() []byte
}
//tcp数据包格式
type TCP struct {
	BaseLayer
	SrcPort, DstPort                           TCPPort
	Seq                                        uint32
	Ack                                        uint32
	DataOffset                                 uint8
	FIN, SYN, RST, PSH, ACK, URG, ECE, CWR, NS bool
	Window                                     uint16
	Checksum                                   uint16
	Urgent                                     uint16
	sPort, dPort                               []byte
	Options                                    []TCPOption
	Padding                                    []byte
	opts                                       [4]TCPOption
	tcpipchecksum
}
```

TCP结构体是实现了Layer接口的，其实Ethernet，IPV4，UDP等结构体也实现了Layer接口

在上述代码中，我们调用函数时，传入的LayerType协议层的类型为layers.LayerTypeTCP，函数返回值为interface类型，必须转换成TCP结构体

tcp, _ := tcpLayer.(*layers.TCP)

tcp是layers.TCP这个具体类型的指针，通过tcp则可以获取数据包中tcp协议的相关字段。

## 3、4 自定义层

自定义层有助于实现当前不包含在gopacket layers包中的协议。

```
import (
    "fmt"
    "github.com/google/gopacket"
)
// 创建自定义层数据结构，并实现Layer接口中的函数LayerType()、LayerContents()、LayerPayload()
type CustomLayer struct {
    // This layer just has two bytes at the front
    SomeByte    byte
    AnotherByte byte
    restOfData  []byte
}
// 注册自定义层类型，然后我们才可以使用它
// 第一个参数是ID. 自定义层使用大于2000的数字，它必须是唯一的
var CustomLayerType = gopacket.RegisterLayerType(
    2001,
    gopacket.LayerTypeMetadata{
        "CustomLayerType",
        gopacket.DecodeFunc(decodeCustomLayer),
    },
)

//自定义层实现LayerType
func (l CustomLayer) LayerType() gopacket.LayerType {
    return CustomLayerType
}

//自定义层实现LayerContents
func (l CustomLayer) LayerContents() []byte {
    return []byte{l.SomeByte, l.AnotherByte}
}

//自定义层实现LayerPayload
func (l CustomLayer) LayerPayload() []byte {
    return l.restOfData
}

//实现自定义的解码函数
func decodeCustomLayer(data []byte, p gopacket.PacketBuilder) error {
    p.AddLayer(&CustomLayer{data[0], data[1], data[2:]})
    return p.NextDecoder(gopacket.LayerTypePayload)
}
func main() {
    rawBytes := []byte{0xF0, 0x0F, 65, 65, 66, 67, 68}
    packet := gopacket.NewPacket(
        rawBytes,
        CustomLayerType,
        gopacket.Default,
    )
    fmt.Println("Created packet out of raw bytes.")
    fmt.Println(packet)
    // Decode the packet as our custom layer
    customLayer := packet.Layer(CustomLayerType)
    if customLayer != nil {
        fmt.Println("Packet was successfully decoded with custom layer decoder.")
        customLayerContent, _ := customLayer.(*CustomLayer)
        // Now we can access the elements of the custom struct
        fmt.Println("Payload: ", customLayerContent.LayerPayload())
        fmt.Println("SomeByte element:", customLayerContent.SomeByte)
        fmt.Println("AnotherByte element:", customLayerContent.AnotherByte)
    }
}
```

结合上述代码可知，实现自定义的层需要3步：

1、创建自定义层的结构体，并实现Layer接口中的函数LayerType()、LayerContents()、LayerPayload()

2、按照解码函数签名来实现自定义解码函数，名称可自行命名。

解码函数签名如下：

type DecodeFunc func([]byte, PacketBuilder) error

3、使用gopacket.RegisterLayerType函数来注册自定义层

## 3、5 TCP流重组

为什么需要tcp流重组？

```go
package main

import (
	"bufio"
	"flag"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/google/gopacket"
	"github.com/google/gopacket/examples/util"
	"github.com/google/gopacket/layers"
	"github.com/google/gopacket/pcap"
	"github.com/google/gopacket/tcpassembly"
	"github.com/google/gopacket/tcpassembly/tcpreader"
)

var iface = flag.String("i", "eth0", "Interface to get packets from")
var snaplen = flag.Int("s", 1600, "SnapLen for pcap packet capture")

// Build a simple HTTP request parser using tcpassembly.StreamFactory and tcpassembly.Stream interfaces

// httpStreamFactory implements tcpassembly.StreamFactory
type httpStreamFactory struct{}

// httpStream will handle the actual decoding of http requests.
type httpStream struct {
	net, transport gopacket.Flow
	r              tcpreader.ReaderStream
}

func (h *httpStreamFactory) New(net, transport gopacket.Flow) tcpassembly.Stream {
	hstream := &httpStream{
		net:       net,
		transport: transport,
		r:         tcpreader.NewReaderStream(),
	}
	go hstream.run() // Important... we must guarantee that data from the reader stream is read.

	// ReaderStream implements tcpassembly.Stream, so we can return a pointer to it.
	return &hstream.r
}

func (h *httpStream) run() {
	buf := bufio.NewReader(&h.r)
	for {
		req, err := http.ReadRequest(buf)
		if err == io.EOF {
			// We must read until we see an EOF... very important!
			return
		} else if err != nil {
			log.Println("Error reading stream", h.net, h.transport, ":", err)
		} else {
			bodyBytes := tcpreader.DiscardBytesToEOF(req.Body)
			req.Body.Close()
			log.Println("Received request from stream", h.net, h.transport, ":", req, "with", bodyBytes, "bytes in request body")
		}
	}
}

func main() {
	defer util.Run()()
	var handle *pcap.Handle
	var err error

	// Set up pcap packet capture
	handle, err = pcap.OpenLive(*iface, int32(*snaplen), true, pcap.BlockForever)
	if err != nil {
		log.Fatal(err)
	}

	// Set up assembly
	streamFactory := &httpStreamFactory{}
	streamPool := tcpassembly.NewStreamPool(streamFactory)
	assembler := tcpassembly.NewAssembler(streamPool)

	// Read in packets, pass to assembler.
	packetSource := gopacket.NewPacketSource(handle, handle.LinkType())
	packets := packetSource.Packets()
	ticker := time.Tick(time.Minute)
	for {
		select {
		case packet := <-packets:
			if packet.NetworkLayer() == nil || packet.TransportLayer() == nil || packet.TransportLayer().LayerType() != layers.LayerTypeTCP {
				log.Println("Unusable packet")
				continue
			}
			tcp := packet.TransportLayer().(*layers.TCP)
			//将数据包进行重组
			assembler.AssembleWithTimestamp(packet.NetworkLayer().NetworkFlow(), tcp, packet.Metadata().Timestamp)

		case <-ticker:
			//每隔一分钟，刷新之前两分钟内不活动的连接
			assembler.FlushOlderThan(time.Now().Add(time.Minute * -2))
		}
	}
}
```

基本步骤如下：

1、创建httpStreamFactory结构体，实现tcpassembly.StreamFactory接口

2、创建连接池

 streamPool := tcpassembly.NewStreamPool(streamFactory)

3、创建重组器

assembler := tcpassembly.NewAssembler(streamPool)

4、将数据包添加到重组器中

assembler.AssembleWithTimestamp(packet.NetworkLayer().NetworkFlow(), tcp, packet.Metadata().Timestamp)



# 三、总结

首先，gopacket库是google大厂背书，从使用文档、质量、社区活跃度来说都很不错

其次，使用方式简单，扩展性好。gopacket提供了自定义的接口，可根据自身需要进行定制化开发

最后，gopacket定义的layers齐全，如果是实时捕获数据后进行协议解析，采用其内置的layer即可，无需自己手动去解析繁杂的协议了。

