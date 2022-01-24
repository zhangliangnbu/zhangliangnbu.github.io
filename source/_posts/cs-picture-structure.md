---
title: 计算机视觉基础
date: 2021-01-24 17:54:25
tags:
- cs
categories:
- 其他
---

# 数字图像表示

- 二值图：二维矩阵，也叫黑白图，每个点的值用一个比特表示 0-1
- 灰度图：二维矩阵，每个点的值用8比特表示 0-255
- 彩色图：多重二位矩阵

![cs-picture-structure](/images/cs-picture-structure.png)

# 视频和图像格式基础

- 封装格式：视频文件包括编码后的音频和编码后的视频，将这二者封装成一个文件，就涉及到封装格式
- 视频编码：将视频源文件进行编码，主流编解码格式为H264
- 帧格式：视频源文件包含一系列的帧文件，帧有帧的封装格式如YUV

# YUV420-888

扩展：YUV_420_888 介绍

首先YUV是一种颜色空间，基于YUV的颜色编码是流媒体的常用编码方式。“Y”表示明亮度（Luminance、Luma），“U”和“V”则是色度、浓度（Chrominance、Chroma）。

YUV420是一类格式的集合，YUV420并不能完全确定颜色数据的存储顺序。举例来说，对于4x4的图片，在YUV420下，任何格式都有16个Y值，4个U值和4个V值，不同格式只是Y、U和V的排列顺序变化。

I420（YUV420Planar的一种）则为YYYYYYYYYYYYYYYYUUUUVVVV，NV21（YUV420SemiPlanar）则为YYYYYYYYYYYYYYYYUVUVUVUV。

在Camera2中，YUV_420_888通常为YUV420Planar排列（手机不同可能会存在差异，目前我用过的几个手机都是这个格式）。实际应用中，可以根据采集到的数据中的pixelStride值判断具体的格式，再去做数据转换，其中pixelStride代表行内颜色值间隔。

![cs-yuv-structure](/images/cs-yuv-structure.png)

说明：

- 图中PixelStride表示一个颜色点的字节长度
- 图中LineStride也叫RowStride，表示一行字节长度

<!--more-->

# Android 华为30Pro摄像头示例

```bash
format=35, width=3648, height=2736, plans count=3, timestamp=492152060372000
plan-0-Y, bufferSize=9980928, pixelStride=1, rowStride=3648
plan-1-U, bufferSize=4990463, pixelStride=2, rowStride=3648
plan-2-V, bufferSize=4990463, pixelStride=2, rowStride=3648

->NV21 or NV 12

// ImageFormat.YUV_420_888
// Multi-plane Android YUV 420 format
//This format is a generic YCbCr format, capable of describing any 4:2:0 chroma-subsampled planar or semiplanar buffer (but not fully interleaved), with 8 bits per color sample.
//Images in this format are always represented by three separate buffers of data, one for each color plane. Additional information always accompanies the buffers, describing the row stride and the pixel stride for each plane.
//The order of planes in the array returned by Image#getPlanes() is guaranteed such that plane #0 is always Y, plane #1 is always U (Cb), and plane #2 is always V (Cr).
//The Y-plane is guaranteed not to be interleaved with the U/V planes (in particular, pixel stride is always 1 in yPlane.getPixelStride()).
//The U/V planes are guaranteed to have the same row stride and pixel stride (in particular, uPlane.getRowStride() == vPlane.getRowStride() and uPlane.getPixelStride() == vPlane.getPixelStride(); ).
//For example, the android.media.Image object can provide data in this format from a android.hardware.camera2.CameraDevice through a android.media.ImageReader object.

// PixelStride
The distance between adjacent pixel samples, in bytes.
This is the distance between two consecutive pixel values in a row of pixels. It may be larger than the size of a single pixel to account for interleaved image data or padded formats. Note that pixel stride is undefined for some formats such as RAW_PRIVATE, and calling getPixelStride on images of these formats will cause an UnsupportedOperationException being thrown. For formats where pixel stride is well defined, the pixel stride is always greater than 0.

// RowStride
The row stride for this color plane, in bytes.
This is the distance between the start of two consecutive rows of pixels in the image. Note that row stried is undefined for some formats such as RAW_PRIVATE, and calling getRowStride on images of these formats will cause an UnsupportedOperationException being thrown. For formats where row stride is well defined, the row stride is always greater than 0.
```

# 参考

- 图像的基本表示方法，单通道与多通道 (https://zhuanlan.zhihu.com/p/129314401)
- 行宽（line/row stride）说明（https://blog.csdn.net/bjrxyz/article/details/52690661）
- 图片宽、高、步长、像素深度形象展示 (https://blog.csdn.net/guo_lei_lamant/article/details/106572915)
- A programmer's view on digital images: the essentials (https://www.collabora.com/news-and-blog/blog/2016/02/16/a-programmers-view-on-digital-images-the-essentials/)
- YUV color formats(https://envytools.readthedocs.io/en/latest/hw/memory/g80-surface.html#yuv-color-formats)
- 后续：数字图像处理系列、计算机视觉基础、多媒体处理、计算机视觉处理、计算机图形学
- MATLAB图像与视频处理
- FFmpeg从入门到精通+音视频开发进阶指南 2本 FFMPEG视音频编解码基础书籍
- [零基础入门音视频开发]https://www.jianshu.com/nb/22515965
- [详解 YUV 格式](https://www.jianshu.com/p/358bf8b7eacc)
- [Camera2不能设置NV21](https://juejin.im/post/6844904064568803341)
- [Android:  YUV_420_888编码Image转换为I420和NV21格式byte数组](https://www.polarxiong.com/archives/Android-YUV_420_888%E7%BC%96%E7%A0%81Image%E8%BD%AC%E6%8D%A2%E4%B8%BAI420%E5%92%8CNV21%E6%A0%BC%E5%BC%8Fbyte%E6%95%B0%E7%BB%84.html)
