# 快速启动一个管理平台项目GIN-VUE-ADMIN

## 推荐理由

开发一个管理平台，或者工具服务，其中免不了许多重复的工作，如果有个基础功能完成的脚手架 就比较好了，Gin-Vue-admin 就提供了这种脚手架的能力，同时官网配置详细的视频教程，非常时候新生使用。项目体验链接：[https://www.gin-vue-admin.com/docs/experience](https://www.gin-vue-admin.com/docs/experience)

## 简介

GIN-VUE-ADMIN是一个基于vue和gin开发的全栈前后端分离的开发基础平台，拥有jwt鉴权，动态路由，动态菜单，casbin鉴权，表单生成器，代码生成器等功能，提供了多种示例文件，让大家把更多时间专注在业务开发上。

## 快速开始

### 技术选型

- 前端：用基于 [Vue](https://vuejs.org/) 的 [Element](https://github.com/ElemeFE/element) 构建基础页面。
- 后端：用 [Gin](https://gin-gonic.com/) 快速搭建基础restful风格API，[Gin](https://gin-gonic.com/) 是一个go语言编写的Web框架。
- 数据库：采用`MySql`(5.6.44)版本，使用 [gorm](http://gorm.cn/) 实现对数据库的基本操作。
- 缓存：使用`Redis`实现记录当前活跃用户的`jwt`令牌并实现多点登录限制。
- API文档：使用`Swagger`构建自动化文档。
- 配置文件：使用 [fsnotify](https://github.com/fsnotify/fsnotify) 和 [viper](https://github.com/spf13/viper) 实现`yaml`格式的配置文件。
- 日志：使用 [zap](https://github.com/uber-go/zap) 实现日志记录。

### 环境准备

```
- node版本 > v12.18.3
- golang版本 >= v1.16
# 克隆项目
git clone https://github.com/flipped-aurora/gin-vue-admin.git
```

## 配置调整

```go
# config.yaml 为项目配置，包含如下配置
# JWT：jwt token 配置
# Zap：日志配置
# Redis：缓存配置
# Email：邮件配置
# system：环境配置
# captcha： 验证码配置
# mysql： 数据库配置
# Local： 本地上传文件配置
# Qiniu：静态资源存储，七牛云存储配置

```

### 服务端启动

```go
# 进入server文件夹
cd server

# 使用 go mod 并安装go依赖包
go generate

# 编译
go build -o server main.go (windows编译命令为go build -o server.exe main.go )

# 运行二进制
./server (windows运行命令为 server.exe)
```

## 启动web端

```go
# 进入web文件夹
cd web

# 安装依赖
cnpm install || npm install

# 启动web项目
```

## 展示

**项目目录结构**

```go
├─server         （后端文件夹）
    │  ├─api            （API）
    │  ├─config         （配置包）
    │  ├─core           （核心文件）
    │  ├─docs           （swagger文档目录）
    │  ├─global         （全局对象）
    │  ├─initialiaze    （初始化）
    │  ├─middleware     （中间件）
    │  ├─model          （结构体层）
    │  ├─resource       （资源）
    │  ├─router         （路由）
    │  ├─service         (服务)
    │  ├─source         (初始化需要的数据)
    │  ├─plugin         (插件)
    │  └─utils          （公共功能）
    └─web            （前端文件）
        ├─public        （发布模板）
        └─src           （源码包）
            ├─api       （向后台发送ajax的封装层）
            ├─core       （用来修改系统基础可运行配置）
            ├─assets    （静态文件）
            ├─components（组件）
            ├─router    （前端路由）
            ├─store     （vuex 状态管理仓）
            ├─style     （通用样式文件）
            ├─utils     （前端工具库）
            └─view      （前端页面）
```

**项目效果图**

![Untitled](%E5%BF%AB%E9%80%9F%E5%90%AF%E5%8A%A8%E4%B8%80%E4%B8%AA%E7%AE%A1%E7%90%86%E5%B9%B3%E5%8F%B0%2051181/Untitled.png)

 

![Untitled](%E5%BF%AB%E9%80%9F%E5%90%AF%E5%8A%A8%E4%B8%80%E4%B8%AA%E7%AE%A1%E7%90%86%E5%B9%B3%E5%8F%B0%2051181/Untitled%201.png)

## 参考资料

GIN-VUE-ADMIN官网：[https://www.gin-vue-admin.com/docs/deployment](https://www.gin-vue-admin.com/docs/deployment)