* # [爱奇艺、B站、腾讯等视频下载] 摸鱼之道高一尺魔高一丈Lux

  ### 我不是摸鱼，我只是想学习更多的知识（手动狗头）

  我们做程序员的，如果不努力学习就会被社会的洪流卷得体无完肤。但是公司偏偏不给这个机会，优酷、爱奇艺、bilibili等你知道或不知道的视频网站均被**公司防火墙拦截**了，所谓道高一尺，魔高一丈，今儿我就要破了这个门儿，和老板斗，其乐无穷。

  ## 前提工作（以下所有内容均以windows为例，linux或者macos下载对应的软件即可）

  * 准备一台公网的服务器，此服务器可以访问外网（我是找运维申请的，理由是说做测试机），开放22端口，也就是sshd服务
  * 下载[Lux](https://github.com/iawia002/lux)，Lux的前身就是annie，现在annie改名了。在release中下载对应系统的执行文件
  * 下载[ffmpeg](https://www.gyan.dev/ffmpeg/builds/)在 release builds下载执行文件
  * 编写能够通过ssh隧道的代理。

  ## 首先写ssh隧道连接

  ##### 1、第一个是无代码版，但是每次启动都要手动输入账号密码。打开cmd命令行，输入下面命令：（基础知识ssh-tunnel）

  ```shell
  $ ssh -D 127.0.0.1:本机要代理的端口 你服务器的账号@你服务器的地址
  # 比如输入以下指令：ssh -D 127.0.0.1:1080 root@118.195.141.206
  # 之后就会让你输入密码，密码输入成功就会登录到你的服务器，不要关闭这个cmd窗口，要保持ssh隧道一直存在
  ```

  ##### 2、第二个是代码版，生成好执行文件，每次只需要双击启动即可：

  ```go
  package main
  
  import (
  	"context"
  	"fmt"
  	"log"
  	"net"
  	"os"
  	"os/signal"
  	"syscall"
  
  	"github.com/armon/go-socks5"
  	"golang.org/x/crypto/ssh"
  )
  
  var (
  	username         = "xxxxxxxxxxxxx"      // 你服务器登录密码
  	password         = "xxxxxxxxxxxxx"      // 你服务器登录账号
  	serverAddrString = "xxxxxxxxxxxxx"      // 你的服务器连接地址 如：118.195.141.206
  	serverPortString = ":22"                // 默认ssh端口号，不要改
  	localAddrString  = "127.0.0.1:1080"     // 本地需要代理的端口，若要运行本案例，不要改
  )
  
  func main() {
  	sshConf := &ssh.ClientConfig{
  		User: username,
  		Auth: []ssh.AuthMethod{ssh.Password(password)},
  		HostKeyCallback: func(hostname string, remote net.Addr, key ssh.PublicKey) error {
  			return nil
  		},
  	}
  	sshConn, err := ssh.Dial("tcp", serverAddrString+serverPortString, sshConf)
  	if err != nil {
  		fmt.Println("error tunnel to server: ", err)
  		return
  	}
  	defer sshConn.Close()
  	log.Println("链接ssh成功")
  	go func() {
  		conf := &socks5.Config{
  			Dial: func(ctx context.Context, network, addr string) (net.Conn, error) {
  				return sshConn.Dial(network, addr)
  			},
  		}
  		serverSocks, err := socks5.New(conf)
  		if err != nil {
  			fmt.Println(err)
  			return
  		}
  		if err := serverSocks.ListenAndServe("tcp", localAddrString); err != nil {
  			fmt.Println("本地创建socks5 服务失败", err)
  		}
  	}()
  	// 用于手动退出
  	ch := make(chan os.Signal)
  	signal.Notify(ch, syscall.SIGINT, syscall.SIGTERM)
  	<-ch
  	return
  }
  ```

  启动成功后会出现：

  ```shell
  2022/02/22 10:36:25 链接ssh成功
  ```

  至此，您已经成功通过本地利用ssh通道连接到了你的服务器，并在本地开启了127.0.0.1:1080端口，通过socks5就能实现本地流量从127.0.0.1:1080端口到通道请求到外网。

  ### 给浏览器添加socks5代理：

  ###### 以chrome内核为例，微软的edge同样适用，火狐也可以

  1. 下载SwitchyOmega，并给浏览器安装此插件
  2. 配置SwitchyOmega，在代理服务器下，选择代理协议：SOCKS5，代理服务器就填写本地127.0.0.1，代理端口就是1080。然后点击应用选项。并在插件栏应用此代理配置。

  ![设置sock5代理](https://cdn.gocn.vip/forum-user-images/20220307/f8d38552db5645e8aa62149a3601f44a.jpg)

  ##### 这时候通过当前浏览器就可以访问视频网站了

  #### 你以为这次教学就到此为止了吗？正片才刚刚开始！

  ## 下载学习视频资料

  做为程序员，温故而知新是我们学习的重要指标之一，那么把B站的学习资料下载下来，当忘记的时候可以再看一次，这是程序员的必修课。

  接下里，<u>**Lux**</u>终于要登场了。

  方才我们已经下载了Lux，并将他放到了咱们的公共环境变量Path中（配置环境变量是程序员的基础），由于Lux下载下来的是流媒体片段，我们还需要ffmpeg来对片段进行合成，所以也需要把ffmpeg放到公共环境变量中。

  此时如果网络没有问题，打开B站的某个视频，就可以通过命令行进行下载了：

  ```shell
  lux -C https://www.bilibili.com/video/BV1D341177ER?spm_id_from=333.999.0.0
  ```

  下载下来后是流媒体分片，但是因为我们有ffmpeg（**记得设置为环境变量**），自动会合成Mp4文件。

  但是。。。公司不是不让访问视频网站吗，此时我们再请到linux下的选手proxychains为我们代劳。

  Windows下也可以使用proxychains了，下载[proxychains](github.com/shunf4/proxychains-windows)，我选择的是proxychains_0.6.8_win32_x64.zip，也将其加入到公共环境变量中。

  ```shell
  # 我是把proxychains_0.6.8_win32_x64中的文件proxychains_win32_x64.exe改名为了proxychains.exe，方便命令行打入
  
  # 这时，一定要把配置以下，否则会找不到配置文件：
  # 1、在文件管理器中输入 %USERPROFILE% 会跳转到当前登录用户所在的目录下（如我的是：C:\Users\anyanfei）
  # 2、在当前目录下新建文件夹，名字为：.proxychains   （不要忘了前面的英文小数点）
  # 3、把proxychains_0.6.8_win32_x64文件里的proxychains.conf文件复制到刚才创建的文件夹下
  # 4、更改代理配置，找到proxychains.conf最后一行，有个[ProxyList]，在下面写入socks5 localhost 1080  ，最后保存
  ```

  现在，什么都挡不住我们了，我们可以尽情的下载我们想要的视频和资料：

  ```shell
  # 现在我们输入：proxychains lux -C https://www.bilibili.com/video/BV1D341177ER?spm_id_from=333.999.0.0
  # 以下是产生的内容：
  [PID16432] [I] 2022/03/04 11:21:38 Mswsock.dll (FP)ConnectEx(416 127.0.0.1:37712 16) DIRECT
  [PID16432] [I] 2022/03/04 11:21:38 Mswsock.dll (FP)ConnectEx(524 127.0.0.1:37712 16) DIRECT
  [PID16432] [I] 2022/03/04 11:21:38 Mswsock.dll (FP)ConnectEx(564 127.0.0.1:37712 16) DIRECT
  [PID16432] [I] 2022/03/04 11:21:38 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:38 Mswsock.dll (FP)ConnectEx(652 224.182.190.198:443 16) -> www.bilibili.com:443 PROXY
  [PID16432] [I] 2022/03/04 11:21:39 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:39 Mswsock.dll (FP)ConnectEx(768 224.31.238.12:443 16) -> api.bilibili.com:443 PROXY
  [PID16432] [I] 2022/03/04 11:21:40 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:40 Mswsock.dll (FP)ConnectEx(788 224.49.235.86:9305 16) -> 1xclc7sd.v1d.szbdyd.com:9305 PROXY
  [PID16432] [I] 2022/03/04 11:21:40 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:43 Mswsock.dll (FP)ConnectEx(808 224.237.207.59:1193 16) -> cl78gzny.v1d.szbdyd.com:1193 PROXY
  [PID16432] [I] 2022/03/04 11:21:43 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:43 Mswsock.dll (FP)ConnectEx(820 224.230.146.165:443 16) -> cn-zjjh4-dx-v-11.bilivideo.com:443 PROXY
  [PID16432] [I] 2022/03/04 11:21:44 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:44 Mswsock.dll (FP)ConnectEx(796 224.244.177.173:9305 16) -> ozc26csk.v1d.szbdyd.com:9305 PROXY
  [PID16432] [I] 2022/03/04 11:21:44 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:44 Mswsock.dll (FP)ConnectEx(652 224.59.105.205:4483 16) -> xy125x70x163x57xy.mcdn.bilivideo.cn:4483 PROXY
  [PID16432] [I] 2022/03/04 11:21:44 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:44 Mswsock.dll (FP)ConnectEx(796 224.177.44.141:9305 16) -> 1znkz71o.v1d.szbdyd.com:9305 PROXY
  [PID16432] [I] 2022/03/04 11:21:44 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:44 Mswsock.dll (FP)ConnectEx(824 224.90.8.7:4483 16) -> xy182x148x15x157xy.mcdn.bilivideo.cn:4483 PROXY
  [PID16432] [I] 2022/03/04 11:21:44 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:44 Mswsock.dll (FP)ConnectEx(792 224.14.168.192:4483 16) -> xy171x222x122x86xy.mcdn.bilivideo.cn:4483 PROXY
  [PID16432] [I] 2022/03/04 11:21:45 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:45 Mswsock.dll (FP)ConnectEx(652 224.82.156.85:9305 16) -> 1znkz6wh.v1d.szbdyd.com:9305 PROXY
  [PID16432] [I] 2022/03/04 11:21:45 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:45 Mswsock.dll (FP)ConnectEx(812 224.31.238.12:80 16) -> api.bilibili.com:80 PROXY
  
   Site:      哔哩哔哩 bilibili.com
   Title:     北京冬奥会，谢谢带给我们惊喜的中国冬奥健儿，都是最棒的！
   Type:      video
   Stream:
       [80-7]  -------------------
       Quality:         高清 1080P avc1.640032
       Size:            14.58 MiB (15288560 Bytes)
       # download with: lux -f 80-7 ...
  
  
  Downloading captions...
  Downloading danmaku ...
  [PID16432] [I] 2022/03/04 11:21:46 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:46 Mswsock.dll (FP)ConnectEx(652 224.248.186.45:443 16) -> comment.bilibili.com:443 PROXY
  [PID16432] [I] 2022/03/04 11:21:46 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:46 <> localhost:1080
  [PID16432] [I] 2022/03/04 11:21:46 Mswsock.dll (FP)ConnectEx(824 224.49.235.86:9305 16) -> 1xclc7sd.v1d.szbdyd.com:9305 PROXY
  [PID16432] [I] 2022/03/04 11:21:46 Mswsock.dll (FP)ConnectEx(820 224.237.207.59:1193 16) -> cl78gzny.v1d.szbdyd.com:1193 PROXY
  1.12 MiB / 14.58 MiB [=====>-----------------------------------------------------------------] 935.44 KiB p/s 7.70% 14s
  
  ```

  我建议大家不用去看这玩意儿为啥能够下载B站视频，因为源码中实际上是去请求了B站中的一个API，获取到了m3u8的内容，请求此API需要密钥和签名，至于这个密钥和签名方式Lux的作者为什么会知道，我还真在issues上中找到答案，实际上是通过安卓端反编译找到的。所以这个坑我为大家踩了，不用看源码看是如何找到的了，这个工具拿来用即可。

  

  ##### 最后有不清楚的地方，欢迎大家来在咱们GOCN下进行留言互动。

  

  ## 参考资料

  * https://github.com/shunf4/proxychains-windows
  * https://github.com/iawia002/lux
  * https://ffmpeg.org/
  * https://github.com/armon/go-socks5
  * https://golang.org/x/crypto/ssh