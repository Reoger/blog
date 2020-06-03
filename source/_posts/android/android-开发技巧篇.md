---
title: android-开发技巧表
date: 2018-01-17 19:50:52
categories: 实习日记
tags: android,debug
---

# 参考链接
* [超详细的dubug教程](http://blog.csdn.net/my_truelove/article/details/52240558)
* [Android Studio 调试技巧](http://www.10tiao.com/html/169/201612/2650821802/1.html)
* [Android中开发需要的高效助推的命令总结](https://www.jianshu.com/p/a3459ffe4a95?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
* [Android adb shell am 命令介绍](http://blog.csdn.net/soslinken/article/details/50245865)
* [adb shell dumpsys 命令用法](https://www.jianshu.com/p/d859480613a4)
* [AMS之dumpsys篇](http://gityuan.com/2017/07/04/ams_dumpsys/)

# debug 技巧
关于android studio的debug技巧，前面的两篇参考文章已经写的特别好了，也没必要再写一次了，主要就记录一下debug的关键概念和快捷键。至于如何debug，就请参考前面两篇文章。

## 工具栏介绍

![](http://ovec6nnof.bkt.clouddn.com/too.png)

上图从左往右看，名称和作用如下表所示。

|名称|作用|备注|快捷键|
|--|--|---|---|
|show Execution Point|定位到当前正在调试的位置|无备注|alt+F10|
|step over|单步跳过|一步一步执行，遇到方法会直接执行方法，然后进入下一步，不会进入方法内容|F8|
|step into|单步跳入|单步向下执行，如果当前是自定义方法，hi进入方法内部，系统方法则不进入方法内部|F7|
|Force step into|强制单步跳入|与单步跳入不同的是，不管什么方法他都会进入|Alt + Shfit + F7|
|step out|单步跳出|与单步跳入相对，表示从方法体中跳出，回到进入方法的位置，以继续断点|shfit + F8|
|run to cursor |执行到光标处|直接从当前位置运行到光标处，但是能被中间的断点拦截。|alt + F9|
|Evalyate Exoression|计算表达式|支持在点点过程中，通过直接赋值或者表达式方式，修改任意表俩个的值。|alt+F8|

再加来几个快捷键。

|快捷键|功能|说明|
|--|--|--|
|右击断点|为断点设置执行条件，或打印信息|只能针对本断点生效|
|alt + 单击| 查看断点时变量的值|无|
|ctrl+alt+F8|为断点添加执行条件，或打印信息|这个可以对所有的断点生效|


# 工具篇
## uiautorviewer
可以通过这个工具快速定位到UI控件的ID，并通过ID快速找到相应的逻辑。


## DDMS
可以利用ddms这个工具，实现截屏，查看线程和堆信息，日志信息，进程，广播状态信息，模拟来电，呼叫和短信等功能。
具体使用方法参考这里<https://developer.android.com/studio/profile/monitor.html>


# 命令篇
## adb shell am 
am 就是activity manager的简称，可以用于启动activity、打开或关闭进程、发送广播等操作。
关于具体的命令，可以参考这里<http://blog.csdn.net/soslinken/article/details/50245865>.
然后，这里就记录常用的``adb shell am``命令。
[注：这里默认省略了``adb shell``]

|命令|作用|备注|示例|
|---|---|---|---|
|``am start -n <package name>/<ativity name> ``|启动acivity|-n 表示以组件式启动，还可以|``am start -a android.settings.INPUT_METHOD_SETTINGS``//使用Action方式打开系统设置-输入法设置|
|``am start -a -n --es extra "hello" --ei pid 10  <package name>/<ativity name>``|待参数的启动activity|--es 表示带string，--ei 表示整型数据,都是以键值对的形式|`` am start -a -n --es extra "hello" --ei pid 10 com.reoger.app/com.example.cm.myapplication.NextActivity``|
|``am  broadcast -a <action>``|启动广播|还可以通过``--user``指定用户发送广播|``com.android.broadcast.test``|
|``am  broadcast  -a  <action> --es <key>  <value> ``|带信息的发送广播|--es表示字符串，还有--ez（布尔值）等多种类型数据，都是以键值对的形式|`` am  broadcast  -a  com.android.broadcast.test --es adb_extra "hello"``|
|``am startservice <package name>/<service name>``|启动服务|可以通过``--user<USER_ID>``指定启动的用户|``am startservice com.reoger.app/com.example.cm.myapplication.MyService``|
|``am force-stop <package name>``|关闭指定包名的应用程序|无|``am force-stop com.reoger.app ``|
|``am kill <package name>``| 杀死与应该程序包想关联的所有进程，但只会杀死安全进程|可以通过``--user <USER_ID>``指定用户 |``am kill com.reoger.app``|
|``am kill -all``|杀死全部的后台进程|无|``am kill -all``|

详情参考这里：<http://blog.csdn.net/soslinken/article/details/50245865>

## adb shell pm
pm即是 package manager的简称，可以用于安装应用、查询应用信息、系统权限、控制应用。

|命令|作用|备注|示例|
|---|---|---|---|
|``pm list packages [options] [fileter]``|打印所有已经安装的应用的包名|options 常用的有``-3`` 表示只显示第三方应用的包名，filter表示按名字筛选|``pm list packages -e `` 显示可用的应用和包名|
|``pm list permission [options] [group]``|打印权限|``-g``表示按组列出，``-s``表示简短打印|``pm list permission-groups``  打印所有已知的权限组|
|``grant <package_name> <permission>``|授予应用权限|必须android 6.0及以上的设备|``grant com.reoger.app android.permission.WRITE_EXTERNAL_STORAGE``|
|``revoke <package_name> <permission>``|撤销应用权限|必须android 6.0及以上的设备|``revoke com.reoger.app android.permission.WRITE_EXTERNAL_STORAGE``|
|``pm clear <package name>``|清除应用数据|无|``pm clear com.reoger.app``|
|``pm enable <package or component>``|使得packaege或componet可用|只针对第系统应用|``pm enable com.reoger.app``|
|``pm hide <package or component>``|隐藏package或componet|被隐藏应用在管理中变得不可见，桌面图标也会消失|``pm hide com.reoger.app``|
|``pm unhide <package or component>``|取消隐藏package或componet|桌面图标需要重新添加|``pm unhide com.reoger.app``|

详情请参考这里：<https://www.cnblogs.com/JianXu/p/5380882.html>

## adb shell dumysys

|命令|作用|
|----|---|
|``dumpsys cpuinfo ``|查看CPU信息|
|``dumpsys activity``|查看一大堆信息，包括activity、broadcasts、providers、permissions等等信息|
|``dumpsys activity top``| 获取当前android系统中与用户交互的activity的详细信息|
|``dumpsys activity activities ``|显示当前所有运行的任务栈,可以与管道``- grep XXX``结合使用，用于筛选我们需要的任务栈|
|``dumpsys activity meminfo <package name>``| 显示应用内存使用的情况|
|``dumpsys activity package <package name>``|显示apk的信息|


其中的``dumpsys activity [options] [WHAT]``参数可选如下：

|option|含义|
|------|----|
|-a |包括所有可用server状态|
|-c |包括client状态，即app端情况|
|-p package|限定输出指定包名|

其中WHAT参数可选如下：

|WHAT|	解释	|对应源码|
|----|-------|--------|
|a[ctivities]|	activity状态|	dumpActivitiesLocked()|
|b[roadcasts] [PACKAGE_NAME]|	broadcast状态	|dumpBroadcastsLocked()|
|s[ervices] [COMP_SPEC …]	|service状态	| newServiceDumperLocked().dumpLocked |
|prov[iders] [COMP_SPEC …]	|content provider状态	| dumpProvidersLocked()|
|p[rocesses] [PACKAGE_NAME]|	进程状态|	dumpProcessesLocked()|
|o[om]	|内存管理	|dumpOomLocked()|
|i[ntents] [PACKAGE_NAME]|	pending intent状态|	dumpPendingIntentsLocked()|
|r[ecents]|	最近activity|	dumpRecentsLocked()|
|perm[issions]	|URI授权情况|	dumpPermissionsLocked()|
|all|	所有activities信息	|dumpActivity()|
|top|	顶部activity信息	|dumpActivity()|
|package|	package相关信息	|dump()|

最后，附上我用于测试adb 命令的demo：<https://github.com/Reoger/adbTest>