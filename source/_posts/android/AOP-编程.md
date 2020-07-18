---
title: AOP 编程
date: 2018-01-18 20:22:12
categories: android 
tags: [android,aop]
---

# 参考资料
* [Android AOP面向切面编程详解](https://www.jianshu.com/p/9fb07b2596f7)
* [深入理解Android之AOP](http://blog.csdn.net/innost/article/details/49387395)
* <http://blog.csdn.net/autfish/article/details/51184405>
---

# AOP 基本思想
刚接触编程的时候，一般先接触的c语言这类的OOP（Procedure Oriented，即面向过程）语言，面相过程是一种以过程为中心的编程思想。这些都是以什么正在发生为主要目标进行编程，就是分析出解决问题所需要的步骤，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了。

然后，我们就会学到另一种更符合我们思维方式的一种思想，OOP（Object-Oriented Programming，即面向过程程序设计）语言，面向过程其实是最为实际的一种思考方式，就算是面向对象的方法也是含有面向过程的思想。可以说面向过程是一种基础的方法。它考虑的是实际地实现。一般的面向过程是从上往下步步求精，所以面向过程最重要的是模块化的思想方法。对比面向过程，面向对象的方法主要是把事物给对象化，对象包括属性与行为。当程序规模不是很大时，面向过程的方法还会体现出一种优势。因为程序的流程很清楚，按着模块与函数的方法可以很好的组织。
但是，在重复代码很多，需要重复利用的情况下，面相对象程序设计也显得很鸡肋，必要每次都主动去调用该方法，而且有特殊需求的时候还不能去需改写好的方法，因为这可能会造成其他其他调用该方法的地方出现问题。

这个时候AOP就出现了。AOP是Aspect Oriented Programming的缩写，即『面向切面编程』。它和我们平时接触到的OOP都是编程的不同思想，OOP，即『面向对象编程』，它提倡的是将功能模块化，对象化，而AOP的思想，则不太一样，它提倡的是针对同一类问题的统一处理，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

最后总结成一句话就是：
**在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。**


# 编程实践
讲了思想，我们通过具体的例子来进行说明。我们实现为方法自动添加log日志，不同于以前的主动调用，我们通过Aspect来实现AOP注入。

项目代码托管：<https://github.com/Reoger/AOPTest>

首先是配置：``app``下的``build.gradle``文件，配置类似如下代码：
```
apply plugin: 'com.android.application'
import org.aspectj.bridge.IMessage
import org.aspectj.bridge.MessageHandler
import org.aspectj.tools.ajc.Main

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.aspectj:aspectjtools:1.8.9'
        classpath 'org.aspectj:aspectjweaver:1.8.9'
    }
}
repositories {
    mavenCentral()
}
final def log = project.logger
final def variants = project.android.applicationVariants
variants.all { variant ->
    if (!variant.buildType.isDebuggable()) {
        log.debug("Skipping non-debuggable build type '${variant.buildType.name}'.")
        return;
    }

    JavaCompile javaCompile = variant.javaCompile
    javaCompile.doLast {
        String[] args = ["-showWeaveInfo",
                         "-1.8",
                         "-inpath", javaCompile.destinationDir.toString(),
                         "-aspectpath", javaCompile.classpath.asPath,
                         "-d", javaCompile.destinationDir.toString(),
                         "-classpath", javaCompile.classpath.asPath,
                         "-bootclasspath", project.android.bootClasspath.join(File.pathSeparator)]
        log.debug "ajc args: " + Arrays.toString(args)

        MessageHandler handler = new MessageHandler(true);
        new Main().run(args, handler);
        for (IMessage message : handler.getMessages(null, true)) {
            switch (message.getKind()) {
                case IMessage.ABORT:
                case IMessage.ERROR:
                case IMessage.FAIL:
                    log.error message.message, message.thrown
                    break;
                case IMessage.WARNING:
                    log.warn message.message, message.thrown
                    break;
                case IMessage.INFO:
                    log.info message.message, message.thrown
                    break;
                case IMessage.DEBUG:
                    log.debug message.message, message.thrown
                    break;
            }
        }
    }
}

android {
    compileSdkVersion 25
    buildToolsVersion '25.0.0'
    defaultConfig {
        applicationId "com.example.cm.aoptest"
        minSdkVersion 20
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    compile 'org.aspectj:aspectjrt:1.8.9'
    testCompile 'junit:junit:4.12'
}

```
 
然后就是实现我们的要实现的注入的log方法，``AspectLogTest.java``代码如下：
```
import android.util.Log;


import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;


/**
 * Created by CM on 2018/1/18.
 *
 */
@Aspect
public class AspectLogTest {

    @Target(ElementType.METHOD) //可以注解在方法 上
    @Retention(RetentionPolicy.RUNTIME) //运行时（执行时）存在
    public @interface AspectLog {
        String value();
    }

    final static String TAG = "TAG";

    @Pointcut("execution(@com.example.cm.aoptest.log.AspectLogTest.AspectLog * *(..))")
    public void logForActivity(){

    }

    @Before("logForActivity()")
    public Object doRealLog(JoinPoint joinPoint){
        try {
            MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
            AspectLog aspectJAnnotation = methodSignature.getMethod().getAnnotation(AspectLog.class);
            String log = aspectJAnnotation.value();
            Log.d(TAG, "doRealLog: 打印日志"+log);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return true;
    }
}

```

写好了接口，我们来实现自动注入代码，这里我们还是需要借助注解来实现。如果我们向打印``onCreate()``、``onStart()``等方法的中的信息，我们在``MainActivity``中这么写。
```
@AspectLogTest.AspectLog(value = "来自OnStop的日志信息")
@Override
protected void onStop() {
    super.onStop();
}

@AspectLogTest.AspectLog(value = "来自OnPause的日志信息")
@Override
protected void onPause() {
    super.onPause();
}

@AspectLogTest.AspectLog(value = "来自OnCreate的日志信息")
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
}
```
然后就实现这里自动log功能。
当然，从这个例子看起来好像和自己手动写``Log.d()``没有区别，只是用注解的方式调用而已。但是这里仅仅是为了展示他的功能，真正使用的时候我们可以通过他们来实现复杂的控制。

介绍一下Aspect常见的注解。
* **@Aspect** 把当前类标识为一个切面供容器读取
* **@Before** 表示一个前置增强方法
* **@AfterReturning** 表示一个后置增强方法
* **@AfterThrowing** 异常抛出增强
* **@After** final增强，不管是抛出异常或是正常退出都会执行
* **@Around** 环绕增强，先执行注入的方法，只有返回对应现场才能执行被注入的方法。
* **@DeclareParents** 引介增强

介绍一下``execution``切点函数，其定义为``execution([方法修饰符] [返回类型] 方法名 参数 [异常模式])``
在上例中，我们的``execution``函数为：
```
execution(@com.example.cm.aoptest.log.AspectLogTest.AspectLog * *(..))
```
注意这里的方法名需要添加包名，*表示匹配任意字符，..表示匹配任意字符，可以匹配任意多个元素。
+必须跟在类名后面，表示类本身和继承或扩展制定类的所有类。

# 基础概念
* **PointCut 切入点**：想要动态添加代码的连接点
* **Advice 通知** ：向切点中注入代码实现方法
* **Aspect 切面** ：切入点和通知的集合。
* **Joint Point 连接点**：所有目标方法都是连接点
* **Weaving 编织** 将切面代码注入到目标中，并生成代码混合到.class的过程。