# 免费！让Https证书不再成为烦恼

## 前言

  书承上回，上期写了《实战goproxy为中国steam登录加速》后，总是觉得服务端获取到的客户端信息太少了，为了获取更多的客户端信息，就需要解包设置一个中间代理人，我简简单单的用《Lego》申请了证书，两种申请方式，一种直接就是cli，一种是代码内申请直接使用。



#### 献上letsencrypt的申请证书的工作原理，就知道为什么有申请和挑战的环节了：

![image](https://letsencrypt.org/images/howitworks_authorization.png)



## 先介绍Lego 的 cli方式获取证书与更新证书

```shell
# 通过源码进行安装
go install github.com/go-acme/lego/v4/cmd/lego@latest

# -----------------申请证书-----------------
lego --email="87066062@qq.com" --domains="www.laghaim.cn" --http run
# 申请的证书有效期为90天
# 申请成功后就会获得一下文件了。
# www.laghaim.cn.crt
# www.laghaim.cn.issuer.crt
# www.laghaim.cn.json
# www.laghaim.cn.key

# -----------------更新证书-----------------
lego --email="87066062@qq.com" --domains="www.laghaim.cn" --http renew
# 但是此时需要注意，这时续订必须要小于30天

# 如果想提前更新证书，指定剩余天数即可：
lego --email="87066062@qq.com" --domains="www.laghaim.cn" --http renew --days 45
```

如果想自动续租，写个crontab就可以咯。

## 

## 接下来就是利用Lego的库代码进行申请证书和续订证书

关键地方我都进行了注释，我喜欢一次性把整段代码贴出来，大家也可以直接粘贴整段对部分常量参数进行修改后，直接调用GetCertificateFromLego()函数

```go
package get_certificate_from_lego

import (
    "crypto"
    "crypto/ecdsa"
    "crypto/elliptic"
    "crypto/rand"

    "github.com/go-acme/lego/v4/certcrypto"
    "github.com/go-acme/lego/v4/certificate"
    "github.com/go-acme/lego/v4/challenge/http01"
    "github.com/go-acme/lego/v4/challenge/tlsalpn01"
    "github.com/go-acme/lego/v4/lego"
    "github.com/go-acme/lego/v4/registration"
)

const (
    EmailStr  = "87066062@qq.com" // 修改为自己的电子邮件
    OneDomain = "www.laghaim.cn"  // 修改为自己的域名
)

type MyUser struct {
    Email        string
    Registration *registration.Resource
    key          crypto.PrivateKey
}

func (u *MyUser) GetEmail() string {
    return u.Email
}
func (u MyUser) GetRegistration() *registration.Resource {
    return u.Registration
}
func (u *MyUser) GetPrivateKey() crypto.PrivateKey {
    return u.key
}

func GetCertificateFromLego() (*certificate.Resource, error) {
    // 创建myUser用户对象。新对象需要email和私钥才能启动，私钥我们自己生成
    privateKey, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
    if err != nil {
        return nil, err
    }

    myUser := MyUser{
        Email: EmailStr,
        key:   privateKey,
    }

    config := lego.NewConfig(&myUser)

    // 这里配置密钥的类型和密钥申请的地址，记得上线后替换成 lego.LEDirectoryProduction ，测试环境下就用 lego.LEDirectoryStaging
    config.CADirURL = lego.LEDirectoryStaging
    config.Certificate.KeyType = certcrypto.RSA2048

    // 创建一个client与CA服务器通信
    client, err := lego.NewClient(config)
    if err != nil {
        return nil, err
    }

    // 这里需要挑战我们申请的证书，必须监听80和443端口，这样才能让Let's Encrypt访问到咱们的服务器
    err = client.Challenge.SetHTTP01Provider(http01.NewProviderServer("", "80"))
    if err != nil {
        return nil, err
    }
    err = client.Challenge.SetTLSALPN01Provider(tlsalpn01.NewProviderServer("", "443"))
    if err != nil {
        return nil, err
    }

    // 把这个客户端注册，传递给myUser用户里
    reg, err := client.Registration.Register(registration.RegisterOptions{TermsOfServiceAgreed: true})
    if err != nil {
        return nil, err
    }
    myUser.Registration = reg

    request := certificate.ObtainRequest{
        Domains: []string{OneDomain}, // 这里如果有多个，就写多个就好了，可以是多个域名
        Bundle:  true,                // 这里如果是true，将把颁发者证书一起返回，也就是返回里面certificates.IssuerCertificate
    }
    // 开始申请证书
    certificates, err := client.Certificate.Obtain(request)
    if err != nil {
        return nil, err
    }
    // 申请完了后，里面会带有证书的PrivateKey Certificate，都为[]byte格式，需要存储的自行转为string即可
    return certificates, nil
}

// 如果要进行续订，可将certificates, err := client.Certificate.Obtain(request)替换为certificates, err := client.Certificate.Renew(request)
// renew里面的参数就很简单了，第一个参数就是第一次申请返回的指针的值certificates，第二个参数bundle上面已经讲过传true即可，后面两个参数一个传false，一个传空字符串""即可。开启PAC自动配置
```

然后剩下的内容，就是使用和解析打印一下咯啦：

```go
package main
import (
    "crypto/tls"
    "crypto/x509"
    "fmt"
    "log"

    "server/get_certificate_from_lego"
)
func main(){
    cs, err := get_certificate_from_lego.GetCertificateFromLego()
    if err != nil {
        log.Fatalln("obtains certificate:", err)
    }
    ca, err := tls.X509KeyPair(cs.Certificate, cs.PrivateKey)
    if err != nil {
        log.Fatalln(err)
    }
    if ca.Leaf, err = x509.ParseCertificate(ca.Certificate[0]); err != nil {
        log.Fatalln(err)
    }
    fmt.Println(ca.Leaf)
}
```

## 参考资料

[lego](https://github.com/go-acme/lego)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)