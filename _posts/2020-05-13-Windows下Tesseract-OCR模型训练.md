---
layout:     post
title:      Windows下Tesseract-OCR模型训练
subtitle:   Tesseract-OCR-3.0.5 数字识别训练与合并多次训练数据
date:       2020-05-13
author:     yym439
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - OCR
---
## 1、工具下载
[V3.05参考博客1](https://blog.csdn.net/ruyulin/article/details/89046148)

[V3.05参考博客2](https://www.cnblogs.com/xpwi/p/9604567.html)

[V4.10参考博客](https://zhuanlan.zhihu.com/p/77013854)

### 1.1 预备知识
在开始准备用于训练的图片之前，建议先看看官方Wiki的[ImproveQuality](https://tesseract-ocr.github.io/tessdoc/ImproveQuality)提高图像质量的说明，重要几点是：
- Tesseract在DPI为至少为300 DPI的图像上效果最佳，所以需要考虑提高图片的DPI，一般图片默认的都是72 。
- 将图像转换为黑白（就是二值化）。
- 消除噪点（就是降低干扰）。
- 旋转（就是将文字尽量水平）。
- 移除不必要的边界（就是要适当裁剪图片）。

## 2、数据采集，手动矫正

```
 1.生成tif文件：
    打开jTessBoxEditor工具后，点击Tools，点Merge TIFF，
    选中num-1文件夹中所有图片，点击打开。保存名：num.font.exp1.tif

 2.生成bok文件：
    tesseract num.font.exp1.tif num.font.exp1 batch.nochop makebox

 3.字符和位置进行校正
    使用jTessBoxEditor工具打开每个tif进行字符和位置校正，然后保存

 4.生成TR文件
    tesseract num.font.exp1.tif num.font.exp1 nobatch box.train
```

## 3 、生成训练模型数据

```
1. 新建字体特征文件
    创建一个名称为font_properties的字体特征文件；
    用记事本打开，输入以下下内容：font 0 0 0 0 0
    这里全取值为0，表示字体不是粗体、斜体等等

2. 从所有文件中提取字符
    unicharset_extractor num.font.exp1.box num.font.exp2.box num.font.exp3.box

3. 生成shape文件
    shapeclustering -F font_properties -U unicharset num.font.exp1.tr num.font.exp2.tr num.font.exp3.tr

4. 生成聚集字符特征文件
    mftraining -F font_properties -U unicharset -O unicharset num.font.exp1.tr num.font.exp2.tr num.font.exp3.tr

5. 合并所有tr文件
    cntraining num.font.exp1.tr num.font.exp2.tr num.font.exp3.tr

6. 修改文件名
    把训练过程创建的五个文件：shapetable，normproto，inttemp，pffmtable，unicharset，
    用lang.为前缀重命名（例如num.）

7. 生成训练结果
    combine_tessdata num.

```


## 4、测试

```
1. 将num.traineddata文件拷贝到C:\Program Files (x86)\Tesseract-OCR\tessdata目录下，
可以进行测试。

2.cmd输入测试指令：
    tesseract 1.png result -l num
```

>注意：
生成box文件时报错empty page：[解决](https://blog.csdn.net/fire669842703/article/details/103009578)


## 5、Windows下编译Tesseract 3.05

[参考博客V3.05](https://www.polarxiong.com/archives/Tesseract-3-05%E5%8F%8A%E4%B9%8B%E5%90%8E%E7%89%88%E6%9C%AC%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL.html)

[参考博客V5.0](https://zhuanlan.zhihu.com/p/75737812)

[Windows下静态编译 Tessearct3.05](https://github.com/yym439/Tesseract-OCR_for_Windows)

### 5.1 源码编译库
```
动态库编译：
1. 下载安装cmake
2. 下载cppan
3. cmake编译：
    cd tesseract
    cppan  （下载编译依赖）
    mkdir build
    cd build
    cmake .. (或者使用cmake-gui进行编译)
4. 打开build文件夹，打开.sln工程，INSTALL安装编译生成库及应用程序

静态库编译：
1.修改libtesseract库工程配置：
	配置属性-常规-配置类型（静态库）
	配置属性-高级-目标文件拓展名（.lib）
	配置属性-C/C++-代码生成器-运行库（MT）

注：MT表示静态库；MD表示动态库; MT/MD多个d表示调试模式
	
```

### 5.2 VC工程引用第三方库

```
动态库工程配置：
    1.属性-VC++目录-包含目录（添加头文件目录）-库目录（动态库的lib文件目录，隐式调用）
    2.链接器-输入-附加依赖项（动态库的lib文件名称）
	3.拷贝dll文件到exe目录（或者设置dll目录）


静态库工程配置：
	1.属性-VC++目录-包含目录（添加头文件目录）
	2.链接器-常规-附加库目录（静态库lib文件目录）
	3.链接器-输入-附加依赖项（静态库的lib文件名称）
```

>注意：
动态链接库dll目录配置方法：
	1.配置属性->调试->环境：输入path=包含dll文	件的文件夹路径（=左右不能有空格）
	2.将dll文件拷贝到生成的.exe所在的文件夹中

调用示例：
```
#include <iostream>
#include <memory>
#include <allheaders.h> // leptonica main header for image io
#include <baseapi.h>

int main(int argc, char* argv[])
{
	tesseract::TessBaseAPI tess;
	//if (tess.Init("E:/OpenCV_DNN数据集/tessdata", "eng"))
	if (tess.Init("C:\\Program Files (x86)\\tesseract\\tessdata", "num"))

	{
		std::cout << "OCRTesseract: Could not initialize tesseract." << std::endl;
		return 1;
	}
	// setup
	tess.SetPageSegMode(tesseract::PageSegMode::PSM_AUTO);
	tess.SetVariable("save_best_choices", "T");
	// read image
	auto pixs = pixRead("4.png");
	if (!pixs)
	{
		std::cout << "Cannot open input file: " << argv[1] << std::endl;
		return 1;
	}
	// recognize
	tess.SetImage(pixs);
	tess.Recognize(0);
	// get result and delete[] returned char* string
	std::cout << std::unique_ptr<char[]>(tess.GetUTF8Text()).get() << std::endl;
	// cleanup
	tess.Clear();
	pixDestroy(&pixs);
	return 0;
}
```