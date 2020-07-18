---
title: LayoutParams 使用
date: 2018-07-09 10:55:48
categories: android
tags: android,LayoutParams
---
#  ViewGroup.LayoutParams 的使用记录
LayoutParams类 一般用来child view（子视图） 向 parent view（父视图）传达自己的意图（孩子想变成什么样向其父亲说明），即子布局想在父布局中显示的属性（例如子布局多大、在那个位置）通过LayoutParams来进行展示。

# 常见的属性
## 宽(width)和高(hegit)
宽和高一般来说有三种值：
1. 确定的数字
2. MATH_PARENT，和父布局一样大
3. WRAP_CONTENT，自己本身多大就显示多大

## margin
margin属性，上下左右边距。

## 
更多的属性设置，可以参考<https://developer.android.com/reference/android/R.styleable#ViewGroup_Layout>.


## 具体使用
例如向一个``RelativeLayout``的父布局中添加一个View，示例代码如下：
```
        RelativeLayout.LayoutParams layoutParams = (RelativeLayout.LayoutParams) view.getLayoutParams();
        layoutParams.topMargin = y;
        layoutParams.leftMargin = x;
        view.setLayoutParams(layoutParams);
```
**注意**，父布局xml中是是``RelativeLayout``,子布局在``setLayoutParams``时，也必须是``RelativeLayout.LayoutParams``，否则会报错。


## 具体实践 
添加用户引导，高亮某个按钮时的实现思路。一种总体思路，创建一个透明主题的activity，在activity中按钮的位置，添加一个相同的高亮按钮到这个透明主题的activity中，然后启动这个activity，将其覆盖展现在需要用户引导的界面，如此就实现了用户引导。
重点分析： 在父布局中添加子布局，并指定了位置。
关键代码：
```
//获取指定 view的位置，这个sTargetButton就是目标view
  int[] location = new int[2];
    sTargetButton.getLocationOnScreen(location);

//将view添加到父布局中,调整到制定位置。
 ViewGroup.MarginLayoutParams margin = new ViewGroup.MarginLayoutParams(view.getLayoutParams());
        RelativeLayout.LayoutParams layoutParams = (RelativeLayout.LayoutParams) view.getLayoutParams();
        layoutParams.topMargin = y;
        layoutParams.leftMargin = x;
        view.setLayoutParams(layoutParams);
```

# 参考
* <https://developer.android.com/reference/android/view/ViewGroup.LayoutParam>
* <https://blog.csdn.net/superherowupan/article/details/51345975>