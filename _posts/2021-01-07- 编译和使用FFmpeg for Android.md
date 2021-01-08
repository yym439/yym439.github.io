---
layout:     post
title:      编译和使用FFmpeg for Android
subtitle:   编译和使用FFmpeg for Android 
date:       2021-01-07
author:     yym439
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - Android
    - FFmpeg
---

### 一、 Linux下编译FFmpeg for Android

#### 1. [下载NDK-r15 for linux](https://blog.csdn.net/gyh198/article/details/75036686)

#### 2. [FFmpeg3.4.8源码下载](http://ffmpeg.org/download.html)

#### 3. 修改configure文件

```
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR)$(SLIBNAME)'
替换为：
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```

#### 4. 在ffmpeg根目录下新建androidBuilder.sh脚本：

```
#!/bin/bash
# 修改为自己NDK包根目录
export NDK_HOME=/Users/parker/Library/Android/sdk/ndk/android-ndk-r15c
#根据自己的需求修改编译平台版本
export PLATFORM_VERSION=android-21
#定义编译方法
function build
{
    #echo 输出命令
    echo "start build ffmpeg for $ARCH"
    #调用configure命令开始编译，并传入对应的参数
    ./configure --target-os=linux \
    --prefix=$PREFIX --arch=$ARCH \
    --disable-doc \
    --disable-static \
    --disable-yasm \
    --disable-asm \
    --disable-symver \
    --disable-ffmpeg \
    --disable-ffplay \
    --disable-ffprobe \
    --disable-ffserver \
    --cross-prefix=$CROSS_COMPILE \
    --enable-cross-compile \
    --enable-shared \
    --enable-gpl \
    --sysroot=$SYSROOT \
    --enable-small \
    --extra-cflags="-Os -fpic $ADDI_CFLAGS" \
    --extra-ldflags="$ADDI_LDFLAGS" \
    $ADDITIONAL_CONFIGURE_FLAG
    make clean
    make
    make install
    echo "build ffmpeg for $ARCH finished"
}

#编译 arm-v7a
PLATFORM_VERSION=android-21
ARCH=arm
CPU=armeabi-v7a #CPU架构
PREFIX=$(pwd)/android_all/$CPU  #输出路径：当前目录/android_all/CPU架构/
TOOLCHAIN=$NDK_HOME/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64
CROSS_COMPILE=$TOOLCHAIN/bin/arm-linux-androideabi- #交叉编译环境路径
ADDI_CFLAGS="-march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16 -mthumb -mfpu=neon"
ADDI_LDFLAGS="-march=armv7-a -Wl,--fix-cortex-a8"
SYSROOT=$NDK_HOME/platforms/$PLATFORM_VERSION/arch-$ARCH/
build
```

#### 5. 运行编译
    
- ./androidBuilder.sh
- armeabi-v7a目录下的include和lib就是我们引入Android需要使用的头文件和动态链接库

#### 6. 编译失败问题：

- NDK包含B0宏定义,导致编译失败：
    - libavcodec/aaccoder.c 修改B0为b0
    - libavcodec/hevc_mvs.c 修改B0为b0、修改xB0 yB0为xb0 yb0
    - libavcodec/opus_pvq.c 修改B0为b0

- [参考博客](https://www.jianshu.com/p/df9401e6fba5?utm_campaign=haruki&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)


### 一、Android中使用FFmpeg

#### 1. 创建Native C++工程

- 拷贝include文件夹到cpp中
- 拷贝so库到src\main\jniLibs中

#### 2.编写CMakeLists.txt,链接第三方so库
```
cmake_minimum_required(VERSION 3.10.2)

#设置第三方库头文件路径,需要设置，不然导入头文件会找不到
include_directories(${CMAKE_SOURCE_DIR}/include)

# 设置第三方库路径,不要使用相对路径(导入第三方库会找不到)
set(DIR ${CMAKE_SOURCE_DIR}/../jniLibs)

#设置工程名称
project("avplayer")

#打印变量信息：在Gradle-app-buil-run中编译(在Run中查看日志)
message("yym: ${DIR}")

#导入第三方库
add_library(avcodec-57
        SHARED
        IMPORTED)
set_target_properties(avcodec-57
        PROPERTIES IMPORTED_LOCATION
        ${DIR}/${ANDROID_ABI}/libavcodec-57.so)

add_library(avdevice-57
        SHARED
        IMPORTED)
set_target_properties(avdevice-57
        PROPERTIES IMPORTED_LOCATION
        ${DIR}/armeabi-v7a/libavdevice-57.so)

add_library(avformat-57
        SHARED
        IMPORTED)
set_target_properties(avformat-57
        PROPERTIES IMPORTED_LOCATION
        ${DIR}/armeabi-v7a/libavformat-57.so)

add_library(avutil-55
        SHARED
        IMPORTED)
set_target_properties(avutil-55
        PROPERTIES IMPORTED_LOCATION
        ${DIR}/armeabi-v7a/libavutil-55.so)

add_library(postproc-54
        SHARED
        IMPORTED)
set_target_properties(postproc-54
        PROPERTIES IMPORTED_LOCATION
        ${DIR}/armeabi-v7a/libpostproc-54.so)

add_library(swresample-2
        SHARED
        IMPORTED)
set_target_properties(swresample-2
        PROPERTIES IMPORTED_LOCATION
        ${DIR}/armeabi-v7a/libswresample-2.so)

add_library(swscale-4
        SHARED
        IMPORTED)
set_target_properties(swscale-4
        PROPERTIES IMPORTED_LOCATION
        ${DIR}/armeabi-v7a/libswscale-4.so)

add_library(avfilter-6
        SHARED
        IMPORTED)
set_target_properties(avfilter-6
        PROPERTIES IMPORTED_LOCATION
        ${DIR}/armeabi-v7a/libavfilter-6.so)

# 创建动态库
add_library(
            native-lib
            SHARED

            mylog.c
            native-lib.cpp )

#搜素系统库到指定变量
find_library( 
              log-lib
              log )

#链接需要的第三方动态库
target_link_libraries(
                        native-lib
                        avfilter-6
                        avcodec-57
                        avdevice-57
                        avformat-57
                        avutil-55
                        postproc-54
                        swresample-2
                        swscale-4
                        ${log-lib} )
```


#### 3.修改app build.gradle配置文件

```
android {
    defaultConfig {
        ndk {
            abiFilters "armeabi-v7a"
        }
    }
    sourceSets.main {
        jniLibs.srcDirs = ['jniLibs']
    }
    externalNativeBuild {
        cmake {
            path "src/main/cpp/CMakeLists.txt"
            version "3.10.2"
        }
    }
}
```