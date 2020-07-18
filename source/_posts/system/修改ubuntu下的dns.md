---
title: 修改ubuntu下的dns
date: 2018-10-13 19:08:55
categories: system 
tags: [linux]
---

# 修改ubuntu下的dns
适用环境：oracle vm 虚拟机下的ununtu-14.04.5版本。
## 验证有效的方法：<https://blog.csdn.net/u014453443/article/details/80878061>

基本步骤：
1. 查看当前dns配置：
```
cat /etc/resolv.conf
```
2. 添加dns配置,在interfaces文件下添加需要的dns配置，
```
sudo vi /etc/network/interfaces
```
参考的dns配置地址：
```
dns-nameservers 8.8.8.8 
dns-nameservers 8.8.4.4 
```
参考配置如下：
![image.png](https://upload-images.jianshu.io/upload_images/2178834-3b11eca15608f683.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 重启，后验证dns
```
cat /etc/resolv.conf
```

![image.png](https://upload-images.jianshu.io/upload_images/2178834-41be0ea18ebb5d5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

dns修改成功！ 


## 可能有效的方法 <https://blog.csdn.net/qq_27818541/article/details/75730125>

## 验证无效，但是其他版本可能有效的方法 <https://blog.csdn.net/zd147896325/article/details/81078414>