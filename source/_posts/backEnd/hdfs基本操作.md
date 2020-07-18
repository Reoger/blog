---
title: hdfs基本操作
date: 2017-10-24 09:42:01
categories: backEnd
tags: [hdfs]
---
>HDFS（Hadoop Distributed File System）是Hadoop分布式计算中的数据存储系统，是基于流数据模式访问和处理超大文件的需求而开发的。

# 基本概念
下面介绍几个hdfs文件系统中几个常见的概念：

1. Block
HDFS中的存储单元是每个数据块block，HDFS默认的最基本的存储单位是64M的数据块。
2. NameNode
元数据节点，该节点用来管理文件系统中的命名空间。
3. DataNode
数据节点，是HDFS真正存储数据的地方。
4. Secondary NameNode
从元数据节点，从元数据节点并不是NameNode出现问题时候的备用节点，它的主要功能是周期性的将NameNode中的namespace image和edit log合并，以防log文件过大。
5. edit log
修改日志，用于记录hdfs中文件的相关操作日志。



# 命令操作

我们可以通过hadoop命令来实现文件的基本操作，包括添创建、删除目录，上传下载文件操作。
 ## 创建目录
 我们可以通过下面的命令来创建一个input目录。
 ``hadoop fs -mkdir hdfs://localhost:9000/input``
 这里的``hdfs://localhost:9000``表示的是本地的hdfs系统，如果是本地的hdsf系统，我们其实可以省略，我们上面的命令可以简写成：
 `` hadoop fs -mkdir /input``
当然，如果是远程的hadfs系统，我们就需要标注hdfs地址了，例如``hdfs://http://172.22.66.245:9099``，想知道hdfs开放的是那个端口，直接直接访问``http://localhost:50070/dfshealth.html#tab-overview``查看开放端口。
综上，我们在远程hdfs系统上创建的一个名为input目录的命令如下：
``hadoop fs -mkdir hdfs://172.22.66.245:9090/input``

## 上传文件
上传文件的也很简单，基本的格式为``hadoop fs -put localFile hdfsFileDir``
其中的``localFile``代表要上传的文件，``hdfsFileDir``代表要上传的到hdfs文件系统的目录。
例如，我要将D盘根目录下的keseh.txt上传到hdsf文件系统的``input``目录下，完整的命令如下：
``hadoop fs -put D:\keshe.txt hdfs://localhost:9000/input``
当然，因为是本地的hdfs系统，故hdfs可以省略，即可以简写成：
``hadoop fs -put D:\keshe.txt /input``
当然远程hdfs系统，这个hdfs就不能省略了，示例命令如下：
``hadoop fs -put D:\keshe.txt hdfs://172.22.66.245:9090/input``

## 下载文件
下载文件的基本格式为``hadoop fs -get hdfsFile localFileDir``，其中的``hdfsFile``表示的是hdfs系统中的文件，``localFileDir``表示要下载的到本地的目录。
这里就直接给出示例命令了.
从本地的hadfs系统获取，``hadoop fs -get hdfs://localhost:9000/input/keshe.txt F:\``

远程的hdfs系统获取：``hadoop fs -get hdfs://172.22.66.245:9090/input/keshe.txt F:\``

## 删除文件
删除文件的格式为``hadoop fs -rm hdfsFile``或者是``hadoop fs -rm -r hdfsFile``。
每次可以删除多个文件或者目录。例如，删除远程hdfs文件的命令为：
``hadoop fs -rm  hdfs://172.22.66.245:9090/input/keshe.txt``

## 基本命令
以上的操作都比较简单，在简单记录一下hadoop中的基本命令。

| 命令 | 格式 | 作用 | 实例 |
|:----:|:---:|------|------| 
| ls   |  hadoop fs -ls /< hdfs dir>  |    列出hadfs文件系统中某个目录下的文件 |   hadoop fs -ls hdfs://localhost:9000/input  |
| put  | hadoop fs -put < local file > < hdfs dir >  | 上传文件或者目录到hdfs系统中指定目录中 |  hadoop fs -put D:\keshe.txt hdfs://localhost:9000/input |
| get | hadoop fs -get < hdfs file > < local file or dir>| 下载hdfs文件系统中的文件或者目录 | hadoop fs -get hdfsFile localFileDir|
| moveFromLocal | hadoop fs -moveFromLocal  < local src > ... < hdfs dst >| 类似于将本地文件剪切到hdfs文件系统中 | hadoop fs -moveFromLocal D:\keshe.txt   hdfs://localhost:9000/input |
| copyFromLocal | hadoop fs -copyFromLocal  < local src > ... < hdfs dst > | 类似于将文件从本地复制到hdfs文件系统中 | hadoop fs -copyFromLocal D:\keshe.txt   hdfs://localhost:9000/input|
| mkdir | hadoop fs -mkdir < hdfs path> | 在hdfs文件系统中创建目录 | hadoop fs -mkdir hdfs://localhost:9000/input|
| rm    |  hadoop fs -rm < hdfs file > ...| 删除hdfs文件目录中的文件 |hadoop fs -rm  hdfs://localhost:9000/input/keshe.txt|
| cp   | hadoop fs -cp  < hdfs file >  < hdfs file >|复制hdfs文件或者福文件夹| hadoop fs -cp   hdfs://localhost:9000/input/keshe.txt   hdfs://localhost:9000/input/user |  
| mv | hadoop fs -mv < hdfs file >  < hdfs file > | 移动文件，也可以当着重命名来使用|hadoop fs -cp  hdfs://localhost:9000/input/keshe.txt  hdfs://localhost:9000/haha.txt|
| count | hadoop fs -count < hdfs path >| 统计hdfs对应路径下的相关信息,显示为目录个数，文件个数，文件总计大小，输入路径| hadoop fs -count  hdfs://localhost:9000/input |
| du   | hadoop fs -du < hdsf path> | 显示hdfs对应路径下每个文件夹和文件的大小| hadoop fs -du  hdfs://localhost:9000/input |
| text | hadoop fs -text < hdsf file> |将文本文件或某些格式的非文本文件通过文本格式输出| hadoop fs -text  hdfs://localhost:9000/input/haha.txt |

暂时先记录这么多了，下面我们用代码实现基本的文件操作。

# 用代码实现
在真是实现之前，是需要我们提前好代码开发环境的，至于如何搭建，参照前面上一篇博客，这里不再展开。

## 查看所有目录下的文件
我们先来看如何查看hdfs文件指定目录下所有的文件信息，即相当于ls命令的。
下面就是主要实现，我们住需要在main方法中调用就可以打印出hdfs文件系统中input目录下所有的文件和目录信息。
```
    private static void  ListItem(){
        try {
            Configuration config = new Configuration();
            Path dfs = new Path("hdfs://172.22.66.245:9090/input");
            FileSystem fileSystem = dfs.getFileSystem(config);
            FileStatus[] status = fileSystem.listStatus(dfs);
            for (int i = 0; i < status.length; i++) {
                System.out.println(status[i].getPath().toString());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

## 上传文件

代码很简单，直接上代码了,upFile方法实现了将本地D盘根目录下的2017-09-25_093127.jpg图片上传到hdfs文件系统中的input文件夹下。
```
    private static void upFile(){
        try{
            Configuration conf = new Configuration();
            Path path = new Path("hdfs://172.22.66.245:9090/input");
            Path localPath = new Path("D:\\2017-09-25_093127.jpg");
            FileSystem fs = path.getFileSystem(conf);
            fs.copyFromLocalFile(localPath,path);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

## 下载文件
下载文件比较坑的一点就是，我们需要先创建一个这样的文件，然后以流的形式传入到填充到这个文件中去，例如下面的示例代码就是将hdfs文件系统中的haha.txt文件的内容写到D盘根目录下的hello.txt文件中。
```
private static void downFile(){
        try{
            Configuration conf = new Configuration();
            Path path = new Path("hdfs://172.22.66.245:9090/input/haha.txt");
            FileSystem fs = path.getFileSystem(conf);
            FSDataInputStream open = fs.open(path);
            OutputStream output = new FileOutputStream("D://hello.txt");
            IOUtils.copyBytes(open,output,4096,true);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

# 参考链接
1. [hadoop HDFS常用文件操作命令](https://segmentfault.com/a/1190000002672666)
2. [HDFS中的基础概念](http://blog.csdn.net/zh521zh/article/details/51784450)