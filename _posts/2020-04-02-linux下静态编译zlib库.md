---
layout:     post
title:      linux下静态编译zlib库
subtitle:   linux下静态编译zlib库
date:       2020-04-02
author:     yym439
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - Linux
---

## 1.下载编译ZLIB源码

[官网](http://www.zlib.net/)

```
1.解压：
tar -xvf zlib-1.2.11.tar.gz

2.修改CMakeLists.txt文件,新增：
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
注意：编译静态库需要添加-fPIC，否则动态库链接该静态库的时候会提示出错
（relocation R_X86_64_32 against）。

3. cmake .
4. make install
```