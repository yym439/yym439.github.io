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

## 1、Dockerfile

- Dockerfile是一个文本格式的配置文件, 用户可以使用 Dockerfile 来快速创建自定义的镜像

- 由一行行命令语句组成，支持以#开头的注释

## 2、基本结构

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

## 3、指令说明


Dockerfile指令分为：配置指令（配置镜像信息）和操作指令（具体执行操作）

### 3.1 配置指令

![配置指令](https://yym439.github.io/img/dockerfile-1.jpg "配置指令")

### 3.2 操作指令

![配置指令](https://yym439.github.io/img/dockerfile-2.jpg "配置指令")


## 4、创建镜像

```shell
#创建完Dockerfile后，即可运行创建镜像
docker build -t yym439/myjava:1.0 (镜像名称) . (Dockerfile文件路径)
```

## 5、Dockerfile示例 

### 5.1 创建renren-fast后端服务镜像

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

### 5.2 镜像添加ssh服务


编写run.sh：
```
#!/bin/bash 
/usr/sbin/sshd -D

```

编写authorized_keys：
``` shell
ssh-keygen -t  rsa

cat -/  ssh/id rsa.pub >authorized_keys 
```

编写dockerfile：
```dockerfile
#设置继承镜像
FROM  ubuntu:18.04
#提供一些作者的信息
MAINTAINER docker_user (user@docker.com)
#下面开始运行命令， 此处更改ubuntu的源为国内163的源
#RUN echo  "deb http://mirrors.163.com/ubuntu/  bionic main restricted universe
#multiverse" >>  /etc/apt/sources.list
#RUN echo  "deb http://mirrors.163.com/ubuntu/  bionic-security main restricted
#universe multiverse" >> /etc/apt/sources.list
#RUN echo  "deb  http://mirrors.163.com/ubuntu/  bionic-updates main restricted
#universe multiverse" >> /etc/apt/sources.list
#RUN echo  "deb http://mirrors.163.com/ubuntu/ bionic-proposed main restricted
#universe multiverse" >> /etc/apt/sources.list
#RUN echo  "deb http://mirrors.163.com/ubuntu/ bionic-backports main restricted
#universe multiverse" >> /etc/apt/sources.list
RUN apt-get update

#安装ssh 服务
RUN apt-get install -y openssh-server
RUN mkdir -p  /var/run/sshd
RUN mkdir -p /root/.ssh

#取消pam限制
RUN sed -ri  's/session  required  pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd

#复制配置文件到相应位置， 并赋予脚本可执行权限
ADD authorized_keys /root/.ssh/authorized_keys
ADD run.sh /run.sh
RUN chmod 755 /run.sh

#开放端口
EXPOSE 22
#设置自启动命令
CMD  ["/run.sh"]
```

创建镜像

> docker build -t sshd:dockerfile .


运行容器：

> docker run -d -p 10022:22 sshd

在宿主机打开一个终端，连接到新建的容器：

> ssh 宿主机ip -p 10022