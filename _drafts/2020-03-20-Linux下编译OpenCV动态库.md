---
layout:     post
title:      Linux下安装编译OpenCV动态库
subtitle:   Linux下编译安装OpenCV动态库
date:       2020-03-20
author:     yym439
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - Linux OpenCV
---

## 1. 编译工具安装

### 1.1 安装cmake与cmake gui

[官网下载地址](https://cmake.org/download/)

[其他下载地址](https://cmake.org/files/v3.17/)
```
官网直接下载已经编译过的二进制cmake版本:

1.解压： tar -zxvf cmake-3.17.0-rc3-Linux-x86_64\ \(1\).tar.gz

2.添加全局环境变量(添加完打开新的终端才会生效)：
sudo vim /home/yym/.bashrc
export PATH=$PATH:/home/yym/cmake-3.17.0-rc3-Linux-x86_64/bin

3. 打开cmake界面
cmake­-gui
```


## 2. 下载OpenCV源码

[官网](https://opencv.org/)

### 2.1  cmake静态编译OpenCV

```
1. 解压Opencv ,在解压的目录下新建Release文件夹,进入文件夹中

2.右键终端打开:cmake-­gui选择源码路径为Opencv解压出的文件夹,build路径为创建好的Release,勾上Advanced选项,点击configure.
注意：编译静态库需要把BUILD_SHARED_LIBS选项勾去除

3.点击configure之后,会跳出CmakeSetup窗口,选择Unix MakeFiles + Use default native compliers

4.configure一次之后,窗口中会出现cmake选项,
选择我们需要的模块,去掉不需要的模块,动态库调用opencv静态库之后(动态库即包含了opencv的库功能)

5.点击configure,把出现红框的选项勾去除

6.再点击configure,生成完毕后再点击Generate,提示:Generating Done之后即可关闭cmake­-gui

7.进入Release目录内,右键打开终端,输入指令make回车,即开始编译,

8.第三方库位置: /home/yym/opencv-3.4.9/release/3rdparty/lib
  opencv库位置: /home/yym/opencv-3.4.9/release/lib
  头文件位置: /home/yym/opencv-3.4.9/include

9.编译通过后,使用sudo make install直接安装,也可以直接获取静态库文件和头文件进行调用

```

### 2.2 linux编写动态库调用opencv静态库编程



## 3. OpenCV库模块说明

[官网模块说明](https://docs.opencv.org/3.4.9/)

```
1. core模块: 定义了Opencv最为基础的数据结构，必须要有。

2. imgproc: 图片的处理

3. imgcodecs: 用于图片的读写

```