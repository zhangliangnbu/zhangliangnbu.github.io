---
title: Drawable和Mipmap资源的区别
date: 2019-01-14 11:25:12
tags:
- android
- mipmap
- drawable
categories:
- Android开发
---

# 区别

在apk安装的时候，`mipmap-xxx/`下的所有分辨率的图片都会保留，而`drawablexxx/`下的图片只有保留适配设备分辨率的图片，其余图片会丢弃掉，减少了APP安装大小。

> It’s best practice to place your app icons in `mipmap-` folders (not the `drawable-`folders) because they are used at resolutions different from the device’s current density。——from [Android Developers Blog](https://android-developers.googleblog.com/2014/10/getting-your-apps-ready-for-nexus-6-and.html)
>
> Using a mipmap as the source for your bitmap or drawable is a simple way to provide a quality image and various image scales, which can be particularly useful if you expect your image to be scaled during an animation. ——from [android-4.3](https://developer.android.com/about/versions/android-4.3)
>
>  if you are building different versions of your app for different densities, you should know about the "mipmap" resource directory.  This is exactly like "drawable" resources, except it does not participate in density stripping when creating the different apk targets. ——from [Google+ Message](https://plus.google.com/+DianneHackborn/posts/QTA9McYan1L)

# 用法

- `mipmapxxx/`下放置APP启动图标，以及需要高质量缩放动画的图片。
- 其他图片资源放在`drawablexxx/`下。

---

# 参考

- [Android Developers Blog](https://android-developers.googleblog.com/2014/10/getting-your-apps-ready-for-nexus-6-and.html)
- [android-4.3](https://developer.android.com/about/versions/android-4.3)
- [Google+ Message](https://plus.google.com/+DianneHackborn/posts/QTA9McYan1L)

----