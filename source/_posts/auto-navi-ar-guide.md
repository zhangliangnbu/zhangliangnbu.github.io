---
title: 车载AR导航开发
date: 2022-01-05 11:05:42
tags:
- auto
- navigation
categories:
- 车机开发
---

本文档主要介绍利用某导航SDK开发AR导航的流程（获取并传递图片）和注意事项，基础导航、图像的检测、AR融合算法皆由SDK提供。

<!-- more -->

# 车载AR导航介绍

车载AR导航机制一般为：首先利用摄像头将前方道路的真实场景实时捕捉下来，再结合汽车当前定位及传感器（陀螺仪、惯导）数据、地图导航信息以及场景AI识别，进行融合计算，然后生成虚拟的导航指引模型，并叠加到真实道路上，从而创建出更贴近驾驶者真实视野的导航画面。

车载AR导航系统需要实现的能力：

- 车道级导航能力。基础导航能力。
- 图像获取、检测与识别能力。
- AR融合算法能力。

# 前置条件

- 基础导航。AR导航是基于基础导航的，基础导航的基本功能必须是完善的、没有问题的。
- 基于DR的定位模式。DR定位模式可以提高AR导航的精度，使得AR引导提示得更加精准。
- 摄像头。支持YUV格式帧流的摄像头（基本都支持），预览帧率必须不小于20fps（这个要根据具体SDK的AR服务要求来，我所使用的SDK中请求图片的帧率约为20fps）。
- 传感器。三轴陀螺仪、三轴加速度，其丢帧率和稳定性要符合SDK提供方的要求。
- 车身信息。车速信息、倒车信号、方向盘信息、车灯信息、雨刮器信息等。

其他具体要求看SDK提供方的文档。



# 整体架构

架构图如下

![auto-navi-ar-dev](/images/auto-navi-ar-dev.svg)

说明：

1. HMI通过摄像头获取路况预览帧流（Camera数据）、通过传感器获取加速度和陀螺仪信息（IMU数据）、通过车机接口获取车辆相关信息如方向盘等（Viechle数据）等，传给AutoSDK中ARService进行处理。
2. HMI通过卫星天线、传感器等获取定位、加速度、陀螺仪等信息传给AutoSDK中定位服务进行处理。
3. 通过以上两步，SDK将AR图像替换底图并传递给HMI回调信息。

# 开发流程

主要流程为：查看摄像头参数、获取并缓存图片、传递图片、传递各种传感器和车身信息等。

本次开发中使用了罗技C270i USB摄像头，车机底层做了适配，因此可以直接用Android原生的Camera2相关接口开发。

## 查看摄像头参数

```java
CameraManager manager = (CameraManager) context.getSystemService(Context.CAMERA_SERVICE);
CameraCharacteristics characteristics = manager.getCameraCharacteristics(cameraId);
StreamConfigurationMap map = characteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);

//  摄像头支持的格式：
// 256 0x100 JPEG
// 34 0x22 PRIVATE
// 35 0x23 YUV_420_888 
int[] formats = map.getOutputFormats();

// YUV_420_888格式支持的像素尺寸:
// w/h= 864,480
// w/h= 640,360
// w/h= 800,448
// w/h= 800,600
// w/h= 640,480
// ...
Size[] sizes = map.getOutputSizes(ImageFormat.YUV_420_888);

// 支持的帧率：
// [[10, 30], [30, 30], [0, 0]]
Range<Integer>[] fpsRanges = characteristics.get(CameraCharacteristics.CONTROL_AE_AVAILABLE_TARGET_FPS_RANGES);
```

## 获取预览帧流

利用Camera2API获取摄像头预览帧流。

由于AR服务是通过回调请求图片，因此拍摄后的图片需要进行缓存，待AR回调时传给AR服务。可使用一个长度为2的队列进行缓存，要保证缓存数据为最新。队列长度不能太大，否则会造成较大的延时，也不能太小，否则会丢帧。

部分关键代码如下：

```java
// 设置ImageReader
// width和height可以设置成最终需要展示在车机屏幕上的宽高，摄像头会自动匹配宽高以及宽高比进行输出
// 格式为 ImageFormat.YUV_420_888
// 最大缓存图片数为2就足够了
mImageReader = ImageReader.newInstance(width, height, ImageFormat.YUV_420_888, 2);
mImageReader.setOnImageAvailableListener(mOnImageAvailableListener, mBackgroundHandler);

// 设置摄像头预览请求
mPreviewRequestBuilder = mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
// 用ImageReader来承接预览的帧
mPreviewRequestBuilder.addTarget(mImageReader.getSurface());

// 在CameraCaptureSession.StateCallback#onConfigured回调里设置如下
// 设置自动聚焦连续拍照
mPreviewRequestBuilder.set(CaptureRequest.CONTROL_AF_MODE, CaptureRequest.CONTROL_AF_MODE_CONTINUOUS_PICTURE);
// 设置拍摄帧率
mPreviewRequestBuilder.set(CaptureRequest.CONTROL_AE_TARGET_FPS_RANGE, new Range<>(30, 30));
// 请求重复拍照
mCaptureSession.setRepeatingRequest(mPreviewRequest, mCaptureCallback, mBackgroundHandler);

// 处理图片
// 在ImageReader#setOnImageAvailableListener回调里可以获取图片，通过如下代码可查看图片信息
Log.e(TAG, "ImageSaver-run: " + "format=" + mImage.getFormat() +
        ", width=" + mImage.getWidth() +
        ", height=" + mImage.getHeight() +
        ", plans count=" + mImage.getPlanes().length +
        ", timestamp=" + mImage.getTimestamp());
for (int i = 0; i < mImage.getPlanes().length; i ++) {
    Log.e(TAG, "ImageSaver-run: " +
            ", plan-" + i +
            ", bufferSize=" + mImage.getPlanes()[i].getBuffer().remaining() +
            ", pixelStride" + mImage.getPlanes()[i].getPixelStride() +
            ", rowStride" + mImage.getPlanes()[i].getRowStride()
    );
}
// 图片转换 将拍摄的帧转换为AR服务能够接收的帧
// 这里需要匹配SDK需要的格式，不同的SDK要求不一样，但大同小异，基本信息基本一致
private ImageInfo parseToImageInfo(Image image) {
    ImageInfo info = new ImageInfo();
    info.type = IMAGE_TYPE.IMAGE_TYPE_YUV_420_888;
    info.width = image.getWidth();
    info.height = image.getHeight();
    info.timeStamp = image.getTimestamp();
    info.data = new ArrayList<>();
    for (int i = 0; i < image.getPlanes().length; i++) {
        Image.Plane plane = image.getPlanes()[i];
        ByteBuffer buffer = plane.getBuffer();
        byte[] bytes = new byte[buffer.remaining()];
        buffer.get(bytes);
        ImageChannel ic = new ImageChannel(bytes, bytes.length, plane.getRowStride(), plane.getPixelStride());
        info.data.add(ic);
    }
    return info;
}

```

## 根据SDK文档设置AR服务

- 获取AR服务
- 设置资源代理，主要是AR引导图片、配置文件等
- 设置AR服务需要的摄像头参数
- 设置AR引擎需要在view中显示的大小
- 在AR服务回调里传入图片
- 设置IMU信息。导航过程中实时更新。
- 设置车辆状态信息：雨刮、档位、转向灯、方向盘。导航过程中实时更新。
- 等等

具体代码就不贴了，不同SDK不一样，根据SDk文档来即可。
