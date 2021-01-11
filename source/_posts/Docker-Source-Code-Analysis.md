---
title: Docker Source Code Analysis
date: 2020-06-06 09:07:21
tags:
---

# Docker 源码分析（一）

## 总体代码结构

Docker总体上是用的是Client/Server模式，所有的命令都可以通过RESTful接口传递。

整个Docker软件的架构中可以分成三个角色：

- Daemon：常驻后台运行的进程，接收客户端请求，管理docker容器。
- Client：命令行终端，包装命令发送API请求。
- Engine：真正处理客户端请求的后端程序。

## 代码结构

Docker的代码结构比较清晰，分成的目录比较多，有以下这些：

- api：定义API，使用了Swagger2.0这个工具来生成API，配置文件在api/swagger.yaml
- builder：用来build docker镜像的包，看来历史比较悠久了
- bundles：这个包是在进行docker源码编译和开发环境搭建的时候用到的，编译生成的二进制文件都在这里。
- cli：使用cobra工具生成的docker客户端命令行解析器。
- client：接收cli的请求，调用RESTful API中的接口，向server端发送http请求。
- cmd：其中包括docker和dockerd两个包，他们分别包含了客户端和服务端的main函数入口。
- container：容器的配置管理，对不同的platform适配。
- contrib：这个目录包括一些有用的脚本、镜像和其他非docker core中的部分。
- daemon：这个包中将docker deamon运行时状态expose出来。
- distribution：负责docker镜像的pull、push和镜像仓库的维护。
- dockerversion：编译的时候自动生成的。
- docs：文档。这个目录已经不再维护，文档在另一个仓库里。
- experimental：从docker1.13.0版本起开始增加了实验特性。
- hack：创建docker开发环境和编译打包时用到的脚本和配置文件。
- image：用于构建docker镜像的。
- integration-cli：集成测试
- layer：管理 union file system driver上的read-only和read-write mounts。
- libcontainerd：访问内核中的容器系统调用。
- man：生成man pages。
- migrate：将老版本的graph目录转换成新的metadata。
- oci：Open Container Interface库
- opts：命令行的选项库。
- pkg：
- plugin：docker插件后端实现包。
- profiles：里面有apparmor和seccomp两个目录。用于内核访问控制。
- project：项目管理的一些说明文档。
- reference：处理docker store中镜像的reference。
- registry：docker registry的实现。
- restartmanager：处理重启后的动作。
- runconfig：配置格式解码和校验。
- vendor：各种依赖包。
- volume：docker volume的实现。