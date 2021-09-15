# Android RIL 介绍

## 简介
    Android 作为通用智能设备平台，其首要功能就是通话、短信、数据等通信功能。这也是移动OS与桌面windows的最大区别。本文会介绍Android Ril框架结构以及关键业务流程。其次本文源代码来自于aosp 11.0.0_r1分支。

    撰写本文目的也是市面上对于Telephony&RIL介绍的内容过于陈旧，以本文对目前RIL架构进行梳理，方便后续理解HIDL，其作为Android Project Treble中的重要内容对于解决Android碎片化有着十分重要的作用

## RIL框架结构
    RIL(Radio Interface Layer, 无线电通信接口层)在Android中的实现代码可以大致分为两大部分：

        1.Framework框架层中Java相关程序，简称RILJ
        2.HAL层中的c/c++程序，简称RILC

如图：

![ril-arch](https://github.com/GaryTsuiC/GaryTsui.github.io/blob/main/Android/RIL/pic/ril-arch.png)

    RILC包括三部分RILD、Libril、Reference-ril，核心代码主要分布在rild、libril、reference-ril三个目录下。编译后会生成libril.so、libreference-ril.so动态链接库文件和rild可执行文件。

    RILC模块主要实现LibRIL和Reference-RIL之间的相互调用，完成RIL请求转为换成AT命令和AT命令下发到RIL的交互。

## 1.RILJ 

    TBD

## 2.RILC

### 2.1 RILC 代码目录

如图：

![ril-arch](https://github.com/GaryTsuiC/GaryTsui.github.io/blob/main/Android/RIL/pic/rild-list.png)

    按照模块分别编译，可以得到libril.so, libreference-ril.so和rild的结果文件。各自make以及其产物：

![ril-arch](https://github.com/GaryTsuiC/GaryTsui.github.io/blob/main/Android/RIL/pic/make-libril.png)

![ril-arch](https://github.com/GaryTsuiC/GaryTsui.github.io/blob/main/Android/RIL/pic/make-libril.png)

![ril-arch](https://github.com/GaryTsuiC/GaryTsui.github.io/blob/main/Android/RIL/pic/make-reference-ril.png)

![ril-arch](https://github.com/GaryTsuiC/GaryTsui.github.io/blob/main/Android/RIL/pic/make-ril-out.png)

### 2.1 RILC运行机制

![ril-arch](https://github.com/GaryTsuiC/GaryTsui.github.io/blob/main/Android/RIL/pic/rilc-arch.png)

### 2.2 RILD启动

![ril-arch](https://github.com/GaryTsuiC/GaryTsui.github.io/blob/main/Android/RIL/pic/rild-init.png)

    入口函数主要完成了3个作用：
    1、开启EventLoop循环，完成RIL与RILJ层数据交互（HIDL）
    2、打开动态库reference并构建ReaderLoop循环，完成RIL与Modem层数据交互（通过AT）
    3、注册reference的回调函数。