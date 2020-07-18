---
title: gradle基础
date: 2018-01-12 14:50:13
categories: android
tags: [android,gradle]
---

*Gradle 是一个能通过插件形式自定义构建逻辑的优秀构建工具。*

以下的一些特性让我们选择了 Gradle：
* 使用领域专用语言（DSL）来描述和控制构建逻辑
* 构建文件基于 Groovy，并允许通过 DSL 来声明元素、使用代码操作 DSL 元素这样的混
合方式来自定义构建逻辑
* 内置了 Maven 和 Ivy 来进行依赖管理
* 相当灵活。允许使用最好的实现，但是不会强制实现的形式
* 插件可以提供它们的 DSL 和 API 来定义构建文件
* 优秀的 API 工具与 IDE 集成


这里主要记录几个比较常用，重要的点。
# android Task
* ``assemble`` 组合项目所有输出，他可以细分成``assemableDebug``和``assembleRelease``。
* ``check`` 执行所有检查，他拥有 ``lint`` 依赖
* ``connectedCheck`` 在一个连接的设备或者模拟器上执行检查，他们可以在所有连接的设备上并行执行检查,他拥有``connectedAndroidTest``依赖
* ``deviceCheck`` 通过APIs连接远程设备来执行检查，主要用于CI（持续集成）服务上。
* ``build`` 执行``assemle``和``check``的所有工作
* ``clean`` 清空项目的输出。
* ``installDebug`` 安装测试版的apk
* ``installRelease`` 安装发布版的apk
* ``uninstall`` 卸载安装，他包含三个动作``uninstallDebug``卸载测试版本， ``uninstallRelease``卸载发行版本和``uninstallDebugAndroidTest``卸载android测试。

# gradle 添加依赖
gradle的依赖主要分成本地包依赖和远程包依赖。
依赖的关键字主要包括：
1. ``compile`` 源代码(src/main/java)编译时的依赖
2. ``runtime`` 源代码(src/main/java)执行时的依赖
3. ``testCompile`` 测试代码(src/main/test)编译时的依赖
4. ``testRuntime`` 测试代码(src/main/test)执行时的依赖
5. ``archives`` 项目打包时的依赖
6. ``provided`` 只编译，并不将jar包导入到apk中。

依赖格式:
```
dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
}
```
其中的group表示组织，name表示要依赖的库，vesion表示版本。
我们可以简写成
```
dependencies {
    compile 'org.hibernate:hibernate-core:3.6.7.Final'
}
```
当然，不单只有对jar包的依赖，还可以有对文件的依赖，对项目的依赖，他们的写法依次如下：
```


dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar']) //将本地lib目录下所有的jar包进行依赖
    compile files('libs/picasso-2.4.0.jar')
    compile project(':libraries:lib1')  //对项目lib1添加依赖
}
```

##本地依赖
上面讲到的就是本地依赖的方式：
```
dependencies {
compile fileTree(dir: 'libs', include: ['*.jar'])
}
android {
...
}
```
## 外部依赖
首先需要添加远程仓库，android中至少添加一个远程仓库，例如使用开放的maven仓库。
```
repositories{
    mavenCentral()
}
```
然后，就对添加具体依赖。
```
dependencies{
    compile group: 'commons-collections',name: 'commons-collections', version: '3.2'
}
```

# 混淆
启动混淆非常简单，只需要在``build.gradle``中启动即可。
```
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```
其中的``minfyEnabled``默认是false，即不混淆，因为启动混淆编译速度会比较慢。
与混淆相关的还有一个`` shrinkResources ``属性，可以通过将其设置为``true``，设置资不打包没用的资源。

混淆通用规则：
```
#指定压缩级别
-optimizationpasses 5

#不跳过非公共的库的类成员
-dontskipnonpubliclibraryclassmembers

#混淆时采用的算法
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*

#把混淆类中的方法名也混淆了
-useuniqueclassmembernames

#优化时允许访问并修改有修饰符的类和类的成员 
-allowaccessmodification

#将文件来源重命名为“SourceFile”字符串
-renamesourcefileattribute SourceFile
#保留行号
-keepattributes SourceFile,LineNumberTable

#保持所有实现 Serializable 接口的类成员
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

#Fragment不需要在AndroidManifest.xml中注册，需要额外保护下
-keep public class * extends android.support.v4.app.Fragment
-keep public class * extends android.app.Fragment

# 保持测试相关的代码
-dontnote junit.framework.**
-dontnote junit.runner.**
-dontwarn android.test.**
-dontwarn android.support.test.**
-dontwarn org.junit.**
```

补充的混淆规则：
1. 第三方库混淆规则。这个比较常见，直接接入官方说明文档。

2. model实体类，典型在转化json的时候，必须保证model不被混淆，因此需加入--keep public class

3. JNI中调用的类以及方法不可被混淆

4. WebView中JavaScript调用的接口不混淆

5. AndroidMainfest、四大组件以及Application的子类不混淆

6. Parcelable的子类和Creator静态成员变量不混淆，否则会产生Android.os.BadParcelableException异常

7. Layout布局使用的View构造函数、android:onClick等。


# apk打包流程
![打包流程](https://upload-images.jianshu.io/upload_images/1441907-8a2c24bbb71c2cbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/536)
1. aapt打包res资源文件，生成R文件。
2. 处理aidl,生成java文件
3. 编译java 生成class文件
4. 通过dex命令，将class文件生成dex文件
5. apkbuilder阶段，将资源、dex、so合并成apk
6. 对apk进行签名
7. zipalign阶段,将签名后的apk进行对齐


# 参考连接

* [Gradle Android Plugin 中文手册](https://www.gitbook.com/book/chaosleong/gradle-for-android)

* [Gradle User Guide 中文版](https://dongchuan.gitbooks.io/gradle-user-guide-/)

* [Android Plugin for Gradle Release Notes](https://developer.android.com/studio/releases/gradle-plugin.html)

* [Android 打包过程](https://www.jianshu.com/p/7c288a17cda8)