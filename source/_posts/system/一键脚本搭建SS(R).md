---
title: 一键脚本搭建SS/搭建SSR服务并开启BBR加速
date: 2018-07-30 15:44:55
categories: 转载
tags: ss,ssr
---
# 一键脚本搭建SS/搭建SSR服务并开启BBR加速

 2018年1月20日 497条评论 127,696次阅读 293人点赞

自己写的一键搭建shadowsocks/搭建shadowsocksR的shell脚本，一键脚本适用[Vultr](https://www.flyzy2005.win/vps/vultr-deploy/)上的和[搬瓦工](https://www.flyzy2005.win/vps/bandwagon-coupon-buy/)所有机型（CentOS、Ubuntu、Debian），搭建ss服务器支持所有**客户端类型**，本机你是iOS，Android，Windows，Mac，或者是Linux，搭建ss/ssr都是适用的科学上网方式。一键脚本搭建SS/SSR服务器，**绝对没有任何问题**，任何问题欢迎留言。一键脚本内容包括一键搭建shadowsocks/一键搭建shadowsocksR+一键开启bbr加速，适合新手小白。

**纯新手**也可以搭建ss/ssr，录了个**视频教程**，不想看文字的可以看视频，或者结合起来一起看：[搭建ss视频教程](https://www.flyzy2005.win/fan-qiang/shadowsocks/install-shadowsocks-in-one-command/#vultrss)。

文章目录

*   [什么是shadowsocks](#shadowsocks "什么是shadowsocks")
*   [一键脚本搭建ss/ssr支持系统版本](#ssssr "一键脚本搭建ss/ssr支持系统版本")
*   [代理服务器购买](#i "代理服务器购买")
*   [连接远程Linux服务器](#Linux "连接远程Linux服务器")
*   [一键搭建SS/搭建SSR服务](#SSSSR "一键搭建SS/搭建SSR服务")
    *   [一键搭建shadowsocks](#shadowsocks-2 "一键搭建shadowsocks")
    *   [一键搭建shadowsocksR](#shadowsocksR "一键搭建shadowsocksR")
*   [一键开启BBR加速](#BBR "一键开启BBR加速")
*   [客户端搭建shadowsocks/shadowsockR代理实现科学上网](#shadowsocksshadowsockR "客户端搭建shadowsocks/shadowsockR代理实现科学上网")
    *   [客户端搭建ss代理](#ss "客户端搭建ss代理")
    *   [客户端搭建ssr代理](#ssr "客户端搭建ssr代理")
*   [vultr搭建ss视频教程](#vultrss "vultr搭建ss视频教程")
*   [一键脚本更新日志](#i-2 "一键脚本更新日志")

## **什么是shadowsocks**

shadowsocks可以指一种SOCKS5的加密传输协议，也可以指基于这种加密协议的各种数据传输包。

**shadowsocks实现科学上网原理？**shadowsocks正常工作需要服务器端和客户端两端合作实现，首先，客户端（本机）通过ss（shadowsocks）对正常的访问请求进行SOCK5加密，将加密后的访问请求传输给ss服务器端，服务器端接收到客户端的加密请求后，解密得到原始的访问请求，根据请求内容访问指定的网站（例如Google，YouTube，Facebook，instagram等），得到网站的返回结果后，再利用SOCKS5加密并返回给客户端，客户端通过ss解密后得到正常的访问结果，于是就可以实现你直接访问该网站的“假象”。

**为什么选择shadowsocks？**不限终端（安卓，苹果，Windows，Mac都可用），流量便宜（服务器500G只要15元），方便（一键脚本，不需要专业知识）。

**为什么要自己搭建ss/ssr？**你也许会觉得买ss服务也很方便，但是你得要考虑以下几个问题。首先，买的ss服务，限制很多，终端可能只能同时在线2个，每个月就一点点流量可能价格却不便宜，有时候还被别人做手脚，流量跑的贼快；其次，别人收钱跑路怎么办？很多这种情况的；更重要的是，如第一个问题中描述的shadowsocks原理，如果有心人做了一点手脚，是可以得到你的访问记录的；而自己搭建ss/ssr服务，一键脚本也就10来分钟就可以搞定。

## **一键脚本搭建ss/ssr支持系统版本**

脚本系统支持：CentOS 6+，Debian 7+，Ubuntu 12+

注：这个脚本支持的系统版本是指ss服务器的版本（都没看过也没关系，不影响搭建），你本机是Windows、Mac、Linux，或者你想用手机端搭建ss/ssr服务器，安卓和苹果，都是可以的。

## **代理服务器购买**

作为跳板的代理服务器推荐[Vultr](https://www.flyzy2005.win/vps/vultr-deploy/)和[搬瓦工](https://www.flyzy2005.win/vps/bandwagon-coupon-buy/)，一是因为本脚本在这两家的所有VPS都做了测试，二是因为都是老牌VPS服务商，不怕跑路。

推荐你们使用Vultr：[优惠注册链接](https://www.flyzy2005.win/goto/vultr)，购买图解与节点推荐可以参考[Vultr购买图解步骤](https://www.flyzy2005.win/vps/vultr-deploy/)，最低月付2.5刀，也是目前博主自用以及运行小站的VPS，月付方便，随时重置。

对于宽带是电信或者是联通的用户，可以试一下搬瓦工的CN2**电信/联通直连线路**（季付/半年付/年付），**GT线路**详情可以参考[搬瓦工洛杉矶CN2 GT线路测评](https://www.flyzy2005.win/vps/bandwagon-cn2/)。如果想体验最优的**GIA线路**，也可以尝试搬瓦工CN2 GIA线路（**强力推荐**，效果爆炸，全程CN2，年付平均下来一个月30元左右，晚上高峰时期线路也不拥堵。2018年5月15日正式启动，[搬瓦工洛杉矶CN2 GIA线路测评](https://www.flyzy2005.win/vps/bandwagonhost-cn2-gia/)），当然你也可以直接用[Vultr](https://www.flyzy2005.win/vps/vultr-deploy/)，搬瓦工暂时不支持月付。

Vultr和搬瓦工上的所有机型是绝对可以一键脚本搭建shadowsocks/搭建shadowsocksR+开启bbr加速成功的，任何问题**欢迎留言**~

## **连接远程Linux服务器**

购买完成后根据[Windows通过Xshell连接Linux](https://www.flyzy2005.win/tech/linux/xshell-connect-linux/)或者[Mac通过Terminal远程连接Linux](https://www.flyzy2005.win/tech/linux/mac-connect-vps-linux/)即可。

你如果身边没有电脑，一定要搞什么手机搭建ss服务器 ![:symbols:](https://www.flyzy2005.win/wp-content/themes/Kratos/images/smilies/symbols.png) 也是可以的，毕竟一键脚本只需要复制几行脚本命令就行了。iOS用户可以使用Termius这个工具，直接在App Store下载就行。Android没有用过，反正能ssh连接的软件就行~

## **一键搭建SS/搭建SSR服务**

注意，shadowsocks/shadowsocksR这两个**只需要搭建一个**就可以了！！！！SS与SSR之间的比较一直是各有各的说法，王婆卖瓜自卖自夸。我用的是SS，因为SS的iOS版本比较容易下载，并且被没有觉得ss容易被探查到~

### **一键搭建shadowsocks**

在[购买VPS](https://www.flyzy2005.win/vps/vultr-deploy/)并用[Xshell连接上你刚购买的VPS](https://www.flyzy2005.win/tech/linux/xshell-connect-linux/)后，你将看到如下图所示的界面：

![vutlr-connect-result](https://www.flyzy2005.win/wp-content/uploads/2018/01/vutlr-connect-result.png)

如红框中所示，root@vult（root@ubuntu）说明已经连接成功了，之后你只需要**在绿色光标处直接复制以下代码**就可以了（直接复制即可，如每段代码下方截图中所示）。

1.下载一键搭建ss脚本文件（直接在绿色光标处复制该行命令回车即可，**只需要执行一次**，卸载ss后也不需要重新下载）

 <textarea class="crayon-plain print-no" style="tab-size: 4; font-size: 12px !important; line-height: 15px !important; z-index: 0; opacity: 0; overflow: hidden;" readonly="readonly" wrap="soft" data-settings="dblclick">git clone https://github.com/flyzy2005/ss-fly</textarea>

 ``git clone https://github.com/flyzy2005/ss-fly``

 
![shadowsocks-ss-fly-clone](https://www.flyzy2005.win/wp-content/uploads/2018/01/shadowsocks-ss-fly-clone.png)

如果提示`bash: git: command not found`，则先安装git：
```
 Centos执行这个： yum -y install git
 Ubuntu/Debian执行这个： apt-get -y install git
```


2.运行搭建ss脚本代码

```
 ss-fly/ss-fly.sh -i flyzy2005.com 1024

```

其中**flyzy2005.com**换成你要设置的shadowsocks的密码即可（这个**flyzy2005.com**就是你ss的密码了，是需要填在客户端的密码那一栏的），密码随便设置，最好**只包含字母+数字**，一些特殊字符可能会导致冲突。而第二个参数**1024**是**端口号**，也可以不加，不加默认是1024~（**举个例子**，脚本命令可以是ss-fly/ss-fly.sh -i qwerasd，也可以是ss-fly/ss-fly.sh -i qwerasd 8585，后者指定了服务器端口为8585，前者则是默认的端口号1024，两个命令设置的ss密码都是qwerasd）：

![shadowsocks-install](https://www.flyzy2005.win/wp-content/uploads/2018/01/shadowsocks-install.png)

界面如下就表示一键搭建ss成功了：

![ss-fly-success-new](https://www.flyzy2005.win/wp-content/uploads/2018/01/ss-fly-success-new.png)

注：如果需要改密码或者改端口，只需要重新**再执行一次搭建ss脚本代码**就可以了，或者修改`/etc/shadowsocks.json`这个配置文件。

3.相关ss操作
```

 修改配置文件：vim /etc/shadowsocks.json

 停止ss服务：ssserver -c /etc/shadowsocks.json -d stop

 启动ss服务：ssserver -c /etc/shadowsocks.json -d start

 重启ss服务：ssserver -c /etc/shadowsocks.json -d restart

```
4.卸载ss服务

```
 ss-fly/ss-fly.sh -uninstall
```

### **一键搭建shadowsocksR**

**再次提醒**，如果安装了SS，就不需要再安装SSR了，如果要改装SSR，请按照上一部分内容的教程先卸载SS！！！

1.下载一键搭建ssr脚本（**只需要执行一次**，卸载ssr后也不需要重新执行）

`git clone https://github.com/flyzy2005/ss-fly`，此步骤与一键搭建ss一致，可以参考：[下载脚本](https://www.flyzy2005.win/fan-qiang/shadowsocks/install-shadowsocks-in-one-command/#shadowsocks)

2.运行搭建ssr脚本代码

```
 ss-fly/ss-fly.sh -ssr
```

![ss-fly-insall-ssr](https://www.flyzy2005.win/wp-content/uploads/2018/01/ss-fly-insall-ssr.png)

3.输入对应的参数

执行完上述的脚本代码后，会进入到输入参数的界面，包括服务器端口，密码，加密方式，协议，混淆。可以直接输入回车选择默认值，也可以输入相应的值选择对应的选项：

![ss-fly-ssr-options](https://www.flyzy2005.win/wp-content/uploads/2018/01/ss-fly-bbr-options.png)

全部选择结束后，会看到如下界面，就说明搭建ssr成功了：

```

 Congratulations, ShadowsocksR server install completed!

 Your Server IP        :你的服务器ip

 Your Server Port      :你的端口

 Your Password :你的密码

 Your Protocol :你的协议

 Your obfs :你的混淆

 Your Encryption Method:your_encryption_method

 Welcome to visit:https://shadowsocks.be/9.html

 Enjoy it!

```

4.相关操作ssr命令
 ```
 启动：/etc/init.d/shadowsocks start

 停止：/etc/init.d/shadowsocks stop

 重启：/etc/init.d/shadowsocks restart

 状态：/etc/init.d/shadowsocks status

 配置文件路径：/etc/shadowsocks.json

 日志文件路径：/var/log/shadowsocks.log

 代码安装目录：/usr/local/shadowsocks

```
5.卸载ssr服务
```
 ./shadowsocksR.sh uninstall
```

## **一键开启BBR加速**

BBR是Google开源的一套内核加速算法，可以让你搭建的shadowsocks/shadowsocksR速度上一个台阶，本一键搭建ss/ssr脚本支持一键升级最新版本的内核并开启BBR加速。

BBR支持4.9以上的，如果低于这个版本则会自动下载最新内容版本的内核后开启BBR加速并重启，如果高于4.9以上则自动开启BBR加速，执行如下脚本命令即可自动开启BBR加速：

 ```
 ss-fly/ss-fly.sh -bbr
```

![ss-fly-bbr-success-new](https://www.flyzy2005.win/wp-content/uploads/2018/01/ss-fly-bbr-success-new.png)

装完后需要重启系统，输入y即可立即重启，或者之后输入`reboot`命令重启。

判断BBR加速有没有开启成功。输入以下命令：

```
 sysctl net.ipv4.tcp_available_congestion_control
```

如果返回值为：

```
 net.ipv4.tcp_available_congestion_control = bbr cubic reno
```

后面有bbr，则说明已经开启成功了。

## **客户端搭建shadowsocks/shadowsockR代理实现科学上网**

### **客户端搭建ss代理**

各种客户端版本下载地址：[各版本shadowsocks客户端下载地址](https://www.flyzy2005.win/fan-qiang/shadowsocks/ss-clients-download/)

以Windows为例（[shadowsocks电脑版（windows）客户端配置教程](https://www.flyzy2005.win/fan-qiang/shadowsocks/shadowsocks-pc-windows-config/)）：

![shadowsocks-pc-windows](https://www.flyzy2005.win/wp-content/uploads/2018/01/shadowsocks-pc-windows.png)

在状态栏右击shadowsocks，勾选**开机启动**和**启动系统代理**，在**系统代理模式**中选择**PAC模式**，**服务器**->**编辑服务器**，一键安装shadowsocks的脚本默认服务器端口是1024，加密方式是aes-256-cfb，密码是你设置的密码，ip是你自己的VPS ip，保存即可~

**PAC模式**是指国内可以访问的站点直接访问，不能直接访问的再走shadowsocks代理~

OK！一键脚本搭建shadowsocks完毕！科学上网吧，兄弟！[Google](https://www.flyzy2005.win/go/go.php?url=https://www.google.com/)

### **客户端搭建ssr代理**

各种客户端版本下载地址：[各版本SS客户端&SSR客户端官方下载地址](https://www.flyzy2005.win/fan-qiang/shadowsocksr/ss-ssr-clients/)

以Windows为例：

![ssr-pc-windows-config](https://www.flyzy2005.win/wp-content/uploads/2018/01/ssr-pc-windows-config.png)

在状态栏右击shadowsocksR，在**系统代理模式**中选择**PAC模式**，再左击两次状态栏的图标打开编辑服务器界面，如上图所示，按照自己的服务器配置填充内容**，**保存即可~

**PAC模式**是指国内可以访问的站点直接访问，不能直接访问的再走shadowsocksR代理~

OK！一键脚本搭建shadowsocksR完毕！科学上网吧，兄弟！[Google](https://www.flyzy2005.win/go/go.php?url=https://www.google.com/)

## **vultr搭建ss视频教程**

应读者要求录了个视频教程，如果你觉得这些文字还不够生动，不够清楚的话，可以看一下视频教程。

视频获取方式：关注微信公众号**flyzy小站**，发送**视频**即可获得。

![flyzy小站](https://www.flyzy2005.win/wp-content/uploads/2017/12/flyzy.jpg)

## **一键脚本更新日志**

一键脚本源码：[一键搭建shadowoscks/搭建shadowsocksR并开启bbr内核加速](https://www.flyzy2005.win/go/go.php?url=https://github.com/flyzy2005/ss-fly)

2018年1月20日，上传一键安装shadowsocks脚本

2018年1月24日，添加升级内核并开启BBR加速功能

2018年3月28日，将升级内核&&开启BBR加速集成在一个命令中

2018年4月4日，添加一键安装shadowsocksR功能（调用的teddysun大大的[一键搭建SSR脚本](https://www.flyzy2005.win/go/go.php?url=https://shadowsocks.be/9.html) ![:eek:](https://www.flyzy2005.win/wp-content/themes/Kratos/images/smilies/eek.png) ）

2018年4月27日，支持Ubuntu/CentOS/Debian

2018年5月29日，支持[一键安装V2Ray](https://www.flyzy2005.win/v2ray/how-to-build-v2ray/)

关注公众号**flyzy小站**，上面有一些搭建shadowsocks**常见问题**的总结~如果还是不行，欢迎在公众号留言~

**声明**：本文只作为技术分享，请遵守相关法律，严禁做违法乱纪的事情！

Telegram频道已经开通，关注[flyzythink](https://www.flyzy2005.win/go/?https://t.me/flyzythink)，随手分享正能量，了解VPS优惠与补货
Telegram群组已经开通，加入[flyzy小站](https://www.flyzy2005.win/go/?https://t.me/flyzyxiaozhan)，FREE TO TALK
QQ群开通：780593286 [![flyzy小站](//pub.idqqimg.com/wpa/images/group.png "flyzy小站")](https://www.flyzy2005.win/go/?url=https://shang.qq.com/wpa/qunwpa?idkey=1e8e1c9e5c8056b05aaaa5e34f6dc9faa7b655610cc790884043b738ceebb753)

### 原创声明

该日志由 [flyzy小站](https://www.flyzy2005.win "flyzy小站") 于2018年01月20日发表在[shadowsocks](https://www.flyzy2005.win/fan-qiang/shadowsocks/)分类下， 你可以[发表评论](#respond)，并在保留[原文地址](https://www.flyzy2005.win/fan-qiang/shadowsocks/install-shadowsocks-in-one-command/)及作者的情况下转载到你的网站或博客。
转载请注明: [一键脚本搭建SS/搭建SSR服务并开启BBR加速 | flyzy小站](https://www.flyzy2005.win/fan-qiang/shadowsocks/install-shadowsocks-in-one-command/ "本文固定链接 https://www.flyzy2005.win/fan-qiang/shadowsocks/install-shadowsocks-in-one-command/")
关键字: [bbr一键](https://www.flyzy2005.win/tag/bbr%e4%b8%80%e9%94%ae/), [shadowsock](https://www.flyzy2005.win/tag/shadowsock/), [ss/ssr脚本](https://www.flyzy2005.win/tag/ss-ssr%e8%84%9a%e6%9c%ac/), [一键搭建ss脚本](https://www.flyzy2005.win/tag/%e4%b8%80%e9%94%ae%e6%90%ad%e5%bb%bass%e8%84%9a%e6%9c%ac/), [一键脚本搭建ssr](https://www.flyzy2005.win/tag/%e4%b8%80%e9%94%ae%e8%84%9a%e6%9c%ac%e6%90%ad%e5%bb%bassr/), [科学上网](https://www.flyzy2005.win/tag/fan-qiang/)

 ![weinxin](https://www.flyzy2005.win/wp-content/uploads/2017/12/flyzy.jpg)
 **我的微信公众号**

 "flyzy小站"：关注VPS测评与优惠，玩机教程，技术分享，留言及时回复


打开微信“扫一扫”，打开网页后点击屏幕右上角分享按钮

 [bbr一键](https://www.flyzy2005.win/tag/bbr%e4%b8%80%e9%94%ae/) [shadowsock](https://www.flyzy2005.win/tag/shadowsock/) [ss/ssr脚本](https://www.flyzy2005.win/tag/ss-ssr%e8%84%9a%e6%9c%ac/) [一键搭建ss脚本](https://www.flyzy2005.win/tag/%e4%b8%80%e9%94%ae%e6%90%ad%e5%bb%bass%e8%84%9a%e6%9c%ac/) [一键脚本搭建ssr](https://www.flyzy2005.win/tag/%e4%b8%80%e9%94%ae%e8%84%9a%e6%9c%ac%e6%90%ad%e5%bb%bassr/) [科学上网](https://www.flyzy2005.win/tag/fan-qiang/)

 [< 上一篇](https://www.flyzy2005.win/fan-qiang/shadowsocks/ss-clients-download/ "各版本shadowsocks客户端下载地址")

 [下一篇 >](https://www.flyzy2005.win/fan-qiang/shadowsocks/free-ss-account/ "科学上网ss账号分享")

### 相关文章

*   [一键脚本搭建V2Ray V2Ray配置与优化](https://www.flyzy2005.win/v2ray/how-to-build-v2ray/ "一键脚本搭建V2Ray V2Ray配置与优化")
*   [shadowsocks无法打开谷歌学术或出现验证码](https://www.flyzy2005.win/tech/shadowsocks-google-scholar/ "shadowsocks无法打开谷歌学术或出现验证码")
*   [shadowsocks电脑版（Mac）客户端配置教程](https://www.flyzy2005.win/fan-qiang/shadowsocks/shadowsocks-mac-os-config/ "shadowsocks电脑版（Mac）客户端配置教程")
*   [shadowsocks手机版（Android/安卓）客户端影梭配置教程](https://www.flyzy2005.win/fan-qiang/shadowsocks/shadowsocks-android-config/ "shadowsocks手机版（Android/安卓）客户端影梭配置教程")
*   [shadowsocks手机版（iOS/苹果）客户端配置教程](https://www.flyzy2005.win/fan-qiang/shadowsocks/shadowsocks-iphone-ios-config/ "shadowsocks手机版（iOS/苹果）客户端配置教程")
*   [shadowsocks电脑版（windows）客户端配置教程](https://www.flyzy2005.win/fan-qiang/shadowsocks/shadowsocks-pc-windows-config/ "shadowsocks电脑版（windows）客户端配置教程")
*   [各版本shadowsocks客户端百度云下载地址](https://www.flyzy2005.win/fan-qiang/shadowsocks/ss-clients-baidu-cloud-download/ "各版本shadowsocks客户端百度云下载地址")
*   [shadowsocks-manager实现ss多用户管理与流量限制](https://www.flyzy2005.win/fan-qiang/shadowsocks/shadowsocks-manager-config/ "shadowsocks-manager实现ss多用户管理与流量限制")


#### 热门标签

 [搬瓦工](https://www.flyzy2005.win/tag/%e6%90%ac%e7%93%a6%e5%b7%a5/) [Linux](https://www.flyzy2005.win/tag/linux/) [shadowsock配置](https://www.flyzy2005.win/tag/shadowsock%e9%85%8d%e7%bd%ae/) [多用户](https://www.flyzy2005.win/tag/multi-users/) [Java编程思想读书笔记](https://www.flyzy2005.win/tag/java%e7%bc%96%e7%a8%8b%e6%80%9d%e6%83%b3%e8%af%bb%e4%b9%a6%e7%ac%94%e8%ae%b0/) [百度联盟](https://www.flyzy2005.win/tag/baidu-union/) [VPS](https://www.flyzy2005.win/tag/vps/) [iptables](https://www.flyzy2005.win/tag/iptables/) [JVM](https://www.flyzy2005.win/tag/jvm/) [shadowsock](https://www.flyzy2005.win/tag/shadowsock/) [工具](https://www.flyzy2005.win/tag/utils/) [SEO](https://www.flyzy2005.win/tag/seo/) [WordPress](https://www.flyzy2005.win/tag/wordpress/) [vultr](https://www.flyzy2005.win/tag/vultr/) [DigitalOcean](https://www.flyzy2005.win/tag/digitalocean/) [ss](https://www.flyzy2005.win/tag/ss/) [科学上网](https://www.flyzy2005.win/tag/fan-qiang/) [建站](https://www.flyzy2005.win/tag/build-page/) [数据结构](https://www.flyzy2005.win/tag/data-structure/) [CN2 vps推荐](https://www.flyzy2005.win/tag/cn2-vps%e6%8e%a8%e8%8d%90/)

#### VPS推荐

| [![搬瓦工优惠券优惠码](https://www.flyzy2005.win/wp-content/uploads/2018/04/bandwagon.png)](https://www.flyzy2005.win/vps/bandwagon-coupon-buy/)
[**搬瓦工：**年付最低仅需18.79美元](https://www.flyzy2005.win/vps/bandwagon-coupon-buy/) |
| [![vultr优惠码](https://www.flyzy2005.win/wp-content/uploads/2018/06/vultr-logo-300-new.png)](https://www.flyzy2005.win/vps/vultr-deploy/)
[**Vultr：**15个数据中心，最低2.5美元/月](https://www.flyzy2005.win/vps/vultr-deploy/) |
| [![gigsgigscloud优惠码](https://www.flyzy2005.win/wp-content/uploads/2018/07/ggc-logo.png)](https://www.flyzy2005.win/vps/gigsgigscloud-hk-vps/)
[**GigsGigsCloud：**香港VPS，三网直连](https://www.flyzy2005.win/vps/gigsgigscloud-hk-vps/) |

#### 更多

*   [Vultr优惠网](https://www.vultryhw.com "Vultr优惠网 分享Vultr的最新优惠券/优惠码/优惠活动")
*   [搬瓦工优惠网](https://www.bwgyhw.com "搬瓦工优惠网 分享BandwagonHost的最新优惠卷/优惠码/优惠活动")
*   [QQ群组](https://www.flyzy2005.win/go/?url=https://shang.qq.com/wpa/qunwpa?idkey=1e8e1c9e5c8056b05aaaa5e34f6dc9faa7b655610cc790884043b738ceebb753 "flyzy小站QQ交流群")
*   [Telegram群组](https://www.flyzy2005.win/go/go.php?url=https://t.me/flyzyxiaozhan "加入群组，随意聊天")
