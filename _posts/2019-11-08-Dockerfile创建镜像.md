---
layout:     post
title:      Dockerfile创建镜像
subtitle:   使用Dockerfile基本结构、指令，创建镜像
date:       2019-11-08
author:     yym439
header-img: img/post-web.jpg
catalog: true
tags:
    - Docker
---

## Dockerfile

- Dockerfile是一个文本格式的配置文件, 用户可以使用 Dockerfile 来快速创建自定义的镜像

- 由一行行命令语句组成，支持以#开头的注释

## 基本结构

Dockerfile主体内容一般分为：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令

Dockerfile示例：

```Dockerfile
#基础镜像信息
FROM  java:latest

#维护者信息
LABEL  maintainer="yym439@gmail.com"  date="2019-11-08"

#镜像操作指令（安装nginx）
RUN  apt-get update &&  apt-get  install  -y  nginx
RUN  echo  "\ndaernon  off;"  >>  /etc/nginx/nginx.conf

#容器启动时执行指令
CMD  /usr/sbin/nginx
```

## 指令说明


Dockerfile指令分为：配置指令（配置镜像信息）和操作指令（具体执行操作）

### 配置指令

![配置指令](https://yym439.github.io/img/dockerfile-1.jpg "配置指令")

### 操作指令

![配置指令](https://yym439.github.io/img/dockerfile-2.jpg "配置指令")


## 创建镜像

```shell
#创建完Dockerfile后，即可运行创建镜像
docker build -t yym439/myjava:1.0 (镜像名称) . (Dockerfile文件路径)
```

## Dockerfile示例 

### 创建renren-fast后端服务镜像

``` dockerfile
#基础镜像信息
FROM  java:latest
#维护者信息
LABEL  maintainer="yym439@gmail.com"  date="2019-11-08"

#容器内创建文件夹
RUN mkdir /home/yym

#拷贝jar包到容器指定文件夹
COPY renren-fast8080.jar /home/yym

#容器启动时执行命令
CMD ["nohup","java","-jar","/home/yym/renren-fast8080.jar","&"]
```

> 启动容器： docker run -p 8080:8080 -d rr/rr 
