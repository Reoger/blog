---
title: elasticsearch入门
date: 2017-09-27 20:48:46
categories: backEnd
tags: eleasticSearch
---

# Elasticsearch 是什么
Elasticsearch是一个基于[Apache Lucene(TM)](https://lucene.apache.org/core/)的开源搜索引擎。无论在开源还是专有领域，Lucene可以被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库。
但是，Lucene只是一个库。想要使用它，你必须使用Java来作为开发语言并将其直接集成到你的应用中，更糟糕的是，Lucene非常复杂，你需要深入了解检索的相关知识来理解它是如何工作的。
Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。
不过，Elasticsearch不仅仅是Lucene和全文搜索，我们还能这样去描述它：
分布式的实时文件存储，每个字段都被索引并可被搜索
分布式的实时分析搜索引擎
可以扩展到上百台服务器，处理PB级结构化或非结构化数据
而且，所有的这些功能被集成到一个服务里面，你的应用可以通过简单的RESTful API、各种语言的客户端甚至命令行与之交互。
上手Elasticsearch非常容易。它提供了许多合理的缺省值，并对初学者隐藏了复杂的搜索引擎理论。它开箱即用（安装即可使用），只需很少的学习既可在生产环境中使用。
Elasticsearch在Apache 2 license下许可使用，可以免费下载、使用和修改。
随着你对Elasticsearch的理解加深，你可以根据不同的问题领域定制Elasticsearch的高级特性，这一切都是可配置的，并且配置非常灵活。

# 开始使用
因为本人电脑是win10的系统，所以针对elasticsearch所有的操作与环境都是在win10环境下完成，至于其他系统环境，这里不予讨论。

## 安装
安装什么的最简单了，首先肯定要将elasticsearch下载到本地，要下载请戳[这里](https://www.elastic.co/downloads/elasticsearch)，下载完成后，直接解析即可开始我们的探究之旅。关于版本，这里需要做一点说明的是，elasticsearch总共有三个大版本，即1XX、2XX和5XX三个，我们这里使用的是最新的版本``5.6.1``进行学习
解压完成后，我们进入``bin``目录下，点击``elasticsearch.bat``即可运行。在成功运行后，我们通过浏览器访问<http://localhost:9200/>得到json格式的数据，即说明我们的elasicsearch成功安装并运行了。如果你在这里失败了，可能因素很多，有可能是你电脑里的java环境问题（没有java环境、java版本低于8等等），也有可能是你的电脑内存啥的太小了（一般不会出现在自己的电脑上，不过我将他部署在腾讯云上时跑不起来就是因为内存太小了）。失败的解决方案就不多介绍了，我们继续往下走。

## 安装插件
通过前面我们的测试安装，我们发现elasticsearch给我们返回的是json格式的数据，并不是很直观，这里我们通过安装一个叫做``elasticsearch-head-master``的插件，使得我们能更加直观的看到elasticsearch给我们返回的数据。
这个插件的github地址在[这里](https://github.com/zt1115798334/elasticsearch-head-master)
，我们可以通过他的github地址来学习如何安装和使用。
1. 首先肯定是将这个插件下载下来，在github下载资料对你来说应该是很简单了，这里就不赘述了，例如我们可以这样下载：``git clone git://github.com/mobz/elasticsearch-head.git``
2. 进入下载并解压好的源代码的目录。
3. 输入 ``npm install``，安装必要的资源。
4. 通过``npm run start``来启动插件
5. 在浏览器中打开<http://localhost:9100/>来测试是否正常打开。
能正常打开之后，这里时候我们发现elasticsearch和我们head并没有联系到一起，这里因为这两个都运行在独立的进程中，他们之间的访问存在跨域问题，这里时候我们需要进行一个简单的配置才能将这个插件真正运行起来。首先我们需要在``elasticsearch``目录下，找到``config``目录，然后再``config``中找到``elasticsearch.yml``文件，在这里文件的最后，添加下面的两行代码：
```
http.cors.enabled: true
http.cors.allow-origin: "*"
```
保存，退出后，将elasticsearch和head-master都重启后，重新访问<http://localhost:9100/>就会发现，此时我们的集群已经链接。这里时候我们就可以继续探索了。

## 分布式安装
上面的我们的安装属于单利安装，elasticsearch也支持分布式安装。下面介绍一下分布式安装。首先，我们制定一个msater，即为集群的指挥官。还是在之前的节点配置文件下，修改``elasticsearch.yml``文件中的内容，主要就是在末尾添加如下的配置文件：
```
cluster.name: reoger    # 指定集群的名字
node.name: msater       # 对master取名
node.master: true       #指定他为master
network.host: 127.0.0.1 # 绑定ip
```
修改上面的配置重启后，我们再次访问<http://localhost:9100/>就可以发现集群的名字已经修改为我们配置的名字。
然后，我们再来添加其他的集群。步骤也很简单，我们将之前下载好的``elasticsearch``复制一份，然后修改其中的``elasticsearch.yml``文件，添加如下的配置：
```
cluster.name: reoger                            # 指定集群名气，需要和master指定的一致
node.name: slave1                               # 指定服务节点的名字

network.host: 127.0.0.1                         # 指定本地ip
http.port: 8200                                 # 指定端口号

discovery.zen.ping.unicast.hosts: ["127.0.0.1"] # 发现msater的
```
然后通过``./bin/elasticsearch``启动后，通过浏览<http://localhost:9100/>，就会发现我们已经添加了两个集群了。如图：
![分布式安装效果图](http://ovec6nnof.bkt.clouddn.com/2017-09-28_133458.jpg)

## 创建索引
在创建索引之前，很有必要对elasticsearch中的基本概念进行一个简单的了解。
一句话理解就是：Elasticsearch集群可以包含多个**索引**(indices)（数据库），每一个索引可以包含多个**类型**(types)（表），每一个类型包含多个**文档**(documents)（行），然后每个文档包含多个**字段**(Fields)（列）。

下面我们就来创建索引：
**结构化创建**
方法一、通过head-master创建
为了使创建索引比较直观，我们可以直接通过head-master插件来完成，具体我们可以访问<http://localhost:9100/>,查看我们集群的状况，然后选择索引 -> 新建索引，如图所示：

稍等片刻之后就会提示我们索引创建成功，然后在集群概览中就可以看到我们之前添加的索引。
然后选择复合查询，在查询创建我们的索引。json示例代码如下：
```
{
  "novel": {
    "properties": {
      "title": {
        "type": "text"
      }
    }
  }
}
```
如图：
![利用head-master常见索引](http://ovec6nnof.bkt.clouddn.com/%E5%88%9B%E5%BB%BA%E7%B4%A2%E5%BC%95.jpg) 

验证我们是否添加成功，可以在``概览 -> 信息 -> 索引信息 。
如图：
![效果展示](http://ovec6nnof.bkt.clouddn.com/%E6%95%88%E6%9E%9C%E5%B1%95%E7%A4%BA.jpg)
我们可以明显的看出来，我们已经在mapppings中添加了novel这么一个索引。

当然，通过head-master编写不是很直观，我们可以通过更加方便的工具postMan来实现上述功能。
示例操作：
我们需要通过``put``请求``127.0.0.1:9200/people``其中的people是我们要创建的索引，然后通过我们上传的json文件对创建的索引进行说明。
例如上传的的json代码如下：
```
{
	"settings":{
		"number_of_shards":3,
		"number_of_replicas":1
	},
	"mappings":{
		"man":{
			"properties":{
				"name":{
					"type":"text"
				},
				"country":{
					"type":"keyword"
				},
				"age":{
					"type":"integer"
				},
				"data":{
					"type":"date",
					"format": "yyyy-MM-dd HH:mm:ss||yyy-MM-dd||epoch_millis"
				}
			}
		},
		"woman":{
			
		}
	}
}
```
表示的是常见的people
验证方法和之前的一样。

**非结构化创建**

## 插入数据

### 指定文档ID插入
我们这里仅用postman做示例：
![指定文档ID插入](http://ovec6nnof.bkt.clouddn.com/%E6%8C%87%E5%AE%9A%E6%96%87%E6%A1%A3ID%E6%8F%92%E5%85%A5.jpg)

### 不指定文档ID插入
![不指定文档插入](http://ovec6nnof.bkt.clouddn.com/%E4%B8%8D%E6%8C%87%E5%AE%9AID%E6%8F%92%E5%85%A5.jpg)
可以看到，指定文档ID插入我们需要使用put请求方式，并且需要指定文档ID。而不指定文档ID插入则使用post请求方式，并且不需要知道哪个文档ID。

验证是否添加成功，在数据浏览中就可以看到我们插入的数据：
![验证文档是否添加成功](http://ovec6nnof.bkt.clouddn.com/%E9%AA%8C%E8%AF%81%E6%8F%92%E5%85%A5%E6%98%AF%E5%90%A6%E6%88%90%E5%8A%9F.jpg)

## 修改数据
修改数据很简单，
### 直接修改文档
![直接修改文档](http://ovec6nnof.bkt.clouddn.com/%E4%BF%AE%E6%94%B9%E6%96%87%E6%A1%A3.jpg)
### 脚本修改文档
保持要访问的地址和方式不变，将上传的json格式修改为下面的内容，即修改为script的关键字。
```
{
	"script":{
		"lang":"painless",
		"inline":"ctx._source.age+=10"
	}
}
```
或者我们也可以这么写：
```
{
	"script":{
		"lang":"painless",
		"inline":"ctx._source.age=params.age",
		"params":{
			"age":100
		}
	}
}
```
### 删除
### 删除文档
删除文档很简单，只需要简单的通过DELETE方式访问制定ID文档即可删除。
例如我们想删除man索引中文档ID为1的文档，我们通过DELETE方式访问即可。
``127.0.0.1:9200/people/man/1``
### 删除索引
删除索引非常危险，因为删除索引后，索引里面所有的内容都将被删除！！！如果我们确定我们需要删除的话，可以通过head-master来实现。
具体可以选择相应的索引 -> 动作 -> 删除索引 -> 输入删除 -> 成功删除。

或者我们直接通过delete进行访问，例如想要删除people这个索引，我们只需要通过delete方式访问``127.0.0.1:9200/people/``即可删除。

## 查询

### 简单查询
直接通过文档进行查询，请求方式：GET
例如我想查询在people索引中，类型为man，文档ID为1的文档信息的话，直接访问：
``127.0.0.1:9200/people/man/1``即可。

### 条件查询
大多数情况下，简单查询不能满足我们的需求，这个时候我们就可以通过条件查询来满足我们复杂的需求。
如果我们需要查询people索引下所有的文档，可以通过**post方法**访问``127.0.0.1:9200/people/_search``，并将body的内容设置为如下的内容：
```
{
	"query":{
		"match_all":{
			
		}
	}
}
```
当然，大多数情况我们是需要按条件查询，下面是按条件查询的示例json请求格式：
```
{
	"query":{                         //注意 这里是query关键字
		"match":{                     //要查询的的条件
			"country":"Test"
		}
	},
	"sort":[                            //查询结果后的排序
		{"date": {"order":"desc"}}      // 倒叙排序
		]
}
```


### 聚合查询
聚合查询，简单来说就是将要查询的数据组合到一起，进而汇总来自多行的信息。
例如，我想查询people中，年龄相同的聚合信息，查询的方式仍然是post，访问的地址仍然是``127.0.0.1:9200/people/_search``，不同的发送的json数据：
```
{
	"aggs":{					//关键字
		"group_by_word_age":{  	//自定义聚合的名字
			"terms":{			//关键字
				"field": "age"	//聚合的字段名称
			}
		},
		"group_by_data":{		//自定义聚合的名字
			"terms":{			//关键字
				"field": "date" //聚合字段的名称
			}
		}
	}
}
```
我这里返回的信息为：
```
{
....,
    "aggregations": {
        "group_by_data": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 923443200000,
                    "key_as_string": "1999-04-07T00:00:00.000Z",
                    "doc_count": 2
                },
                {
                    "key": -1916697600000,
                    "key_as_string": "1909-04-07T00:00:00.000Z",
                    "doc_count": 1
                }
            ]
        },
        "group_by_word_age": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 28,
                    "doc_count": 3
                }
            ]
        }
    }
}
```
当然，我们不但可以通过``terms``查询字段的聚合信息，还可以通过``status``、``min``、``max``等关键字查询字段的状态信息。


最后，补充一点就是，如果想通过ip地址访问的话，需要修改``elasticsearch.yml``文件中的``network.host:``为我们指定的Ip字段。
暂时先写这么吧~~~
----


# 参考资料
* [Elasticsearch权威指南（中文版）](https://es.xiaoleilu.com/)
* [elasticsearch官方网站](https://www.elastic.co/cn/)
* [慕课网上的教学视频](http://www.imooc.com/video/15762)
* [soJson博客](http://www.sojson.com/tag_elasticsearch_3.html)
