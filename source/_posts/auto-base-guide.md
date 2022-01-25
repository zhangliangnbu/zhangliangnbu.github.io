---
title: 车机开发基础
date: 2021-04-07 12:27:17
tags:
- auto
- can
categories:
- 车机开发
---

介绍实际车机开发中会用到的基础知识和概念，持续更新。

# 常见术语

| 简写      | 全称                                      | 描述                                                         |
| --------- | ----------------------------------------- | ------------------------------------------------------------ |
| BAT       | Battery                                   | 蓄电池，BAT+一般表示电池有电状态                             |
| ACC       | Accessory                                 | 附件档，ACC-ON时可为部分车载附属设备供电，空调等除外。       |
| IGN       | Ignition                                  | 点火档，IGN-ON时，可为所有设备供电，包括空调                 |
| IVI       | In-Vehicle Infotainment                   | 车载娱乐信息系统，车机SoC是其中的核心部分                    |
| HVAC      | Heating, Ventilation and Air Conditioning | 供热通风与空气调节，即空调                                   |
| DVR       | Drive Video Recorder                      | 行车记录仪                                                   |
| HUD       | Head-Up Display                           | 抬头显示，在驾驶位正前方，一般与前车窗重合                   |
| M-CAN     | Multimedia CAN                            | 多媒体CAN协议                                                |
| V-CAN     | Vehicle CAN                               | 车身CAN协议                                                  |
| RVC / RVM | Rear View Camera / Monitor                | 后方车辆摄像头或监控，一般称为倒车影像，包括实时路况显示和雷达信息显示（距离） |
| AVM       | Around View Monitor                       | 360度全景影像系统                                            |

<!-- more -->

说明：

- 档位与供电关系。时间先后：BAT+->ACC-ON->IGN-ON。首先蓄电池处于供电状态，然后可以开启ACC挡，为IVI供电等，最后开启IGN挡，可以开空调，并开动车辆。
- 倒车影像。通过远红外摄像获取车后路况图像，一般有有辅助线，是否一定是通过雷达系统获取距离信息来绘制的？
- 倒车雷达。通过超声波遇障碍物返回机制。可知晓车后障碍物距离信息，通过显示屏或蜂鸣器提示。一般由超声波传感器、控制器和显示器或蜂鸣器等组成。

# CAN相关

> 控制器局域网 (Controller Area Network，简称CAN或者CAN bus) 是一种功能丰富的[车用总线](https://zh.wikipedia.org/w/index.php?title=车用总线&action=edit&redlink=1)标准。

CAN相关详细介绍请见[维基百科](https://zh.wikipedia.org/wiki/%E6%8E%A7%E5%88%B6%E5%99%A8%E5%8D%80%E5%9F%9F%E7%B6%B2%E8%B7%AF)或百度百科。

**SoC借助MCU通过CAN与ECU的通信**

![auto-soc-can-ecu](/images/auto-soc-can-ecu.svg)

SoC与MCU之间的交互可以使用目前比较流行的Protocol Buffer进行通信。

**CAN信号特点**

- SoC发送给ECU的设置类消息，一般为事件性的，单次触发，对应SoC的设置接口。
- ECU发送给SoC的状态类消息，一般为周期性的，周期发送，对应SoC的回调接口。虽然为周期性发送，但MCU一般会处理，只有信号值改变时，才触发SoC的回调，因此对于SoC来说仅仅是有变更时才进行一次回调。
- ECU一般很少提供给SoC获取接口，需要MCU或SoC自己缓存最新的数据，开发时需注意。





# 参考

- [Protocol Buffers](https://developers.google.com/protocol-buffers)
- [汽车上的电源模块](https://www.infineon-autoeco.com/bbs/detail/9779#)
