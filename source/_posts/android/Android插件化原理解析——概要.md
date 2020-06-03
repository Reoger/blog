---
title: Android插件化原理解析——概要
date: 2018-08-07 16:44:55
categories: 转载
tags: android,classLoader
---

Android插件化原理解析——概要
==================

发表于 2016-01-28   |   63874次阅读

2015年是Android插件化技术突飞猛进的一年，随着业务的发展各大厂商都碰到了Android Native平台的瓶颈：

1.  从技术上讲，业务逻辑的复杂导致代码量急剧膨胀，各大厂商陆续出到65535方法数的天花板；同时，运营为王的时代对于模块热更新提出了更高的要求。
2.  在业务层面上，功能模块的解耦以及维护团队的分离也是大势所趋；各个团队维护着同一个App的不同模块，如果每个模块升级新功能都需要对整个app进行升级，那么发布流程不仅复杂而且效率低下；在讲究小步快跑和持续迭代的移动互联网必将遭到淘汰。

H5和Hybird可以解决这些问题，但是始终比不上native的用户体验；于是，国外的FaceBook推出了`react-native`；而国内各大厂商几乎都选择纯native的插件化技术。可以说，Android的未来必将是`react-native`和插件化的天下。

`react-native`资料很多，但是讲述插件化的却凤毛菱角；插件化技术听起来高深莫测，实际上要解决的就是两个问题：

1.  代码加载
2.  资源加载

代码加载
----

类的加载可以使用Java的`ClassLoader`机制，但是对于Android来说，并不是说类加载进来就可以用了，很多组件都是有“生命”的；因此对于这些有血有肉的类，必须给它们注入活力，也就是所谓的**组件生命周期管理**；

另外，如何管理加载进来的类也是一个问题。假设多个插件依赖了相同的类，是抽取公共依赖进行管理还是插件单独依赖？这就是**ClassLoader的管理问题**；

资源加载
----

资源加载方案大家使用的原理都差不多，都是用`AssetManager`的隐藏方法`addAssetPath`；但是，不同插件的资源如何管理？是公用一套资源还是插件独立资源？共用资源如何避免资源冲突？对于资源加载，有的方案共用一套资源并采用资源分段机制解决冲突（要么修改`aapt`要么添加编译插件）；有的方案选择独立资源，不同插件管理自己的资源。

目前国内开源的较成熟的插件方案有[DL](https://github.com/singwhatiwanna/dynamic-load-apk)和[DroidPlugin](https://github.com/Qihoo360/DroidPlugin)；但是DL方案仅仅对Frameworl的表层做了处理，严重依赖`that`语法，编写插件代码和主程序代码需单独区分；而DroidPlugin通过Hook增强了Framework层的很多系统服务，开发插件就跟开发独立app差不多；就拿Activity生命周期的管理来说，DL的代理方式就像是牵线木偶，插件只不过是操纵傀儡而已；而DroidPlugin则是借尸还魂，插件是有血有肉的系统管理的真正组件；DroidPlugin Hook了系统几乎所有的Sevice，欺骗了大部分的系统API；掌握这个Hook过程需要掌握很多系统原理，因此学习DroidPlugin对于整个Android FrameWork层大有裨益。

接下来的一系列文章将以DroidPlugin为例讲解插件框架的原理，揭开插件化的神秘面纱；同时还能帮助深入理解Android Framewrok；主要内容如下：

*   [Hook机制之动态代理](http://weishu.me/2016/01/28/understand-plugin-framework-proxy-hook/)
*   [Hook机制之Binder Hook](http://weishu.me/2016/02/16/understand-plugin-framework-binder-hook/)
*   [Hook机制之AMS&PMS](http://weishu.me/2016/03/07/understand-plugin-framework-ams-pms-hook/)
*   [Activity生命周期管理](http://weishu.me/2016/03/21/understand-plugin-framework-activity-management/)
*   [插件加载机制](http://weishu.me/2016/04/05/understand-plugin-framework-classloader/)
*   [广播的管理方式](http://weishu.me/2016/04/12/understand-plugin-framework-receiver/)
*   [Service的插件化](http://weishu.me/2016/05/11/understand-plugin-framework-service/)
*   [ContentProvider的插件化](http://weishu.me/2016/07/12/understand-plugin-framework-content-provider/)
*   DroidPlugin插件通信机制
*   插件机制之资源管理
*   不同插件框架方案对比
*   插件化的未来

另外，对于每一章内容都会有详细的demo，具体见[understand-plugin-framework](https://github.com/tiann/understand-plugin-framework)；喜欢就点个关注吧～定期更新，敬请期待！

[#android](/tags/android/) [#droidplugin](/tags/droidplugin/) [#plugin framework](/tags/plugin-framework/)

[Android插件化原理解析——Hook机制之动态代理](/2016/01/28/understand-plugin-framework-proxy-hook/)

[你真的了解AsyncTask？](/2016/01/18/dive-into-asynctask/)


[37 日志](/archives)



[RSS](/atom.xml)

 [![Creative Commons](/images/cc-by-nc-sa.svg)](http://creativecommons.org/licenses/by-nc-sa/4.0) 
1.  [1. 代码加载](#代码加载)
2.  [2. 资源加载](#资源加载)
