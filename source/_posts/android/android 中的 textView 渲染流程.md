---
title: android中的类加载
date: 2020-07-11 10:23:33
categories: android
tags: android, TextView
---

## 带着问题看源码

1. TextView 的整体渲染流程 
    主要有 staticLayout（分块绘制）、（动作监听）、（文本渲染处理，展示成 ** 之类的）、（）
2. TextView 怎么做到自动换行，怎么修改换行策略
    跑马灯的实现
3. TextView 在展示不下时，怎么处理的？··· 是怎么添加到末尾的？
    待续
4. TextView 怎么做到在文字末尾添加一个图标（多语言适配）
    待续

知识点：为什么不能在主线程更新 UI

设计考虑：如果在更新 UI 时需要考虑多线程安全问题的话，会导致性能降低，为了避免这种性能降低，所以规定只能在 UI 线程中更新界面，如果不在主线程中做更新界面的实现的话，就会主动抛出异常。


onMeasure

onLayout

onDraw

三种 mode
unspeceieied 没有指定大小，子布局要求多大就多大
exatly 指定进行精确的大小，父布局说多大就多大
at_most 最大多大，父布局设置的上线，子 view 最大的高度或者宽度

MeasureSpec.UNSPECIFIED（0）
、MeasureSpec.EXACTLY（1）
、和MeasureSpec.AT_MOST（2）
