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

1. [下载NDK-r15 for linux](https://blog.csdn.net/gyh198/article/details/75036686)

2. [FFmpeg3.4.8源码下载](http://ffmpeg.org/download.html)

3. 修改configure文件

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

4. 在ffmpeg根目录下新建androidBuilder.sh脚本：

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

4. 运行编译：./androidBuilder.sh
    armeabi-v7a目录下的include和lib就是我们引入Android需要使用的头文件和动态链接库

5. 编译失败问题：

- NDK包含B0宏定义,导致编译失败：
    - libavcodec/aaccoder.c 修改B0为b0
    - libavcodec/hevc_mvs.c 修改B0为b0、修改xB0 yB0为xb0 yb0
    - libavcodec/opus_pvq.c 修改B0为b0

6. [参考博客](https://www.jianshu.com/p/df9401e6fba5?utm_campaign=haruki&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)