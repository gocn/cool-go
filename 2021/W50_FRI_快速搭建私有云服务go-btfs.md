# 快速搭建私有云服务 go-btfs

## 1. go-btfs 是什么？
[go-btfs](https://github.com/tron-us/go-btfs) go-btfs是一个去中心化的文件存储平台，无论图片、文件、视频等等各类文件。
每个人都可以在自己电脑上安装部署BTFS节点，然后大家的节点相互连接，构成一个整体网络，
网络中某一个节点上传文件，其他节点就像访问本地一样直接下载使用。同时，它还支持冗余备份，使用reed-solomn方案，这块更加牛逼。

## 2. 怎么使用

2.1 下载 `github.com/tron-us/go-btfs`，并编译

由于依赖第三方库较多，需要download一下
然后进入指定目录，进行编译

```shell
$ git clone git@github.com:TRON-US/go-btfs.git

$ cd go-btfs

$ git checkout release

$ go mod download

$ cd cmd/btfs

$ go build

``` 

此处生成btfs可执行文件，可以导入到 /usr/local/bin 下面.

2.2 启动本地BTFS节点

```shell
# 设置BTFS节点存储地址空间，换一个就可以起一个新节点
$export BTFS_PATH=/Users/laocheng.cheng/.btfs.ll

$ btfs init 
Generating TRON key with BIP39 seed phrase...
Master public key:  xpub661MyMwAqRbcFYYeCS183yzjqyHjDAYMAdJ6oQPZNqwu3CyH6SgQ5FgvYYNWQA2v8gZWkZJ25Lr4gKuGHf21izyQ5s7aKjMuHGPRJ7AeGpq
initializing BTFS node at /Users/laocheng.cheng/.btfs.lll
generating btfs node keypair with TRON key...done
peer identity: 16Uiu2HAmCQadnAGfADbwi9DmdjZcHPzFNR3r72hfnMPrCEQKjN2k
to get started, enter:

	btfs cat /btfs/QmZjrLVdUpqVU6Pnc8pBnyQxVdpn9J8tfcsycP84W6N93C/readme

$ btfs daemon
Initializing daemon...
go-btfs version: 1.5.3-17053fc
Repo version: 10
System version: amd64/darwin
Golang version: go1.15.15
Repo location: /Users/laocheng.cheng/.btfs.lll
Peer identity: 16Uiu2HAmCQadnAGfADbwi9DmdjZcHPzFNR3r72hfnMPrCEQKjN2k
Swarm listening on /ip4/127.0.0.1/tcp/4001
Swarm listening on /ip4/192.168.106.19/tcp/4001
Swarm listening on /ip4/2.0.1.44/tcp/4001
Swarm listening on /ip6/::1/tcp/4001
Swarm listening on /p2p-circuit
Swarm announcing /ip4/127.0.0.1/tcp/4001
Swarm announcing /ip4/192.168.106.19/tcp/4001
Swarm announcing /ip4/2.0.1.44/tcp/4001
Swarm announcing /ip4/203.12.203.2/tcp/4001
Swarm announcing /ip4/219.143.35.171/tcp/4001
Swarm announcing /ip6/::1/tcp/4001
API server listening on /ip4/127.0.0.1/tcp/5001
WebUI: http://127.0.0.1:5001/webui
HostUI: http://127.0.0.1:5001/hostui
Gateway (readonly) server listening on /ip4/127.0.0.1/tcp/8080
Remote API server listening on /ip4/127.0.0.1/tcp/5101
Daemon is ready

# 配置为host存储模式
$ btfs config profile apply storage-host

```

当`btfs daemon`启动后，
`/Users/laocheng.cheng/.btfs.lll` 这个是我们的地址空间，
`16Uiu2HAmCQadnAGfADbwi9DmdjZcHPzFNR3r72hfnMPrCEQKjN2k` 这个是节点ID，其他节点就是通过节点ID和其沟通。

2.3 再次启动一个新节点

新起一个终端，过程同上
```shell
# 设置BTFS节点存储地址空间
$export BTFS_PATH=/Users/laocheng.cheng/.btfs.ggg

$ btfs init 
......

$ btfs daemon

# 配置为host存储模式
$ btfs config profile apply storage-host
......

```

2.4 组建本地私有网络

特别注意的问题，现在启动的节点，是和BTFS真实网络连接，我们需要一些修改，变成本地网络方式。
此时，选中一个终端，即一个节点；先bootstrap设空，然后把自己创建的节点，全部加入（当然，不用加该终端节点）。
```shell
btfs config --json  Bootstrap "[]"
btfs bootstrap add /ip4/127.0.0.1/tcp/54001/p2p/16Uiu2HAmCQadnAGfADbwi9DmdjZcHPzFNR3r72hfnMPrCEQKjN2k
btfs bootstrap add /ip4/127.0.0.1/tcp/54001/p2p/16Uiu2HAm3GdbCk6Uwst2t3zoYTrgHbifjqJTpLbHHUUPBcBT8oqC
btfs bootstrap add /ip4/127.0.0.1/tcp/54001/p2p/16Uiu2HAmCQadnAGfADbwi9DmdjZcHPzFNR3r72hfnMPrCEQKjN2k
```

此时，我们自己的节点，构成了一个私有云存储网络。


2.5 上传文件 及 任意节点可查看

打开一个节点（终端），上传文件如下

```shell
$ btfs add s
added Qmefmseqwa8un9WXEfqb2GY2ncWmmB2BsAqtcjVJaHahL3 s
 31 B / 31 B [================================================================] 100.00%
```

打开另外一个节点（终端），下载文件到本地

```shell
$ btfs get QmduujE1EgUajwCj2bxjdp4LWz62aameQJmQc7pcBbeAmC
Saving file(s) to QmduujE1EgUajwCj2bxjdp4LWz62aameQJmQc7pcBbeAmC
 27 B / 27 B [================================================================] 100.00% 0s
```

如此一来，私有云的上传下载就搞定了。

3.具体应用

上面我们就私有云的搭建，及上传、下载操作搞定了。
那么对应的应用方案就容易理解，比如你开发一个存储网站。后端数据用咱们的私有云，上传一个key（文件名），对应一个value（文件hash）。
然后我们把key：value记录到mysql or redis。一个存储类网站的基础功能就完成啦。


## 总结

[go-btfs](https://github.com/tron-us/go-btfs) go-btfs是一个去中心化的文件存储平台，各种类型文件都能上传，并且安装方便，经过基本配置，就可以快速搭建自己的私有云服务。
非常推荐大家使用，尤其企业内部云平台搭建使用。


以上所有内容均采用最新官方案例做示例
## 参考资料

- [查看go-btfs源代码](https://github.com/tron-us/go-btfs)
- [go-btfs官方文档参考](https://docs.btfs.io/docs)
