---
title: 散记
date: 2018-08-22 14:44:55
categories: reprint
tags: android,webView
---

WebView缓存原理分析和应用
================
 
发表于 2017-05-13 | 分类于 [技术](http://unclechen.github.io/2017/05/13/WebView%E7%BC%93%E5%AD%98%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90%E5%92%8C%E5%BA%94%E7%94%A8/) |[](/2017/05/13/WebView缓存原理分析和应用/#comments)

[](#一、背景 "一、背景")一、背景
====================

现在的App开发，或多或少都会用到Hybrid模式，到了WebView这边，经常会加载一些js文件（例如和WebView用来Native通信的bridge.js），而这些js文件不会经常发生变化，所以我们希望js在WebView里面加载一次之后，如果js没有发生变化，下次就不用再发起网络请求去加载，从而减少流量和资源的占用。那么有什么方式可以达到这个目的呢？先得从WebView的缓存原理入手。

[](#二、WebView的缓存类型 "二、WebView的缓存类型")二、WebView的缓存类型
==================================================

WebView主要包括两类缓存，**一类是浏览器自带的网页数据缓存**，这是所有的浏览器都支持的、由HTTP协议定义的缓存；**另一类是H5缓存**，这是由web页面的开发者设置的，H5缓存主要包括了App Cache、DOM Storage、Local Storage、Web SQL Database 存储机制等，这里我们主要介绍App Cache来缓存js文件。

[](#三、浏览器自带的网页数据缓存 "三、浏览器自带的网页数据缓存")三、浏览器自带的网页数据缓存
==================================================

[](#1-工作原理 "1.工作原理")1.工作原理
--------------------------

浏览器缓存机制是通过HTTP协议Header里的Cache-Control（或Expires）和Last-Modified（或 Etag）等字段来控制文件缓存的机制。关于这几个字段的作用和浏览器的缓存更新机制，大家可以看看这两篇文章([H5 缓存机制浅析 移动端 Web 加载性能优化](http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=267)，[Android：手把手教你构建 WebView 的缓存机制 & 资源预加载方案](http://www.jianshu.com/p/5e7075f4875f))，里面有详细的介绍。下面从我实际应用的角度，介绍一下通常会在HTTP协议中遇到的Header。

这两个字段是**接收响应时，浏览器决定文件是否需要被缓存；或者需要加载文件时，浏览器决定是否需要发出请求**的字段。

*   **Cache-Control:max-age=315360000，**这表示缓存时长为315360000秒。如果315360000秒内需要再次请求这个文件，那么浏览器不会发出请求，直接使用本地的缓存的文件。这是HTTP/1.1标准中的字段。
    
*   **Expires: Thu, 31 Dec 2037 23:55:55 GMT，**这表示这个文件的过期时间是2037年12月31日晚上23点55分55秒，在这个时间之前浏览器都不会再次发出请求去获取这个文件。这是HTTP/1.0中的字段，如果客户端和服务器时间不同步会导致缓存出现问题，因此才有了上面的Cache-Control，当它们同时出现在HTTP Response的Header中时，Cache-Control优先级更高。
    

下面两个字段是**发起请求时，服务器决定文件是否需要更新**的字段。

*   **Last-Modified:Wed, 28 Sep 2016 09:24:35 GMT，**这表示这个文件最后的修改时间是2016年9月28日9点24分35秒。这个字段对于浏览器来说，会在下次请求的时候，作为Request Header的If-Modified-Since字段带上。例如浏览器缓存的文件已经超过了Cache-Control（或者Expires），那么需要加载这个文件时，就会发出请求，请求的Header有一个字段为`If-Modified-Since：Wed, 28 Sep 2016 09:24:35 GMT`，服务器接收到请求后，会把文件的Last-Modified时间和这个时间对比，如果时间没变，那么浏览器将返回`304 Not Modified`给浏览器，且content-length肯定是0个字节。如果时间有变化，那么服务器会返回`200 OK`，并返回相应的内容给浏览器。
    
*   **ETag:”57eb8c5c-129”，**这是文件的特征串。功能同上面的Last-Modified是一样的。只是在浏览器下次请求时，ETag是作为Request Header中的`If-None-Match:"57eb8c5c-129"`字段传到服务器。服务器和最新的文件特征串对比，如果相同那么返回`304 Not Modified`，不同则返回`200 OK`。当ETag和Last-Modified同时出现时，任何一个字段只要生效了，就认为文件是没有更新的。
    

[](#2-WebView如何设置才能支持上面的协议 "2.WebView如何设置才能支持上面的协议")2.WebView如何设置才能支持上面的协议
--------------------------------------------------------------------------

由上面的介绍可知，只要是个主流的、合格的浏览器，都应该能够支持HTTP协议层面的这几个字段。这不是我们开发者可以修改的，也不是我们应该修改的配置。在Android上，我们的WebView也支持这几个字段。但是我们可以通过代码去**设置WebView的Cache Mode**，而使得协议生效或者无效。WebView有下面几个Cache Mode：

*   LOAD\_CACHE\_ONLY: 不使用网络，只读取本地缓存数据。
*   LOAD_DEFAULT: 根据cache-control决定是否从网络上取数据。
*   LOAD\_CACHE\_NORMAL: API level 17中已经废弃，从API level 11开始作用同LOAD_DEFAULT模式
*   LOAD\_NO\_CACHE: 不使用缓存，只从网络获取数据。
*   LOAD\_CACHE\_ELSE_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据。本地没有缓存时才从网络上获取。

设置WebView缓存的Cache Mode示例代码如下：

1

2

WebSettings settings = webView.getSettings();

settings.setCacheMode(WebSettings.LOAD_DEFAULT);

网上很多人都说根据网络条件去选择Cache Mode，当有网络时，设置为LOAD\_DEFAULT，当没有网络时设置为LOAD\_CACHE\_ELSE\_NETWORK。但是在我的业务中，js文件的更新都是非覆盖式的更新，也就是时候每次改变js文件的时候，文件的url地址一定会发生变化，所以我希望浏览器能够缓存下来js，并且一直使用它，那么我就给它只设置为LOAD\_CACHE\_ELSE\_NETWORK。当然如果你要是可以改js的cdn服务器的Cache-Control字段，那也行啊，用LOAD\_DEFAULT就ok了。至于文件是应该采用覆盖式or非覆盖式的更新，不是我今天要讨论的内容，在web前端领域，这是一个可以聊聊的topic。

> 关于iOS的WebView，我同事在实际测试的时候竟然发现，控制文件缓存的Response Header是Expires字段。。而且iOS无法针对整个WebView设置Cache Mode，只能针对每一个URLRequest去设置。。后续有机会要学习一下iOS那块的情况。

[](#3-在手机里面的存储路径 "3.在手机里面的存储路径")3.在手机里面的存储路径
--------------------------------------------

浏览器默认缓存下来的文件是怎么被存储到了哪里呢？这个问题在接触到WebView以来，就一直是一个谜题。这次由于工作的需要，我特意root了两台手机，一台红米1（Android 4.4）和一台小米4c（Android 5.1），在root高系统版本（6.0和7.1）的两台Nexus都以失败告终之后，我决定还是先看看4.4和5.1系统上，WebView自带的缓存存到了哪里。

首先，不用思考就知道，这些文件一定是在**/data/data/包名/**目录下，在我之前的一篇博客里面提到过，这是每一个应用自己的内部存储目录。

接着，我们打开终端，使用adb连接手机，然后按照下面命令操作一下。

```
// 1.先进入shell

adb shell

// 2.开启root账号 

su

// 3.修改文件夹权限

chmod 777 data/data/你的应用包名/

// 4.修改子文件夹的权限，因为Android命令行不支持向Linux那样的-R命令实现递归式的chmod。。。

chmod 777 data/data/你的应用包名/*

// 5.所以如果你对应用目录层级更深，你就要进一步地chmod。。。

chmod 777 data/data/你的应用包名/*/*

// 6.直到终端里提示你说，no such file or directory时，说明chmod完了，所有的内部存储里面的文件夹和文件都可以看到了，如果大家有更好的方法请一定告诉我，多谢了~
```

*   Android 4.4的目录：`/data/data/包名/app_webview/cache/`，如下图所示的第二个文件夹。

[![Android4.4系统WebView自带缓存路径](https://ww4.sinaimg.cn/large/006tNc79ly1ffjvarjyijj30jg05k0yr.jpg)](https://ww4.sinaimg.cn/large/006tNc79ly1ffjvarjyijj30jg05k0yr.jpg)

可能你注意到了，第一个文件夹是叫Application Cache，我们后面再说它。

*   Android 5.1的目录：`/data/data/包名/cache/org.chromium.android_webview/`下面，如下图所示。

[![](https://ww2.sinaimg.cn/large/006tNc79ly1ffjvztg93zj30jg0a247x.jpg)](https://ww2.sinaimg.cn/large/006tNc79ly1ffjvztg93zj30jg0a247x.jpg)

但是在5.1系统上，`/data/data/包名/app_webview/`文件夹依然存在，只是4.4系统上面存储WebView自带缓存的`app_webview/cache`文件夹不再存在了（注意下App Cache目录还在），如下图所示。

[![Android5.1系统WebView自带缓存路径](https://ww3.sinaimg.cn/large/006tNc79ly1ffjw46ygoqj30jg06443v.jpg)](https://ww3.sinaimg.cn/large/006tNc79ly1ffjw46ygoqj30jg06443v.jpg)

综上所述，WebView自带的浏览器协议支持的缓存，在不同的系统版本上，位置是不一样的。也许除了我root过的4.4、5.1以外，其他版本系统的WebView自带缓存还可能存在于不同的目录里面。

另外一个是关于**缓存文件的存储格式和索引格式**，在不同的手机上可能也有差别，因为之前看到网上的人都说有叫**webview.db**或者**webviewCache.db**的文件，这个文件呢，还不是在`app_webview/cache`或者`org.chromium.android_webview`下面，而是在`/data/data/包名/database/`里面。但是，我这两台root过的手机都没有看到这种文件，而且我把`/data/data/包名/`下面所有的db文件都打开看了，并没有发现有存储url记录的table。。

实际上，以5.1系统为例，我看到了`/data/data/包名/cache/org.chromium.android_webview/`下面有叫**index**和**/index-dir/the-real-index**的文件，以及一堆名称为**md5+下划线+数字**的文件，上面的图中也可以看得到，这块的原理仍然有些疑问，也希望专业的大神可以解答一下。

[](#四、H5的缓存 "四、H5的缓存")四、H5的缓存
=============================

讲完了WebView自带的缓存，下面讲一下H5里面的App Cache。这个Cache是由开发Web页面的开发者控制的，而不是由Native去控制的，但是Native里面的WebView也需要我们做一下设置才能支持H5的这个特性。

[](#1-工作原理-1 "1.工作原理")1.工作原理
----------------------------

写Web页面代码时，指定manifest属性即可让页面使用App Cache。通常html页面代码会这么写：

```

<html manifest="xxx.appcache">

</html>
```
xxx.appcache文件用的是相对路径，这时appcache文件的路径是和页面一样的。也可以使用的绝对路径，但是域名要保持和页面一致。

完整的xxx.appcache文件一般包括了3个section，基本格式如下：
```

CACHE MANIFEST

\# 2017-05-13 v1.0.0

/bridge.js

NETWORK:

*

FALLBACK:

/404.html
```
*   CACHE MANIFEST下面文件就是要被浏览器缓存的文件
*   NETWORK下面的文件就是要被加载的文件
*   FALLBACK下面的文件是目标页面加载失败时的显示的页面

**AppCache工作的原理：**当一个设置了manifest文件的html页面被加载时，CACHE MANIFEST指定的文件就会被缓存到浏览器的App Cache目录下面。当下次加载这个页面时，会首先应用通过manifest已经缓存过的文件，然后发起一个加载xxx.appcache文件的请求到服务器，如果xxx.appcache文件没有被修改过，那么服务器会返回`304 Not Modified`给到浏览器，如果xxx.appcache文件被修改过，那么服务器会返回`200 OK`，并返回新的xxx.appcache文件的内容给浏览器，浏览器收到之后，再把新的xxx.appcache文件中指定的内容加载过来进行缓存。

可以看到，AppCache缓存需要在每次加载页面时都发出一个xxx.appcache的请求去检查manifest文件是不是有更新（byte by byte）。根据这篇文章（[H5 缓存机制浅析 移动端 Web 加载性能优化](http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=267)）的介绍，AppCache有一些坑的地方，且官方已经不推荐使用了，但目前主流的浏览器依然是支持的。文章里主要提到下面这些坑：

*   要更新缓存的文件，需要更新包含它的 manifest 文件，那怕只加一个空格。常用的方法，是修改 manifest 文件注释中的版本号。如：# 2012-02-21 v1.0.0
*   被缓存的文件，浏览器是先使用，再通过检查 manifest 文件是否有更新来更新缓存文件。这样缓存文件可能用的不是最新的版本。
*   在更新缓存过程中，如果有一个文件更新失败，则整个更新会失败。
*   manifest 和引用它的HTML要在相同 HOST。
*   manifest 文件中的文件列表，如果是相对路径，则是相对 manifest 文件的相对路径。
*   manifest 也有可能更新出错，导致缓存文件更新失败。
*   没有缓存的资源在已经缓存的 HTML 中不能加载，即使有网络。例如：\[url=\][http://appcache-demo.s3-website-us-east-1.amazonaws.com/without-network/\[/url](http://appcache-demo.s3-website-us-east-1.amazonaws.com/without-network/[/url)\]
*   manifest 文件本身不能被缓存，且 manifest 文件的更新使用的是浏览器缓存机制。所以 manifest 文件的 Cache-Control 缓存时间不能设置太长。

[](#2-WebView如何设置才能支持AppCache "2.WebView如何设置才能支持AppCache")2.WebView如何设置才能支持AppCache
-----------------------------------------------------------------------------------

WebView默认是没有开启AppCache支持的，需要添加下面这几行代码来设置：

```

WebSettings webSettings = webView.getSettings();

webSettings.setAppCacheEnabled(true);

String cachePath = getApplicationContext().getCacheDir().getPath(); // 把内部私有缓存目录'/data/data/包名/cache/'作为WebView的AppCache的存储路径

webSettings.setAppCachePath(cachePath);

webSettings.setAppCacheMaxSize(5 * 1024 * 1024);
```

注意：WebSettings的setAppCacheEnabled和setAppCachePath都必须要调用才行。

[](#3-存储AppCache的路径 "3.存储AppCache的路径")3.存储AppCache的路径
-----------------------------------------------------

按照Android SDK的API说明，setAppCachePath是可以用来设置AppCache路径的，但是我实际测试发现，不管你怎么设置这个路径，设置到应用自己的内部私有目录还是外部SD卡，都无法生效。AppCache缓存文件最终都会存到`/data/data/包名/app_webview/cache/Application Cache`这个文件夹下面，在上面的Android 4.4和5.1系统目录截图可以看得到，**但是如果你不调用setAppCachePath方法，WebView将不会产生这个目录**。这里有点让我觉得奇怪，我猜测可能从某一个系统版本开始，为了缓存文件的完整性和安全性考虑，SDK实现的时候就吧AppCache缓存目录设置到了内部私有存储。

[](#五、总结 "五、总结")五、总结
====================

[](#相同点 "相同点")相同点
-----------------

WebView自带的缓存和AppCache都是可以用来做文件级别的缓存的，基本上比较好地满足对于非覆盖式的js、css等文件更新。

[](#不同点 "不同点")不同点
-----------------

*   WebView自带的缓存是是协议层实现的（浏览器内核标准实现，开发者无法改变）；而AppCache是应用层实现的。
*   WebView的缓存目录在不同系统上可能是不同的；而对于AppCache而言，AppCache的存储路径虽然有方法设置，但是最终都存储到了一个固定的内部私有目录下。
*   WebView自带的缓存可以在缓存生效的时候不用再发HTTP请求；而AppCache一定会发出一个manifest文件的请求。
*   WebView自带的缓存可以通过设置CacheMode来改变WebView的缓存机制；而AppCache的缓存策略是由manifest文件控制的，也就是说是由web页面开发者控制的。

最后说一下，其实很多时候，这两类缓存是共同在工作的，当manifest文件没有控制某些资源加载时，例如我上面写的xxx.appcache文件里，NETWORK section下面用的是*号，意思是所有不缓存的文件都要去网络加载。此时，这些资源就会走到WebView自带的缓存机制去，结合WebView的CacheMode，我们实际上对这些文件进行了一次WebView自带的缓存。搞清楚这两类缓存的原理有利于我们更好的设计自己的页面和App，尽可能减少网络请求，提高App运行效率。

*   **本文作者：**unclechen
*   **本文链接：** [2017/05/13/WebView缓存原理分析和应用/](/2017/05/13/WebView缓存原理分析和应用/ "WebView缓存原理分析和应用")
*   **版权声明：** 本文采用[知识共享 署名-非商业性使用-相同方式共享 4.0 国际 许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。转载请注明出处！

[\# Android](/tags/Android/) [\# WebView](/tags/WebView/)

[python利用beautifulsoup+selenium自动翻页抓取网页内容](/2016/12/11/python利用beautifulsoup+selenium自动翻页抓取网页内容/ "python利用beautifulsoup+selenium自动翻页抓取网页内容")

[使用React.js开发Chrome插件](/2017/06/16/使用ReactJS开发Chrome插件/ "使用React.js开发Chrome插件")