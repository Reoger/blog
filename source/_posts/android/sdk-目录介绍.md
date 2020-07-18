---
title: sdk 目录介绍
date: 2018-01-11 21:19:46
categories: android
tags: [android,sdk]
---

# sdk 目录下各目录详解

sdk目录下的目录如下：
![dir.png](http://upload-images.jianshu.io/upload_images/2178834-947c10d5de3b8a99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面针对一些工具进行介绍：

1. add-ons
这里保存着附加库，第三方公司为Android平台开发的附加功能。例如googleMaps
2. build tools（重要）
这里保存着一些Android开发常用的工具，例如adb、aidl等等。下面是我的tools目录下的工具包。
![platforms.png](http://upload-images.jianshu.io/upload_images/2178834-56578752af0e735e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* android
这个其实就是Android SDK Manager，用于管理SDK的下载、更新和删除。
* ddms
这个工具集成了Dalvik（为Android 平台定制的虚拟机（VM）），能够让你在模拟器或者设备上管理进程并协助调试。你可以使用它杀死进程，选择某个特定的进程来调试，产生跟踪数据，观察堆（heap）和线程信息，截取模拟器或设备的屏幕画面，还有更多的功能。
* draw9patch
该工具允许你使用所见即所得（WYSIWYG）的编辑器轻松地创建NinePatch图形。它也可以预览经过拉伸的图像，高亮显示内容区域。
* hierarchyviewer
Hierarchyview和UiAutomatorviewer作用类似，都是用于查看当前界面控件，但Hierarchyviewer能显示的属性更为全面(设备需要root，调用的API权限比UiAutomator更高)
* jobb
这个工具可以用来加密、解密apk
* lint
 是代码扫描分析工具，它可以帮助我们发现代码结构/质量问题，同时提供一些解决方案，而且这个过程不需要我们手写测试用例。
* mksdcard
帮助你创建磁盘映像（disk image），你可以在模拟器环境下使用磁盘映像来模拟外部存储卡（例如SD 卡）。
* monitor
Monitor工具具有强大的监控功能，他提供良好界面和众多监控功能，包括devicse监控、update Heap、
* monkeyrunner
的压力测试应用，模拟用户随机按键。
* traceview
这个工具可以将你的Android 应用程序产生的跟踪日志（trace log）转换为图形化的分析视图。
* uiautomatorviewer
用于进行UI测试。
3. docs
这里面是Android SDK API参考文档。
4. emulator
这里存放的是一些安卓模拟器
5. extras
扩展开发包，在此文件夹下保存着额外的usb驱动、intel硬件加速。
6. patcher
增量更新，用于更新记录。
7. platforms
是每个平台SDK真正的文件，存放不同版本的Android系统。里面会根据APILevel划分的SDK版本，本人下载的是Android 23，进入之后的目录如下：
![platforms-tools.png](http://upload-images.jianshu.io/upload_images/2178834-c413d794f93d0106.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
针对上面的目录简单介绍如下；
* data
保存着一些系统资源。
* optional
存放一些可选择的支持库，譬如http client就可以依赖本目录下的``org.apache.http.legacy``jar包。
* skins
存这Android模拟器的皮肤。
* templates
工程的默认模板。
8. platforms-tools（重要）
版本通用的工具，比如adb、aidl、dexdump等等。目录结果如下：
![tools.png](http://upload-images.jianshu.io/upload_images/2178834-362d048fe17c7107.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* adb 
Android Debug Bridge，Android调试桥接器，简称ADB，是用于管理模拟器或真机状态的万能工具，通俗一点讲adb就是pc和移动设备通信的桥梁，它采用了c/s模型，包括三个部分：客户端部分、服务端部分和守护进程部分。
* dmtracedump
是将整个调用过程和时间分析结合，以函数调用图的形式表现出来。
* etc1tool
    etc1tool是一个命令行工具，可以将PNG图像压缩为etc1标准，并且可以进行解压缩。
* fastboot
    Fastboot是Android快速升级的一种方法，Fastboot的协议fastboot_protocol.txt在源码目录./bootable/bootloader/legacy下可以找到。
    Fastboot客户端是作为Android系统编译的一部分，编译后位于./out/host/linux-x86/bin/fastboot目录下。
    Fastboot命令实例：``sudo fastboot flash kernel path-to-kernel/uImage``
    烧写rootfs类似：``sudo fastboot flash system path-to-system/system.img``
* hprof-conv
hprof-conv工具可以将Android SDK工具生成的HPROF文件生成一个标准的格式,主要用于性能测试。
* make_f2fs
转化成f2fs文件格式的文件。
* mke2fs
mke2fs命令被用于创建磁盘分区上的“etc2/etc3”文件系统。
* sqlite3
在adb命令模式下，查看数据库的信息。
9. skins
Android模拟器的皮肤。
10. sources
各版本的sdk源码。
11. system-images
模拟器映像文件。从android-14开始将模拟器映像文件整理在这里(原来放在platforms下)
12. tools
各个版本自带工具，包含重要的工具DDMS
13. AVD Manager
Android手机模拟配置工具，用于配置模拟器。
14. SDK maager
SDK管理器，用于SDK的更新、下载、删除。

参考链接：

* [官方参考文章](https://developer.android.com/studio/command-line/jobb.html)
* [Android系统platform-tools包详解](http://blog.csdn.net/apple_hsp/article/details/51459930)
* [Android SDK目录结构](http://blog.csdn.net/itluochen/article/details/52688935)
* [Android SDK中的强大工具-Monitor](https://www.jianshu.com/p/3f8828b4a05a)