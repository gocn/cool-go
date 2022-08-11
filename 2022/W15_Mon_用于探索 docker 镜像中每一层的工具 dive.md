# 用于探索 docker 镜像中每一层的工具 dive

## 什么是 dive?

用于探索 Docker 镜像、每一层中的内容以及发现缩小 Docker/OCI 镜像大小的方法的工具。

## 安装 dive

```go
go get github.com/wagoodman/dive
```

## dive 特性

* 按层分解 Docker 镜像
* 可视化展示每一层变化
* 分析镜像空间使用百分比
* 快速构建分析镜像
* 支持多种镜像源和容器引擎

## 入门使用

以我自己写的项目做的镜像为例

```shell
dive <your-image-tag>
```

![image-20220405193448424](https://resource.gocloudcoder.com/image-20220405193448424.png)

按层分解 Docker 镜像并且能可视化的展示了每一层做了什么事情。

也可以基于 Dockerfile 快速构建分析。

```shell
dive build -t <some-tag> .
```

或者基于 docker 压缩文件镜像

```shell
dive docker-archive://<your-image>
```

##  巨人的肩膀

* https://github.com/wagoodman/dive