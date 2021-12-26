# 可自搭建网盘并支持多家云存储的公有云文件系统cloudreve

## 什么是 cloudreve?

Cloudreve 可以让您快速搭建起公私兼备的网盘系统。Cloudreve 在底层支持不同的云存储平台，用户在实际使用时无须关心物理存储方式。你可以使用 Cloudreve 搭建个人用网盘、文件分享系统，亦或是针对大小团体的公有云系统。

## cloudreve 特性

* ☁️ 支持本机、从机、七牛、阿里云 OSS、腾讯云 COS、又拍云、OneDrive (包括世纪互联版) 作为存储端
* 📤 上传/下载 支持客户端直传，支持下载限速
* 💾 可对接 Aria2 离线下载，可使用多个从机机点分担下载任务
* 📚 在线 压缩/解压缩、多文件打包下载
* 💻 覆盖全部存储策略的 WebDAV 协议支持
* ⚡ 拖拽上传、目录上传、流式上传处理
* 🗃️ 文件拖拽管理
* 👩‍👧‍👦 多用户、用户组
* 🔗 创建文件、目录的分享链接，可设定自动过期
* 👁️‍🗨️ 视频、图像、音频、文本、Office 文档在线预览
* 🎨 自定义配色、黑暗模式、PWA 应用、全站单页应用
* 🚀 All-In-One 打包，开箱即用
* 🌈 ... ...

## 技术栈

* Go
* Gin
* React

## Docker 一键部署

```shell
docker run -d --name cloudreve -e PUID=1000 -e PGID=1000 -e TZ="Asia/Shanghai" -p 5212:5212 --restart=unless-stopped -v $PWD/uploads:/cloudreve/uploads -v $PWD/config:/cloudreve/config -v $PWD/db:/cloudreve/db -v $PWD/avatar:/cloudreve/avatar xavierniu/cloudreve
```

获取登录信息

```shell
docker logs -f cloudreve
```

![image-20211226174129669](https://resource.gocloudcoder.com/image-20211226174129669.png)

访问 `localhost:5212`即可完成登录

同样可以一键部署在服务器中

默认文件存储在本地目录

![image-20211226174318575](https://resource.gocloudcoder.com/image-20211226174318575.png)

进入管理面板后可以设置存储策略, 非常的方便

![image-20211226174418035](https://resource.gocloudcoder.com/image-20211226174418035.png)

## 个人服务器部署展示

设置存储策略为七牛云存储，并设置相关域名以及 bucket 

使用起来非常的方便

有时间的话还可以查看源码是如何实现的

![image-20211226174608478](https://resource.gocloudcoder.com/image-20211226174608478.png)

## 参考链接

* https://github.com/cloudreve/Cloudreve