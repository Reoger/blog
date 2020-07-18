---
title: intellij IDEA 配置hadoop开发环境
date: 2017-10-23 15:10:25
categories: backEnd 
tags: others,hadoop
---

> 在参考了众多的参考资料之后，终于搭建搭建起了了intellij-IDEA开发hadoop环境，下面就记录一下开发环境的建立过程，以免日后遗忘。

# 开发工具和基础环境准备
首先，本次采用的开发工具选用的是intellij-IDEA，在进行本次开发之前需要将intellij-IDEA下载并安装好，并配置好java的JDK。下面是我本次配置开发环境之前的基础环境，本次采用的是hadoop 2.6.0的版本。
```
IntelliJ IDEA 2017.2.4
Build #IC-172.4155.36, built on September 12, 2017
JRE: 1.8.0_152-release-915-b11 amd64
JVM: OpenJDK 64-Bit Server VM by JetBrains s.r.o
Windows 10 10.0
```
在下载好hadoop之后，我们便可以开始本次环境的配置了。

# 创建项目
这一步非常的简单，我们用IDEA创建一下普通的java项目即可。

![](http://ovec6nnof.bkt.clouddn.com/%E6%96%B0%E5%BB%BA%E9%A1%B9%E7%9B%AE.jpg)
创建项目完成之后是这个样子的。
![](http://ovec6nnof.bkt.clouddn.com/%E5%88%9B%E5%BB%BA%E5%AE%8C%E6%88%90.jpg)
# 添加示例代码
我们先不管三七二十一，在我们的项目中，先copy下面三个类，用来检测环境是否搭建好。
第一个类``WordCountMapper``
```
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;
import java.util.StringTokenizer;

public class WordCountMapper  extends Mapper<LongWritable,Text,Text,IntWritable> {
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        StringTokenizer itr = new StringTokenizer(value.toString());
        while (itr.hasMoreTokens()){
            word.set(itr.nextToken());
            context.write(word,one);
        }
    }
}

```
第二个类:``WordCountReducer ``
```

import org.apache.hadoop.io.IntWritable;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;


import java.io.IOException;


public class WordCountReducer extends Reducer<Text,IntWritable,Text,IntWritable> {

    private IntWritable result = new IntWritable();

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for(IntWritable val:values){
            sum += val.get();
        }

        result.set(sum);
        context.write(key,result);
    }
}
```
第三个测试类：``test``
```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.log4j.chainsaw.Main;

import java.io.IOException;

public class test {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        // write your code here

        Configuration configuration = new Configuration();

        if(args.length!=2){
            System.err.println("Usage:wordcount <input><output>");
            System.exit(2);
        }

        Job job = new Job(configuration,"word count");

        job.setJarByClass(Main.class);
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job,new Path(args[0]));
        FileOutputFormat.setOutputPath(job,new Path(args[1]));

        System.exit(job.waitForCompletion(true)?0:1);
    }
}

```
把代码copy完成之后，项目结构应该是这样子的：
![](http://ovec6nnof.bkt.clouddn.com/%E5%A4%8D%E5%88%B6%E4%BB%A3%E7%A0%81%E5%AE%8C%E6%88%90.jpg)

可以看到，ide提示我们代码还有错，这是因为我们环境还没有搭建完成，所以报错，我们接下来继续下一步。

# 导入hadoop相关的jar包
我们首先需要知道我们要导入的jar包的具体位置： ``\hadoop-2.6.0\share\hadoop``，即在hadoop根目录下的share文件夹中的hadoop文件夹中。
然后在IDEA中打开project Structure选项，快捷键``ctrl+alt+shift+S``。选择modules，如图：
![](http://ovec6nnof.bkt.clouddn.com/%E9%80%89%E6%8B%A9modules.jpg)

选择右边的加号-> jars 如图：</br>
![](http://ovec6nnof.bkt.clouddn.com/%E6%B7%BB%E5%8A%A0jar.jpg)

选择hadoop的jar，具体位置我们前面已经说过了，将hadoop文件夹中所有的文件都选中，按住shift即可多选：
![]()
jar导入完成之后，应该是这个样子的：
![](http://ovec6nnof.bkt.clouddn.com/jar%E5%AF%BC%E5%85%A5%E5%AE%8C%E6%88%90.jpg)

然后在Project Settings中选择Artifacts，如图：
![](http://ovec6nnof.bkt.clouddn.com/%E9%80%89%E6%8B%A9Artifacts.jpg)
选择加号->JAR -> Emptry创建一个空的jar。选择完成后如图：

![](http://ovec6nnof.bkt.clouddn.com/%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84jar.jpg)
我们将其取名为``hadoop``,并选择output Layout下面的加号->Module Output，并勾选include in prokecy builde,完成后如图所示：

![](http://ovec6nnof.bkt.clouddn.com/%E5%88%9B%E5%BB%BAjar%E5%AE%8C%E6%88%90.jpg)

完成上面的操作后，发现我们前面copy的代码已经不在报错了，说明jar包已经成功导入到我们的项目中来了。最后我们需要配置我们的运行环境。

# 配置运行环境
首先选择edit Configurtions,如图；
![](http://ovec6nnof.bkt.clouddn.com/%E9%80%89%E6%8B%A9edit%20Configuratiobns.jpg)
新建application，具体做法，点击右上角的加号->选择application。新建界面如图所示：

![](http://ovec6nnof.bkt.clouddn.com/applicatuon%E5%8F%82%E6%95%B0%E9%85%8D%E7%BD%AE.jpg)
我们需要填的只有三处，首先我们可以为他取个名字，在Name那么输入自定义的名义，然后再Main class那里填写：``org.apache.hadoop.util.RunJar``，选择progream arguments右边的展开按钮，填写此项的值，在这里需要填写三个参数，第一个就是我们前面创建jar的具体位置，我们可以在前面常见的artifacts中查看，示例``D:\intellijIDEA\hadoopTest\out\artifacts\hadoop\hadoop.jar``；第二个参数是我们要运行类的名字（需要加上包名）示例``com.hut.Test``;第三个是输入文件的位置,示例``input/``；第四个是输入文件的名字示例``output/；
参考配置如下：
```
D:\intellijIDEA\hadoopTest\out\artifacts\hadoop\hadoop.jar
com.hut.Test
input/
output/
```
![](http://ovec6nnof.bkt.clouddn.com/program%20agrumets%E5%8F%82%E6%95%B0%E5%80%BC.jpg)

这里需要根据自己项目的实际情况来进行配置，仅供参考。
![](http://ovec6nnof.bkt.clouddn.com/%E9%85%8D%E7%BD%AE%E5%AE%8C%E6%88%90.jpg)


# 运行检测
根据我们前面的配置，我们需要在项目的根目录下新建一个input文件夹。我们可以在这个文件夹中添加一个txt的文件，并随意的写入一些数据。
然后点击运行，成功运行后，会发现我们项目根目录下新添加了一个output文件夹，在``part-r-0000``文件中，就记录了我们在input文件夹中文件单词出现的次数。如图：
![](http://ovec6nnof.bkt.clouddn.com/%E6%88%90%E5%8A%9F%E8%BF%90%E8%A1%8C~.jpg)
值得注意的是，每次成功运行，都需要将output文件夹删除后才能继续下一次的运行。

到此，我们的环境已经配置好了，接下来就可以开开心的学习hadoop的api使用了。

# 参考链接
* [Hadoop Intellij IDEA本地开发环境搭建](http://blog.csdn.net/u010171031/article/details/53024516)
* [intellij idea本地开发调试hadoop的方法](http://blog.csdn.net/duhe2665991640/article/details/60468966)
