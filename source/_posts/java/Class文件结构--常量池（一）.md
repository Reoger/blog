---
title: Class文件结构--常量池（一）
date: 2018-10-14 14:56:55
categories: java 
tags: [java,class,转载]
---


Class文件结构--常量池（一）
=================

---
转载自<https://www.jianshu.com/p/d8492e748c57>


*   字节码查看工具：WinHex

前言
==

*   Java虚拟机实现语言无关性的基石就是Class文件
    ![Java虚拟机提供的语言无关性](https://upload-images.jianshu.io/upload_images/3458176-30ce955fde8883d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/830/format/webp)
*   这篇文章讲Class格式文件的的魔数、版本号和常量池。主要内容是常量池。

Class类文件的结构
===========

全局规范
----

*   **1.任何一个Class文件都对应着唯一一个类或接口的定义信息，但反过来说，类或接口并不一定都得定义在文件里（譬如类或接口也可以通过类加载器直接生成）。本章中，只是通俗地将任意一个有效的类或接口所应当满足的格式称为“Class文件格式”，实际上它并不一定以磁盘文件的形式存在。“Class文件”应当是一串二进制的字节流，无论以何种形式存在。**
    
*   2.Class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在Class文件之中，当遇到需要占用8位字节以上空间的数据项时，则会按照**高位在前（Big-Endian）**的方式分割成若干个8位字节进行存储。无符号数据类型最大占8个字节。
    
*   3.Class文件中存储数据的类型：无符号数和表。
    
*   **无符号数（基本数据类型）**：以u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节和8个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。
    
*   **表（复合数据类型）**：是由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性地“\_info”结尾。表用于描述有层次关系的复合结构的数据。表是一个统称，就好比把ArrayList、LinkedList、Set都是称为集合(Collection)，但是每个集合的内部结构都是不同的，Class中有很多不同的表。如下图中cp\_info类型,是表类型，但是它是一个固定结构的类型吗？不是，它好比Collection集合下的List集合，只是一类集合的统称，实际上cp\_info表是14种具体表类型的统称，constant\_pool\_count-1指出了有多少个cp\_info表，那到底是哪些具体的表，就需要具体看了。
    
*   4.整个Class文件本质上就是一张表，下表就是Class文件格式。Class中所有内容都在这些类型中定义了。
    
    *   **注：表中的数据项，无论是顺序还是数量，甚至于数据存储的字节序（Byte Ordering,Class文件中字节序为Big-Endian）这样的细节，都是被严格限定的，哪个字节代表什么含义，长度是多少，先后顺序如何，都不允许改变。**  
        
        ![](https:https://upload-images.jianshu.io/upload_images/3458176-8fce946bab563076.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/972/format/webp)
        
        Class文件格式
        
        ![](https://upload-images.jianshu.io/upload_images/3458176-183170e077348dd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/847/format/webp)
        
        class文件结构
        
*   5.无论是无符号数还是表，当需要描述同一类型但数量不定的多个数据时，经常会使用一个前置的容量计数器加若干个连续的数据项的形式，这时称这一系列连续的某一类型的数据为某一类型的集合。
    
*   **如上表的描述常量池数据使用了一个constant\_pool\_count、多个constant\_pool,其中constant\_pool是表类型并且数量为constant\_pool\_count值减去1，把一个constant\_pool\_count和多个constant_pool数据项称为常量池集合**
    
*   从Class文件格式中可以看出有：常量池集合、接口索引集合、字段表集合、方法表集合、属性表集合。
    
*   6.具体的Class文件案例，以下讲解会通过这个TestClass类的``TestClass.class``文件来分析。
```
    package com.zlcook.clazz;
    
    public class TestClass{
      private int m;
      public int inc(){
       return m+1;
      }
    }
```

1\. 魔数与Class文件的版本
-----------------

*   由上表得Class文件的前三个数据类型存储了魔数（magic)、次版本号(minor\_version)、主版本号(major\_version)的值，数据类型分别为u4、u2、u2。共占8个字节。
*   魔数：0xCAFEBABE （16进制），值固定，唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。
*   Class文件版本号：次版本号组成u2+主版本号u2。共占4个字节。
*   **高版本的JDK能向下兼容以前的版本的Class文件，但不能运行高版本的Class文件。**
*   JDK1.1的版本号为45.0-45.65535（10进制），之后每个大版本发布主版本号加1，如：JDK1.2：46.0~46.65535。
*   例如：Class文件中紧接着魔数的4个字节的16进制为： ox00000034，那么它代表的十进制版本号为：次版本号为ox0000=0，主版本号为：ox0034 = 52。所以ox00000034的版本号为52.0，对应的JDK版本为JDK1.8

![](https://upload-images.jianshu.io/upload_images/3458176-4113215a9048a30f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/284/format/webp)

``TestClass.class``文件的前8个字节

2\. 常量池
-------

*   先了解常量池中需要存放哪些内容，再讨论用什么类来存放这些内容。

### 2.1 常量池中存放的内容

*   Class文件中包含常量池，那么我就需要知道常量池会包含哪些内容，接下来才是关心class格式文件用什么类型来存放这些内容。
    
*   **字面量（Literal）**
    
*   字面量比较接近于Java语言层面的常量概念，如文本字符串、声明为final的常量值等。
    
*   **符号引用（Symbolic References）**
    
*   符号引用则属于编译原理方面的概念，包括了下面三类常量：  
    类和接口的全限定名（Fully Qualified Name）  
    字段的名称和描述符（Descriptor）  
    方法的名称和描述符
    
*   **其它：**常量池中**主要**内容是上面2项，说明还有其它内容，这部分内容，在下面我们看到用来描述常量池内容的14种常量项的介绍时就发现标志为15、16、18的常量项类型是用来支持动态语言调用的（jdk1.7时才加入的）。
    
*   常量池：可以理解为Class文件之中的资源仓库，它是Class文件结构中与其他项目关联最多的数据类型（后面的很多数据类型都会指向此处），也是占用Class文件空间最大的数据项目之一。
    

### 2.2 常量池中为什么要包含这些内容

*   Java代码在进行Javac编译的时候，并不像C和C++那样有“连接”这一步骤，而是在虚拟机加载Class文件的时候进行动态连接。也就是说，在Class文件中不会保存各个方法、字段的最终内存布局信息，因此这些字段、方法的符号引用不经过运行期转换的话无法得到真正的内存入口地址，也就无法直接被虚拟机使用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、翻译到具体的内存地址之中。关于类的创建和动态连接的内容，在虚拟机类加载过程时再进行详细讲解。

### 2.3 Class文件中如何描述常量池中内容

*   知道Class文件的常量池包含的内容后，我们下面就来看看class格式文件使用了哪些类型数据来存放常量池的内容。
    
*   由Class文件格式可得紧接着主版本号的是常量池入口。

|类型|名称|数量|
|---|----|---|
|u2（无符号数）|constant\_pool\_count|1|
|cp_info（表） |constant_pool|constant\_pool\_count-1|

*   占用的字节数：2+(constant\_pool\_count-1)个具体表所占字节。
    
*   由上表可见，Class文件使用了一个前置的容量计数器（constant\_pool\_count）加若干个连续的数据项（constant_pool）的形式来描述常量池内容。我们把这一系列连续常量池数据称为常量池集合。
    
*   先给看一下TestClass.class文件全局的内容，下面就来分析其中常量池中的内容，其它内容后面的文章在分析。从图片也可以看出常量池内容占据了class文件的很大一部分，当然TestClass类中代码比较少就更显得常量池内容的多了。
 
![TestClass.class文件的16进制内容](https://upload-images.jianshu.io/upload_images/3458176-da04f13d9345d534.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/633/format/webp)


#### 2.3.1 constant\_pool\_count

*   常量池容量计数值（u2类型）：**从1开始**，表示常量池中有多少项常量。即constant\_pool\_count=1表示常量池中有0个常量项。
    
*   设计者将第0项常量空出来是有特殊考虑的，这样做的目的在于满足后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义，这种情况就可以把索引值置为0来表示。Class文件结构中只有常量池的容量计数是从1开始，对于其他集合类型，包括接口索引集合、字段表集合、方法表集合等的容量计数都与一般习惯相同，是从0开始的。
    
*   TestClass.class文件中constant\_pool\_count的十进制值为19，表示常量池中有18项常量，索引范围1-18。
    

![](https://upload-images.jianshu.io/upload_images/3458176-1708ec53425bf43f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/385/format/webp)

TestClass.class文件中constant\_pool\_count的十进制值为19

#### 2.3.2 constant_pool

*   constant\_pool\_count表明了后面有多少个常量项。

##### 14种常量项结构

*   常量池中每一项常量都是一个表，JDK1.7之后共有14种不同的表结构数据。一个常量池中的每个常量项都逃不脱这14种结构。根据下图每个类型的描述我们也可以知道每个类型是用来描述常量池中哪些内容（主要是字面量、符号引用）的。比如：CONSTANT\_Integer\_info是用来描述常量池中字面量信息的，而且只是整型字面量信息。而标志为15、16、18的常量项类型是用来支持动态语言调用的（jdk1.7时才加入的）。
    
![常量池中的14种项目类型](https://upload-images.jianshu.io/upload_images/3458176-cc110223178a4215.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/977/format/webp)
    

    
![常量池中的14种常量项的结构总表](https://upload-images.jianshu.io/upload_images/3458176-8b9bb010f69e4a93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/701/format/webp)
    

![常量池中的14种常量项的结构总表（续）](https://upload-images.jianshu.io/upload_images/3458176-878fa839b1e28cf3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/694/format/webp)

    
*   这14种表（或者常量项结构）的共同点是：表开始的第一位是一个u1类型的标志位（tag），代表当前这个常量项使用的是哪种表结构，即哪种常量类型。
    
*   这14种常量项结构还有一个特点是，其中13表占用得字节固定，只有CONSTANT\_Utf8\_info占用字节不固定，其大小由length决定。为什么呢？因为从常量池存放的内容可知，其存放的是字面量和符号引用，最终这些内容都会是一个字符串，这些字符串的大小是在编写程序时才确定，比如你定义一个类，类名可以取长取短，所以在没编译前，无法确定大小不固定，编译后，通过utf-8编码，就可以知道其长度。
    
|表|占用字节|
|---|------|
|CONSTANT\_Class\_info|3|
|CONSTANT\_Integer\_info|5|
|CONSTANT\_Fieldref\_info|5|
|CONSTANT\_Methodref\_info|5|
|CONSTANT\_Utf8\_info|不固定，取决于length大小|


2.4 查找testClass.class文件的第一个常量项内容
--------------------------------

*   由上面constant\_pool\_count得到值为19，因为从1开始计数，所以说明后面有18个常量项，由于每个常量项的表结构都不同但是第一位相同，所以读到第一位就可以确定表结构了。下面我们就来查看第一个常量项包含得内容，至于其它17个常量项内容类似，最后还会介绍java提供得一个工具命令javap来帮我们分析class文件字节码内容。

![第一个表的tag为10](https://upload-images.jianshu.io/upload_images/3458176-22b038defbeda453.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/655/format/webp)



*   由上图可知常量池中第一项常量标志的16进制值是0x0A=10,查表发现这个常量属于CONSTANT\_Methodref\_info类型，此类型表示类中方法的符号引用。查看该类型的结构如下：

![CONSTANT\_Methodref\_info类型结构](https://upload-images.jianshu.io/upload_images/3458176-18d7609da23c0b93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/851/format/webp)
    
    
    
*   CONSTANT\_Methodref\_info型常量的第二个数据项为index，类型是u2，index存储的是一个索引值，从class文件中查得该值为oX0004=4，即它指向常量池中第4个常量；第三个数据项也是索引其值为0X000F=15，指向常量池种第15个常量。

![](https://upload-images.jianshu.io/upload_images/3458176-1a800d5b071be1b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/623/format/webp)

Paste_Image.png

*   到此为止，第一个常量项是CONSTANT\_Methodref\_info型常量项，该类型常量项用来表示类中方法的符号引用，其内容为tag=10,index1=4,index2=15，因为其表示的是类中方法的符号引用，所以index中存放的不是一个具体得内容，而是一个索引位置，所以说其具体内容存放在另一个常量项中。下面我们就来看看其索引指向的常量项（即第4个常量项）的内容到底是什么？
*   找第4个常量项之前需要知道第4个常量项的开始位置，所以需要知道前3个常量项所占字节数。那好就看第2个常量项，由于第一个常量项共占了5个字节，则紧接着的字节就为第二个常量项的tag，如下图可得其值为0X09=9，说明第2个常量项得项目类型为CONSTANT\_Fieldref\_info。查表得其该类型得字节长度固定占5个字节。

![](https://upload-images.jianshu.io/upload_images/3458176-b27b29fd146c1371.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/638/format/webp)

第二个常量项

*   依次类推查的第3,4个常量项为CONSTANT\_Class\_info型。如下图：

![前4个常量项](https://upload-images.jianshu.io/upload_images/3458176-9dedb81d1b044555.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/646/format/webp)
    

    
*   下面就看第四个常量项CONSTANT\_Class\_info的内容0X070012。 CONSTANT\_Class\_info存放的是指向类或接口的符号引用。
    

![CONSTANT\_Class\_info型常量项](https://upload-images.jianshu.io/upload_images/3458176-31580b94471eec94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/641/format/webp)
    
    
    

根据CONSTANT\_Class\_info项常量项的结构可知其index数据项又是一个索引项，指向全限定名常量项索引，index数据项的值为0X12=18，表示指向第18个常量项，根据constant\_pool\_count的值为19可得，常量池中一共有18个常量项，巧了正好在最后一个，但是要知道18个常量项必须知道前17个常量项所占字节，这里就不一一找了，最后找到第18个常量项CONSTANT\_Utf8\_info在class文件中包含的内容如下：

![第18个常量项](https://upload-images.jianshu.io/upload_images/3458176-942c562db653d825.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/649/format/webp)


*   ![CONSTANT\_Utf8\_info型表的结构](https://upload-images.jianshu.io/upload_images/3458176-6d21f61844e69dc4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/851/format/webp)

    
*   根据tag等于1得第18项是CONSTANT\_Utf8\_info型，该类型存储UTF-8编码的字符串，在TestClass.class文件种该常量项种个数据项的内容如下：
    
    *   length(u2):表示UTF-8编码的字符串占用的字节数，值为0x0010=16.
    *   bytes(u1):表示长度为length的UTF-8编码的字符串.
    *   因为length=16，所以 length后面紧跟的长度为16个字节的连续数据是一个使用**UTF-8缩略编码**表示的字符串。后面紧跟的第一个字节为0x6A=106，那该编码代表的字符为j，我们发现106其实就是字符j对应的[ASCII码](https://link.jianshu.com?t=http://baike.baidu.com/link?url=yWgaMCeHhkhUQfJByEHL7PwMq1jWtYH8sGqB063JSCSy4Si2bWfVPzVMClgYqGyhW8tpLBd6qKWYLISuGI1024oRv9HEUOV_XaIi4QeCE2GTKQHAVU8DcVtr6Gu2UFDZhImE37QR1-FVFHKkg66k3a)。后面16个字节代表的字符就是: java/lang/Object

到此为止，我们得到了第一个常量项CONSTANT\_Methodref\_info的第二个数据项index指向的内容为CONSTANT\_Class\_info常量项，CONSTANT\_Class\_info常量的第二个数据项index指向CONSTANT\_Utf8\_info常量项，CONSTANT\_Utf8\_info常量项的内容为 java/lang/Object 。  
当然CONSTANT\_Methodref\_info常量项还有第三个数据项index，其存放的也是一个其他常量的索引。

*   根据上面的找法我们就可以找出常量池中包含的内容：字面量和符号引用。

2.5 采用javap命令分析class文件
----------------------

*   根据上面的找法我们就可以找出常量池中包含的内容：字面量和符号引用。java考虑到这种找法太麻烦了，所以提供了一个命令javap来帮助我们分析class文件的内容。
    
*   javap分析class文件用法：javap -verbose class文件名

```
    $ javap -verbose TestClass.class
    Classfile /E:/studytry/com/zlcook/clazz/TestClass.class
      Last modified 2017-4-7; size 292 bytes
      MD5 checksum 486567c6d4d7432fc359230fed9c92c7
      Compiled from "TestClass.java"
    public class com.zlcook.clazz.TestClass
      SourceFile: "TestClass.java"
      minor version: 0
      major version: 52
      flags: ACC_PUBLIC, ACC_SUPER
    Constant pool:
       #1 = Methodref          #4.#15         //  java/lang/Object."<init>":()V
       #2 = Fieldref           #3.#16         //  com/zlcook/clazz/TestClass.m:I
       #3 = Class              #17            //  com/zlcook/clazz/TestClass
       #4 = Class              #18            //  java/lang/Object
       #5 = Utf8               m
       #6 = Utf8               I
       #7 = Utf8               <init>
       #8 = Utf8               ()V
       #9 = Utf8               Code
      #10 = Utf8               LineNumberTable
      #11 = Utf8               inc
      #12 = Utf8               ()I
      #13 = Utf8               SourceFile
      #14 = Utf8               TestClass.java
      #15 = NameAndType        #7:#8          //  "<init>":()V
      #16 = NameAndType        #5:#6          //  m:I
      #17 = Utf8               com/zlcook/clazz/TestClass
      #18 = Utf8               java/lang/Object
    {
      public com.zlcook.clazz.TestClass();
        flags: ACC_PUBLIC
        Code:
          stack=1, locals=1, args_size=1
             0: aload_0
             1: invokespecial #1                  // Method java/lang/Object."<init>":()V
             4: return
          LineNumberTable:
            line 2: 0
    
      public int inc();
        flags: ACC_PUBLIC
        Code:
          stack=2, locals=1, args_size=1
             0: aload_0
             1: getfield      #2                  // Field m:I
             4: iconst_1
             5: iadd
             6: ireturn
          LineNumberTable:
            line 6: 0
    }

```

*  上面通过javap命令得到的结果，该结果显示的很友好，由过上面的理论我们可以很清楚的看到常量池一共18项：其中第一项如下：
```
     #1 = Methodref          #4.#15       //  java/lang/Object."<init>":()V
```

*   和我们通过手动方式查看第一个常量项CONSTANT\_Methodref\_info对比一下就知道javap显示的内容是多么友好了。

|第一个常量项|第几个|tag|index|index|最终代表的内容|
|-----------|-----|----|----|-----|-------------|
|class中16进制值|0X0A|0X004|0X000F||
|转换成10进制值|10|4|15|查完4和15才知道|
|javap分析显示的友好值|#1|Methodref|#4|#15|java/lang/Object."\<init>":()V|

2.6 class文件中包含的内容
-----------------

*   下面我们来看一下class文件中常量池的内容和java源码中的内容。
    
*   TestClass.java代码内容
    
```
    package com.zlcook.clazz;
    
    public class TestClass{
      private int m;
      public int inc(){
       return m+1;
      }
    }
```

*   TestClass.class中常量池内容：
```
    Constant pool:
       #1 = Methodref          #4.#15         //  java/lang/Object."<init>":()V
       #2 = Fieldref           #3.#16         //  com/zlcook/clazz/TestClass.m:I
       #3 = Class              #17            //  com/zlcook/clazz/TestClass
       #4 = Class              #18            //  java/lang/Object
       #5 = Utf8               m
       #6 = Utf8               I
       #7 = Utf8               <init>
       #8 = Utf8               ()V
       #9 = Utf8               Code
      #10 = Utf8               LineNumberTable
      #11 = Utf8               inc
      #12 = Utf8               ()I
      #13 = Utf8               SourceFile
      #14 = Utf8               TestClass.java
      #15 = NameAndType        #7:#8          //  "<init>":()V
      #16 = NameAndType        #5:#6          //  m:I
      #17 = Utf8               com/zlcook/clazz/TestClass
      #18 = Utf8               java/lang/Object
```

*   再复习一下常量池中主要存放字面量：如文本字符串、声明为final的常量值等。和符号引用：类和接口的全限定名、字段的名称和描述符、方法的名称和描述符。
*   所以出现com/zlcook/clazz/TestClass、java/lang/Object、m、inc都是应该的，那么I、V、<init>、LineNumberTable都是什么？那肯定是字段描述符或者是方法描述符了。这部分是编译时自动生成的，它们会被class文件中其它部分（字段表field\_info、方法表method\_info、属性表attribute_info）引用到，它们会用来描述一些不方便使用“固定字节”进行表达的内容。譬如描述方法的返回值是什么？有几个参数？每个参数的类型是什么？因为Java中的“类”是无穷无尽的，无法通过简单的无符号字节来描述一个方法用到了什么类，因此在描述方法的这些信息时，需要引用常量表中的符号引用进行表达。

3\. 哪些字面量会进入常量池中
----------------

*   我们知道class文件存放字面量：如文本字符串、声明为final的常量值等。这里的“等”就挺烦人。
*   下面我们来看看哪些字面量会进入常量池。（jdk1.8.0环境）

**8种基本类型：**

测试案例：

*   final类型 FinalTest.java代码

```
    public class FinalTest{
    
       private final int int_num =12;
       private final char char_num = 'a';
       private final short short_num =30;
       private final float float_num = 45.3f;
       private final double double_num =39.8;
       private final byte byte_num =121;
       private final long long_num = 2323L;
       private final boolean boolean_flage = true;
    }
```

*   非final类型 test.java代码
```
    public class test{
    
       private int int_num =12;
       private char char_num = 'a';
       private short short_num =30;
       private float float_num = 45.3f;
       private double double_num =39.8;
       private byte byte_num =121;
       private long long_num = 2323L;
       private long long_delay_num ;
       private boolean boolean_flage = true;
    
       public void init(){
         this.long_delay_num = 5555L;
       }
    }
```

上面代码测试结果：

*   final类型的8种基本类型的值会进入常量池。
*   非final类型的8种基本类型的值double、float、long的值会进入常量池，包括long\_delay\_num的值。

**String类型**
```
*   StringTest.java代码：

    public class StringTest{
    
          private String str1 = "zl"+"cook";
          private String str2 = str1+"hello";
          private String str3 = new String("zlcook here?");
          private String str4 = "everybody "+ new String("here?");
    
          private final String fin1 = "boy";
          private final String fin2 = fin1+ "is boy";
          private final String fin3 = str1+ "is boy";
    }
```

*   StringTest.class的常量池种包含内容：
    
![常量池中包含的字符串类型字面量](https://upload-images.jianshu.io/upload_images/3458176-adab40db73749fe0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/709/format/webp)
    
所有测试数据github： [测试数据](https://link.jianshu.com?t=https://github.com/zlcook/JVM/tree/master/constant)

结束
==

*   这一节主要讲了Class文件魔数、版本号和常量池，比较详细介绍了常量池包含的内容以及用到的14种常量项结构。记住本节讲的常量池是class文件中的常量池，要记住还有运行时常量池，每个class文件中的常量池内容在类加载侯会进入方法区的运行时常量池中存放。当然运行时常量池的内容不仅包含这些还包含运行期加入的常量，常见的就是String类的intern()方法。