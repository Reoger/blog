---
title: centos 6.5下安装ElasticSearch5.6.1问题记录
date: 2017-11-06 22:44:55
categories: 杂烩 
tags: eleasticSearch
---

> 之前在window 10上安装elasticSearch是很简单的，但是真正要linux系统上安装时，还是出现了许多的问题，下面就是我记录的我遇到的一些问题。

# 整体流程
1. 下载elasticSearch5.6.1的压缩包，然后解压到指定文件夹下，参考解压命令：

```
tar -zxvf elasticsearch5.6.1.tar
```
2. 为其创建新用户（root用户无法启动），参考命令如下：
```
[root]# adduser elsearch
[root]# passwd elsearch
[root]# chown -R elsearch:elsearch elasticSearch5.6.1/
```
3. 切换到指定用户，并运行.参考命令：
```
[root]# su elsearch
[elsearch]$ cd elasticSearch5.6.1/bin
[elsearch]$ ./elasticSearch

在正常启动之后，应该就能通过访问http://localhost:9200来使用elasticsearch了
```
4. 配置ip地址访问。
这里和windows下修改的地方一样，但是不一样的是，这里出现的问题会很多,要修改的内容参考如下：
```
vim  elasticsearch-5.6.1/config/elasticsearch.yml

---
cluster.name: EsMaster  # 集群master的名称

node.name: master   # 当前结点的名称
node.master: true  # 设置为master节点

bootstrap.memory_lock: false  # 关闭内存锁定
bootstrap.system_call_filter: false

network.host: 127.0.0.1 # 这里修改成你要绑定的id


http.cors.enabled: true     # 设置跨域访问，为后面的head-master的配置
http.cors.allow-origin: "*"

---
保存，重启elasticsearch，发现诸多问题，下面是我遇到一些问题的记录。
```

5. 问题及解决方案
```
ERROR: [3] bootstrap checks failed
#文件句柄太少，至少要65536
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
#最大线程数太少，至少2048个(经典的2048游戏)
[2]: max number of threads [1024] for user [king] is too low, increase to at least [2048]
#虚拟内存太少，至少262144
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

[1]解决方案：
```
[root]# vi /etc/security/limits.conf

# 添加如下配置：
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096

```
[2]解决方案：
```
[root]#  vi /etc/security/limits.d/90-nproc.conf 

将
*          soft    nproc     1024
改成
*          soft    nproc     2048
```

[3]解决方案：
```
[root]# vim /etc/sysctl.conf 
添加如下代码：
vm.max_map_count=655360

保存后，执行
sysctl -p
```

最后，还有一个错：
```
system call filters failed to install; check the logs and fix your configur                                                                                                             ation or disable system call filters at your own risk
```
解决方案：
在elasticsearch.yml文件中配置如下：
```
[root]# vim elasticsearch.yml

---
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```


差不多解决上面的问题，应该就能正常运行elsearch了。

# 远程访问
我这里是利用ssh进行远程访问，记录一下基础的运行方式：
避免断开连接后``elasticsearch``就停止运行的运行方式：
```
 nohup bin/elasticsearch & 
```
当然，也直接直接进行如下命令来运行：
```
 bin/elasticsearch -d
```

关闭ElasticSearch服务就需要用到ps命令，参考如下：
```
[elsearch]$  ps -ef|grep elastic
elsearch 22097     1  0 15:06 ?        00:01:24 /usr/bin/java -Xms2g -Xmx2g 
...


[root]# kill -9 22097


```

【ps】 在选择启动方式的时候，虽然两种方式都能通过ssh启动elasticsearch服务，但是笔者实践证明，非主节点通过`` bin/elasticsearch -d``这条命令启动的服务不能加入到集群中，所以推荐多用``nohup bin/elasticsearch & ``来避免踩雷。

还有一点就是安装head-master，其实也没有太多的坑，只需要按照步骤一步一步来就OK了。

```
git clone git://github.com/mobz/elasticsearch-head.git
(如果 没有安装git，也可以采用其他方式下载head-master)

cd elasticsearch-head

npm install

（如果提示npm是无效命令，则需要先安装npm，参考步骤如下）

--安装npm
curl --silent --location https://rpm.nodesource.com/setup_5.x | bash -

yum install -y nodejs

输入下面的命令是检验:
[root@]# node -v

[root@]# npm -v
如果不提示是无效命令即成功安装。
----
```
在安装之后，进行简单的配置：
修改elasticsearch-head下Gruntfile.js文件，默认监听在127.0.0.1下9200端口：
在``connect``关键字下，``options``项中添加一项：
```
hostname: '0.0.0.0',
```
保存退出后，输入
```
npm run start
```
来启动head-master。
然后通过浏览器 http://localhost:9100 来访问。

到此，记录结束。

---
最后，记录两个比较有用的操作命令

远程启动服务：
```
方式1：

nohup bin/elasticsearch &

方式2：
./elasticsearch -d

```

远程关闭服务：
```
通过： 
ps -ef|grep elastic
获取elastic的PID
然后通过kill命令杀死就OK：
kill -9 22789
```
## 彩蛋
抄袭一下利用pscp传输文件的命令。

1、把服务器上的/root/dir目录取回本地"C:\My Documents\data\"目录
```
C:\>pscp.exe -r root@192.168.32.50:/root/dir "C:\My Documents\data\"
```
2、把服务器上的/root/file文件取回来本地当前目录
```
C:\>pscp.exe root@192.168.32.50:/root/file .
```
3、把本地目录dir、文件file传输到Linux服务器的/root/，并指定服务器端口2009
```
C:\>pscp.exe -P 2009 -r dir file root@192.168.32.50:/root/
```
4、把本地文件file传输到Linux服务器的/root/
```
C:\>pscp.exe file root@192.168.32.50:/root/
```
它会提示你输入密码，就像Linux下使用scp那样。

# 参考链接
* [CentOS6.5安装ElasticSearch5.5完整纪录与问题总结](http://blog.csdn.net/KingBoyWorld/article/details/77814528)
* [在Linux上安装Elasticsearch5.x](http://www.jianshu.com/p/e62bd4afe325)