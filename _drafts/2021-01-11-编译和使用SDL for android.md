---
layout:     post
title:      编译和使用SDL for android
subtitle:   编译和使用SDL for android
date:       2021-01-
author:     yym439
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Android
---

### 一、windows下编译SDL Android库
1. [源码](https://www.libsdl.org/download-2.0.php)

2. 进入android-project\app\jni目录：创建SDL文件夹

3. 拷贝根目录：include、src、Android.mk 到 SDL文件夹

4. 进入android-project\app\jni\src：创建YourSourceHere.c(自定义实现sdl功能，可以写个空函数)
    ```
    #include <stdio.h> 
    int main(int argc, char *argv[]) {
        return 0;
    }
    ```
5. cmd 进入‘android-project\app\jni目录’：执行，ndk-build,即可生成 libSDL2.so

