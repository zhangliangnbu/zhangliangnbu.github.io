---
title: Android 10系统编译和烧录
date: 2021-12-15 15:10:20
tags:
- android
- android系统开发
categories:
- Android系统开发
---

# 说明

基于Ubuntu 20.04编译AOSP里的Android 10系统，并烧录进Pixel 2实机设备里。

> 烧录指将编译后的系统文件刷入实机或模拟器中

也适用Ubuntu18.4，不一致的地方会做说明，主要是python版本有差异。

应该说[AOSP官网](https://developer.android.google.cn/)对整个流程都有描述，但有些地方可能说得不够清楚明白，为避免不常编译系统的开发者少走弯路，故有此总结。另外需要说明的是，优先看官网文档，官网有不清楚地方再看其他资料比如本文。

AOSP官网有两个，国际版本和国内版本，内容一致，但国内版本速度较快些。

- 国际版本：https://source.android.com
- 国内版本：https://source.android.google.cn

# 搭建编译环境

**硬件环境说明**

[官网构建环境章节](https://source.android.google.cn/setup/build/requirements)上说了编译Android 10必须保证机器的内存在16G以上，实践过程中发现低于16G内存会报各种错误。网上有一些方法可以在低于16G内存的机器上通过编译，尝试了其中的一些，依旧不能完成编译，故最好保证机器内存在16G以上，内存越大编译速度越快，血的教训，注意。

**开发平台版本说明**

- Ubuntu 20.04
- Python 3.6+
- Android 10 系统信息：QP1A.190711.019，android-10.0.0_r1	Android10，Pixel 3a XL, Pixel 3a, Pixel 3 XL, Pixel 3, Pixel 2 XL, Pixel 2, Pixel XL, Pixel	2019-09-05
- 其他

<aside> 💡 Ubuntu 20.4 need Python 3.6+, Ubuntu 18.4 need Python 2.7+

**设置软件环境**

```bash
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 libncurses5-dev lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig
```

**配置Git**

```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

**检查Python**

```bash
python --version
# if no python, then set a link
# use 'whereis python' to find where python locate.
sudo ln -s /usr/bin/python3 /usr/bin/python
```

**安装Repo工具**

```bash
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

**将REPO_URL改清华的源**

我使用的是清华源，使用其他源也可以

方法一：配置环境变量（推荐）

```bash
vi ~/.bashrc
# export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
source ~/.bashrc
```

方法二：直接更改repo文件配置

```bash
# 找到repo
whereis repo
# 替换
sudo vim /usr/bin/repo
# REPO_URL = 'https://gerrit.googlesource.com/git-repo'
# 改为：
# REPO_URL = 'https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

# 下载系统源代码

Android 10源代码也有很多小的版本，我们这里只下载最初的版本`android-10.0.0_r1`，官网可查看版本，见[代号、标记和 Build 号](https://source.android.google.cn/setup/start/build-numbers)，其中我们需要的是标记（Tags）。

```bash
mkdir aosp10
cd aosp10
# 注意这里的连接要改为清华镜像的 -b后面为分支版本，如可改为 android-10.0.0_r47 
# repo init -u https://android.googlesource.com/platform/manifest
# repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-9.0.0_r1 --depth=1
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-10.0.0_r1 --depth=1
# 开始下载 j后面的数字是线程数，推荐写4
repo sync -c -j4
```

<aside> 💡 另外一种下载源码的方法是 先用wget命令下载，然后解压、同步。

# 编译源代码

```bash
$ source build/envsetup.sh
$ lunch aosp_walleye-userdebug
$ m
...等待编译完成，没有报error就表示编译通过
...一般需要几个小时到十几个小时不等，我使用32G内存时，大约需要3~4个小时
```

其中，`lunch`后的参数`aosp_walleye-userdebug`为Pixel 2手机的设备Build编号，因为我后面会将编译后的系统刷入谷歌pixel 2手机中，故选择此Build编号。

设备Build编号选择说明见官网[设备Build编号](https://source.android.google.cn/setup/build/running#selecting-device-build)。

# 实机烧录

**下载并安装硬件驱动文件**

硬件烧录时需要，如果不烧录到硬件则不需要下载。

根据代号`qp1a.190711.019`查找Google和高通相关驱动，地址：[Pixel 2 binaries for Android 10.0.0 (QP1A.190711.019)](https://developers.google.cn/android/drivers#walleyeqp1a.190711.019)

下载完后解压放在aosp源码根目录，并安装：

```bash
sh extract-google_devices-walleye.sh
sh extract-qcom-walleye.sh
# 看完license输入 I ACCEPT 安装完成, 此时ls会看到vendor文件夹
```

**安装刷机工具**

安装和配置ADB、FASTBOOT工具。

安装：有两种方法安装工具，从SDK配置或直接命令行安装下载

```bash
$ sudo apt-get install android-tools-adb
$ sudo apt-get install android-tools-fastboot
```

配置：对于Ubuntu系统需要进行两项配置，一是添加当前用户到 plugdev 组中，二是配置udev 规则，详情请见[官网-在硬件设备上运行应用](https://developer.android.com/studio/run/device)

<aside> 💡 注意使用数据线时，需要有通信能力的，有的usb数据线仅仅只有充电功能，本人亲身经历过！

**刷机**

前提是手机已经解锁，我使用的Pixel 2手机已经解锁，使用以下命令进行烧录。

```bash
# 在lunch环境里
$ source build/envsetup.sh
$ lunch aosp_walleye-userdebug

# 进入BootLoader
$ adb reboot bootloader

# 烧录
$ fastboot flashall -w
```

# 参考

- [官网-AOSP编译](https://source.android.google.cn/setup/start?hl=zh-cn)：起始页
- [官网-代号、标记和 Build 号](https://source.android.google.cn/setup/start/build-numbers)：根据标记下载源码版本，根据代号下载实机设备驱动。
- [官网-在硬件设备上运行应用](https://developer.android.com/studio/run/device)：设置Linux系统以检测USB设备
- [官网-刷写设备](https://source.android.com/setup/build/running?hl=zh-cn#flashing-a-device)
- [清华源-Android 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)：清华源下载AOSP代码
- [清华源-Git Repo 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/)
- [三方博客-编译AOSP并刷入Pixel设备](https://www.cnblogs.com/ciml/p/13714036.html)：参考烧录章节
- [三方博客-Android 9.0编译](https://juejin.cn/post/6844904104368537613)
- [三方博客-ubuntu20.4编译AOSP安卓源码（AndroidP android-9.0.0_r9）](https://blog.csdn.net/mvp_Dawn/article/details/107624203)
- [三方博客-AOSP 编译和烧写（实机烧录）](http://blog.hanschen.site/2019/09/12/aosp_compile_and_flash/)
- [三方博客-Android开发环境配置0：adb和fastboot](https://debugtalk.com/post/android-development-environment-adb-and-fastboot/)

