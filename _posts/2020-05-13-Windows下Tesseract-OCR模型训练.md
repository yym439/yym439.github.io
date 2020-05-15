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
## 一、工具下载
[V3.05参考博客](https://blog.csdn.net/ruyulin/article/details/89046148)

[V4.10参考博客](https://zhuanlan.zhihu.com/p/77013854)

## 二、数据采集，手动矫正

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

## 三 、生成训练模型数据

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


## 四、测试

```
1. 将num.traineddata文件拷贝到C:\Program Files (x86)\Tesseract-OCR\tessdata目录下，
可以进行测试。

2.cmd输入测试指令：
    tesseract 1.png result -l num
```




