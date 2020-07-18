---
title: kotlin 中的拓展函数
date: 2020-07-08 20:43:34
tags: kotlin
categories: [kotlin,let,run]
---


# Kotlin 中的拓展函数

## run
作用：执行传入的函数，并返回执行结果。
调用示例：
```
    val test: String ?= "" 
    test?.run { 
        print("run")
    }
```
``run`` 函数最终返回的是 ``block`` 执行的结果，`` run`` 函数内部没有指向调用者的 it.

实现源码：
```
@kotlin.internal.InlineOnly
public inline fun <R> run(block: () -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

## with
``with``语法糖不再是⼀个拓展函数了，⽽是需要在语法糖的第⼀个参数⾥⾯传⼊接收者对象的实例，第⼆个参数就是带接收者的函数
字⾯值实例，返回的也是 ``block`` 调⽤的结果，这⼀点和 run 语法糖类似。

调用示例:
```
val test: String ?= "" 
    with(test) {
       print("with")
    }
```
实现源码：
```
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block()
}
```

## apply
apply 函数内部会自带调用者，返回的是调用者本身。
调用示例：
```
val test: String ?= "" 
    test?.apply { 
       substring(1)
    }
```

实现源码：
```
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

## alos
``alos`` 和 ``let`` 类似，区别在于 ``alos`` 的返回值是 caller 本身，而 ``let`` 返回的是 ``block`` 的执行结果。
调用示例:
```   
val test: String ?= "" 
test?.also {
        it.length
    }
```

实现源码：
```
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this)
    return this
}

```

## let
`` let `` 方法块中，可以通过 it 来调用 caller, 返回值是方法块的执行结果。

调用示例:
```   
val test: String ?= "" 
test?.let {
        it.length
    }
```

源码：
```
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}

```

## takeIf
``takeIf`` 要求方法块执行结果为 ``Boolean``,如果 ``block``执行的结果为 ``ture``,则 ``takeIf`` 返回值为 ``caller``,否则为 ``null。

调用示例：
```
val test: String ?= "ii" 
    test?.takeIf { 
        !it.isBlank()
    }?.let { 
        print(it)
    }
```

源码：
```
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? {
    contract {
        callsInPlace(predicate, InvocationKind.EXACTLY_ONCE)
    }
    return if (predicate(this)) this else null
}

```
## takeUnless
 ``takeUnless`` 和 ``takeIf`` 相反，当 ``block`` 执行结果为 ``false``时，返回 ``caller``，否则返回 ``null``

 调用示例：
```
val test: String ?= "ii" 
    test?.takeUnless { 
        it.isBlank()
    }?.let { 
        print(it)
    }
```

源码：
```
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T? {
    contract {
        callsInPlace(predicate, InvocationKind.EXACTLY_ONCE)
    }
    return if (!predicate(this)) this else null
}

```
## repeat
需要重复的 action 模版，传入需要执行的次数，和需要执行的 ``block`` 即可。
调用示例：
```
repeat(10) { print(it)}
```

实现源码：
```
@kotlin.internal.InlineOnly
public inline fun repeat(times: Int, action: (Int) -> Unit) {
    contract { callsInPlace(action) }

    for (index in 0 until times) {
        action(index)
    }
}
```

## referrers:
* <https://aisia.moe/2018/03/25/kotlin-contracts-dsl/>
* <https://segmentfault.com/a/1190000015434807#item-2-5>