---
title: android端对elasticsearch进行增、删、改、查
date: 2017-10-12 16:33:08
categories: 杂烩
tags: andrid,okhttp
---

> Retrofit与okhttp共同出自于Square公司，retrofit就是对okhttp做了一层封装。把网络请求都交给给了Okhttp，我们只需要通过简单的配置就能使用retrofit来进行网络请求了，其主要作者是Android大神JakeWharton。

我们上篇学习了如何通过poastMan来模拟http请求来学习elasticsearch的基本请求操作。接下来记录的是如何代码来对我们之前配置好的elasticsearch服务器进行增删改查的基本操作。为了简单起见，我们使用的是开源库``retrofit``来实现android断的网络通信。

# 保证可访问
在正式编写代码之前，我们需要做的一件非常重要的事情就是，确保我们的手机能正确访问到elasticsearch服务器，为了简单起见，我们就实现在同一个局域网之间能正常访问即可。至于如果将elasticsearch如果通过外网访问，这里我就不做介绍了。在确保我们手机与电脑在同一个局域网之后，我们需要做的事情就很简单了，用手机浏览器访问``http://you ip:9200``,如果浏览器显示如下提示：
```
name: "msater",
cluster_name: "reoger",
cluster_uuid: "D8qVJjX5SCSh62mz9ei3og",
version: {
number: "5.6.1",
build_hash: "667b497",
build_date: "2017-09-14T19:22:05.189Z",
build_snapshot: false,
lucene_version: "6.6.1"
},
tagline: "You Know, for Search"
}
```
则说明我们的手机已经可以正常访问到eslasticsearch服务器了，这个时候我们就可以进行android端的代码编写了。如果手机浏览器提示我们无法访问，可能是我们的eslasticsearch并并没有绑定我们指定的ip地址，具体做法就是到我们eslasticseach/config/elasticsearch.yml中，修改``network.host: ``的值（如果没有就创建）为我们要访问的ip地址即可。如图：
![配置IP地址](http://ovec6nnof.bkt.clouddn.com/%E9%85%8D%E7%BD%AEip%E5%9C%B0%E5%9D%80.png)

# 访问测试
下面的代码开始就是直接通过retrofit来进行网络请求，如果这个开源库不是很熟悉的话，可以先通过[这里]()了解``retroift``的基本操作。因为例子比较简单，也可以通过本示例对``retrofit``做一个简单的了解。
通过我们手机浏览器的测试我们知道，我们只需要用代码实现访问之前测试通过的地址就可以陈宫访问，接下来我们就通过``Retrofit``来实现对Elasticsearch的访问测试。
代码很简单，如下：
```
OkHttpClient client=new OkHttpClient();
        Request request = new Request.Builder().url("http://you Ip:9200/").build();
       client.newCall(request).enqueue(new okhttp3.Callback() {
           @Override
           public void onFailure(okhttp3.Call call, IOException e) {
               Log.d("TAG",e.toString());
           }

           @Override
           public void onResponse(okhttp3.Call call, okhttp3.Response response) throws IOException {
               if(response!=null)
                Log.d("TAG",response.body().string());
               else
                   Log.d("TAG","没有返回任何的数据");
           }
       });

```
观察到，当我们运行上面的代码的时候，会打印出我们用浏览器访问的相同的输出。代表着我们的访问测试成功通过。上面ip地址需要修改成你真正的访问ip地址，本例主要通过okhttp来实现网络请求，``retrofit``是对``okhttp``进一步的封装，但是这里我们用okhttp反而更加清晰，简单。

# 创建索引
在我们前面的介绍中，我们已经知道如何通过postman这一工具来创建索引，其实就是通过``put``请求来访问``http://you ip/你要创建的索引名称``，然后通过上传json信息来确定索引的详细信息，示例的json代码如下：
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
					"format":"yyyy-MM-dd HH:mm:ss||yyy-MM-dd||epoch_millis"
				}
			}
		}
	}
}
```
下面我们来在android中创建上述的索引。整体来说，代码还是很简单，我们先看实现代码，然后进行简要的说明：
```
  String sry ="{ \"settings\": { \"number_of_shards\": 3, \"number_of_replicas\": 1 }, \"mappings\": { \"man\": { \"properties\": { \"name\": { \"type\": \"text\" }, \"country\": { \"type\": \"keyword\" }, \"age\": { \"type\": \"integer\" }, \"data\": { \"type\": \"date\", \"format\": \"yyyy-MM-dd HH:mm:ss||yyy-MM-dd||epoch_millis\" } } }, \"woman\": { \"properties\": { \"name\": { \"type\": \"text\" }, \"country\": { \"type\": \"keyword\" }, \"age\": { \"type\": \"integer\" }, \"data\": { \"type\": \"date\", \"format\": \"yyyy-MM-dd HH:mm:ss||yyy-MM-dd||epoch_millis\" } } } } }";

        RequestBody body = RequestBody.create(okhttp3.MediaType.parse("application/json; charset=utf-8"),sry);
        final Request request = new Request.Builder()
                .url("http://192.168.139.1:9200/test")
                .put(body)
                .build();
        OkHttpClient client=new OkHttpClient();
        client.newCall(request).enqueue(new okhttp3.Callback() {
            @Override
            public void onFailure(okhttp3.Call call, IOException e) {
                Log.d("TAG", "onFailure: "+e);
            }

            @Override
            public void onResponse(okhttp3.Call call, okhttp3.Response response) throws IOException {
                if(response!=null)
                    Log.d("TAG",response.body().string());
                else
                    Log.d("TAG","没有返回任何的数据");
            }
        });
```
运行上面的代码就能可以在android端创建一个索引，其实我们的代码也比较简单，就是将我们要上传的json数据当作boby一同提交给服务端就OK了。当然，这里我们的json数据是固定的，但是就对于创建索引来说，固定的json数据也能应该大部分情况了，但是添加数据就不能用固定的json数据，那么我们就不能采用这种简单粗暴的方法了，下面将会介绍如果上传动态json数据。

# 添加数据
前面文章中已经介绍过如果在postman中添加数据，那么我们现在通过自己编写代码来实现在android中向elsactisearch中添加数据，请求方式是一样的，因为实现起来非常简单，我们只介绍不指定文档ID插入。前面我们介绍到指定文档ID添加数据需要使用``put``请求方式，并且需要请求地址中表明ID，而不指定文档ID添加数据则需要用``post``请求方式，并且不需要再请求地址中表明ID。我们这里就实现用不指定文档ID插入。

通过前面的常见索引，相信大家应该知道如果将json数据上传到服务器，我们这里注重讲的书如何将上传动态json数据。
创建动态json数据，我们需要准备json格式对应的bean对象，这里我们采用``gsonFormat``插件来生成bean对象，例如我们要上传的json数据如下：
```
{
	"name": "张三",
	"country": "Englis",
	"age": 18,
	"date": "1999-03-07"
}
```
注意，这里上传的数据要与前面我们创建索引数据格式对应起来，否则不能成功添加。我们将该json对象的bean对象命名为``AddData``，源码如下：
```
public class AddData {


    private String name;
    private String country;
    private int age;
    private String date;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getDate() {
        return date;
    }

    public void setDate(String date) {
        this.date = date;
    }
}

```
创建上述的bean对象后，我们就可以通过代码来动态的修改json数据，主要的代码如下：
```
 AddData addData = new AddData();
        addData.setAge(23);
        addData.setCountry("China");
        addData.setDate("2017-09-19");
        addData.setName("DongMingZhu");//动态设置json数据

        Gson gson = new Gson();
        String dyJson = gson.toJson(addData);
        RequestBody body = RequestBody.create(okhttp3.MediaType.parse("application/json; charset=utf-8"),dyJson);
        final Request request = new Request.Builder()
                .url("http://192.168.139.1:9200/test/woman")
                .post(body)
                .build();
        OkHttpClient client=new OkHttpClient();
        client.newCall(request).enqueue(new okhttp3.Callback() {
            @Override
            public void onFailure(okhttp3.Call call, IOException e) {
                Log.d("TAG", "onFailure: "+e);
            }

            @Override
            public void onResponse(okhttp3.Call call, okhttp3.Response response) throws IOException {
                if(response!=null)
                    Log.d("TAG",response.body().string());
                else
                    Log.d("TAG","没有返回任何的数据");
            }
        });

```
运行上述的代码后，发现我们的json数据已经成功上传到服务端。简单吧~~~


# 删除数据
删除操作就是用通过DELETE访问即可，要删除指定文档，我们通过DELETE访问到指定的文档ID即可，例如我想删除test索引中man类中的id为1的文档，我的访问链接可能就是``http://127.0.0.1/9100/test/man/1`` ，如果我们想要直接删除test索引，我们可以通过delete方法访问``http://127.0.0.1/9100/test``，删除操作很简单，但是我们却要很慎重的对待他，因为我们一旦删除就无法找回。在做删除操作之前最好确保数据备份。因为实现起来很简单，仅贴出删除指定数据的代码：
```
   OkHttpClient client=new OkHttpClient();
        Request request = new Request.Builder().url("http://127.0.0.1/9100/test/man/1").delete().build();
       client.newCall(request).enqueue(new okhttp3.Callback() {
           @Override
           public void onFailure(okhttp3.Call call, IOException e) {
               Log.d("TAG",e.toString());
           }

           @Override
           public void onResponse(okhttp3.Call call, okhttp3.Response response) throws IOException {
               if(response!=null)
                Log.d("TAG",response.body().string());
               else
                   Log.d("TAG","没有返回任何的数据");
           }
       });
```


# 修改数据
 

# 查询数据