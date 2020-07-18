---
title: android中的类加载
date: 2018-03-17 21:21:33
categories: android
tags: android,classLoader
---
主要介绍几个知识点：

# 参考资料
* <https://www.jianshu.com/p/3afa47e9112e>
* <...>

# java中常用的``classLoader``
说到android中的``classLoader``，就不能不先说说java中的``classLoader``是什么。
所谓classLoader就是负责将编译好的class文件加载到指定位置的实现类。具体来说，我们编写java代码时，需要将其编译成.class文件，最终运行时就需要将这些class文件加载到内存才能运行，而加载这些class文件的方法就可以成为classLoader。
在java中常用的classLoader有以下三种：
## Bootstrap Classloder
 这个类加载使用C++语言实现，是虚拟机自身的一部分。他是三个类加载器中最顶层的加载类，主要负责加载%JAVA_HOME%/lib下的核心类或者jar加载到内存中。（值得注意的是，Bootstrao Loader被设计成只能加载包名为java、javax、sun等开头的类.）
## Extention Classloder
扩展类的加载器，由java语言实现，主要负载加载``%JAVA_HOME%/lib/ext``目录下的类库，或者是系统指定的类库。
## AppClassloader
 主要负责加载系统类路径或者指定路径下的类库。
以上三种是java中定义的ClassLoader。

# 下面介绍android中常用的类加载器。
## classLoader
所有的classLoader的基类，他是一个抽象类，所有的``classLoader``最终都会继承自他，我们如果需要自定义classLoader也需要直接或者间接的继承他，并实现其中的``findClass``方法，并通过``defineClass``创建一个类实例。自定义类加载的示例如下：
```
 class NetworkClassLoader extends ClassLoader {
         String host;
          int port;
 
          public Class findClass(String name) {
              byte[] b = loadClassData(name);
              return defineClass(name, b, 0, b.length);
          }
 
          private byte[] loadClassData(String name) {
              // load the class data from the connection
              //省略
          }
      }
```

## BaseDexClassLoader
BaseDexClassLoader继承自ClassLoader，只是对其进行了进一步的封装，并没有实现，他有两个直接的子类``PathClassLoader``和``DexClassLoader``。
简单介绍一下他的构造函数：
```
    public BaseDexClassLoader(String dexPath, File optimizedDirectory, String librarySearchPath, ClassLoader parent) {
      //dexPath 代表目标类所在的apk、DEX或者JAR文件的路径
      //optimizedDirectory用于指定解压出来的dex文件存放的路径
      //librarySearchPath用于指定类中所使用的C/C++库存放的路径
      //parent 用于指定该加载器的父加载器，一般为当前执行类的加载器
    }
```
关于BaseDexClassLoader还有一点要补充的是，由于dex文件被包含在APK或者jar文件中，因此在加载目标类之前需要先从APK或者jar文件中加压处dex文件，optimizedDirectory即为指定解压出来的dex文件存放的路径，这也是对apk中dex根据平台ODEX优化的过程。

## DexClassLoader
DexClassLoader用于加载包含dex的JAR或者APK文件，但是他不能加载jar或者apk文件。他最终加载的都是dex文件，虽然他是从.jar或者.zip，.apk等结尾的文件中加载，但是他们最终都会生成一个对应的dex文件，他操作的还是dex文件。DexClassLoader继承自BaseDexClassLoader，原理如下：
```
public class DexClassLoader extends BaseDexClassLoader {
    public DexClassLoader(String dexPath, String optimizedDirectory,
            String libraryPath, ClassLoader parent) {
        super(dexPath, new File(optimizedDirectory), libraryPath, parent);
    }
}
```
由代码不能看出，DexClassLoader只是简单的对BaseDexClassLoader进行了一层封装，并且指定了``optimizedDirectory``的路径为一个新的文件路径，DexClassLoader通过指定自己的optimizedDirectory，所以它可以加载外部的dex，因为这个dex会被复制到内部路径的optimizedDirectory。
**所以，DexClassLoader一般用来作为动态加载的加载器。**

## PathClassLoader
PathClassLoader也是继承自BaseDexClassLoader，他主要用于加载apk，一般应用于加载android的系统类和app应用的类。他的实现如下：
```
public class PathClassLoader extends BaseDexClassLoader {
    public PathClassLoader(String dexPath, ClassLoader parent) {
        super(dexPath, null, null, parent);
    }

    public PathClassLoader(String dexPath, String libraryPath,
            ClassLoader parent) {
        super(dexPath, null, libraryPath, parent);
    }
}
```
可以看到，PathClassLoader将optimizedDirectory置为null，而optimizedDirectory是用来指定apk或者jar解压出来的dex存放的位置，如果optimizedDirectory为null，则使用其默认路径/data/dalvik-cache，因此无法加载外部的apk的dex，只能加载内部的dex，这些大都是存在系统中已经安装过的apk里面的。

## URLClassLoader
URLClassLoader只能加载jar文件，但是dalvik虚拟机不能识别jar，所以在android中无法使用这个加载器。

## InMemoryDexClassLoader
InMemoryDexClassLoader也是继承自BeseDexClassLoader，是API26新增的加载器，用于加载内存中的dex文件。

## DelegateLastClassLoader
DelegateLastClassLoader继承自PathClassLoader，是API27新增的加载器，用于指定最后的查找策略，查找顺序如下：先判断自己是否加载此类，然后在判断此类的加载器是否加载过此类，最后委托给指定的父加载器。



# ``classloader``的双亲委托模型
classLoader双亲委托模型，当要加载某个类时，先判断自己是否有加载过此类，如果自己没有加在过此类的话，进而判断父类是否加载过，如果某个父类加在过这个类的话，就直接返回，不重复加载。如果一直到顶级父类都没有加载此类的话，这个加载任务就会分发下来，最后用当前类加载器去加载该类。

# 类加载机制实现热修复方案
通过使用dexClassloader动态修改加载dex的顺序即可达到热修复的目的。
