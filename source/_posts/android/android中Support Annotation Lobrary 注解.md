---
title: android中Support Annotation Lobrary 注解
date: 2018-07-10 14:04:48
categories: android
tags: android,annotations
---
# android中Support Annotation Lobrary 注解

官方文档写的太好了，我觉得不可能有他写的更好的文章了，所以就不打算了写了，官方连接<https://developer.android.com/studio/write/annotations?hl=zh-cn>。

# 速查表
|注解|类型|
|--|--|
|``Nullable``|标记的函数可以返回``null``,或者标记的参数可以为``null``|
|``NonNull``|标记的函数可以不能为空,或者标记的参数不能为空|
|``AnimatorRes``|标记整型值是``android.R.animator``类型|
|``AnimRes``|标记整型值是``android.R.anim``类型|
|``AnyRes``|标记整型值是任意资源类型|
|``ArrayRes``|标记整型值是``android.R.array``类型|
|``AttrRes``|标记整型值是``android.R.attr``类型|
|``BoolRes``|标记整型值是布尔值类型|
|``ColorRes``|标记整型值是``android.R.color``类型|
|``DrawableRes``|标记整型值是``android.R.drawable``类型|
|``FractionRes``|标记整型值是``fraction``类型|
|``IdRes``|标记整型值是``android.R.id``类型|
|``IntegerRes``|标记整型值是``android.R.integer``类型|
|``InterpolatorRes``|标记整型值是``android.R.integer``类型|
|``LayoutRes``|标记整型值是``android.R.layout``类型|
|``MenuRes``|标记整型值是``android.R.menu``类型|
|``PluralsRes``|标记整型值是``android.R.plurals``类型|
|``RawRes``|标记整型值是``android.R.raw``类型|
|``StringRes``|标记整型值是``android.R.string``类型|
|``StyleableRes``|标记整型值是``android.R.styleable``类型|
|``StyleRes``|标记整型值是``android.R.style``类型|
|``TransitionRes``|标记整型值是``android.R.transition``类型|
|``XmlRes``|标记整型值是``android.R.xml``类型|
|``ColorRes``|标记参数类型需要传入的是颜色类型的资源ID|
|``Size``|标记参数的大小|
|``Size(min = 1)``|标记集合不可以为空|
|``Size(max = 1024)``|标记字符串最大的字符个数是1024|
|``Size(2)``|表示数组元素个数是2个|
|``Size(multiple = 2)``|标记数组的大小是2的倍数|
|``IntRange(from =0,to = 255)``|标记参数是从0-255之间的整型|
|``FloatRange(from =0.0,to = 255.0)``|标记参数是从0-255之间的浮点型|
|``CallSuper``|标记该方法在重写的同时一定要调用被这个方法|


# 参考链接
* <https://developer.android.com/studio/write/annotations?hl=zh-cn>
* <https://developer.android.com/guide/topics/resources/string-resource>