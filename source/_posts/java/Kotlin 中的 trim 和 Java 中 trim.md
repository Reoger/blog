---
layout: Java 中的 trim() 和 kotlin 中的 trim()
title: Java 中的 trim() 和 kotlin 中的 trim()
date: 2020-06-06 17:34:05
categories: java 
tags: android,trim(),unicode
---

# java 中 trim 方法的实现

源代码如下：
```
public String trim() {
    int len = value.length;
    int st = 0;
    char[] val = value;    /* avoid getfield opcode */

    while ((st < len) && (val[st] <= ' ')) {
        st++;
    }
    while ((st < len) && (val[len - 1] <= ' ')) {
        len--;
    }
    return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
}
```
可以看到，在 java 实现中， ``trim``方法截取掉了前后 Unicode 小于等于空格的所有控制字符。具体截取的字符可以参考 [unicode 字符列表](https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%88%97%E8%A1%A8).

# kotlin 中 trim() 方法的实现

```String.java

@kotlin.internal.InlineOnly
public inline fun String.trim(): String = (this as CharSequence).trim().toString()

public fun CharSequence.trim(): CharSequence = trim(Char::isWhitespace)

public inline fun CharSequence.trim(predicate: (Char) -> Boolean): CharSequence {
    var startIndex = 0
    var endIndex = length - 1
    var startFound = false

    while (startIndex <= endIndex) {
        val index = if (!startFound) startIndex else endIndex
        val match = predicate(this[index])

        if (!startFound) {
            if (!match)
                startFound = true
            else
                startIndex += 1
        } else {
            if (!match)
                break
            else
                endIndex -= 1
        }
    }

    return subSequence(startIndex, endIndex + 1)
}
```
可以看到，kotlin 中的 trim 方法和 java 中的 trim 方法实现基本差不多，唯一的差别就是匹配的字符规则不一样，在 java 匹配的规则是 `` <= ' ' ``, 而 kotlin 中匹配的规则是 `` Char::isWhitespace``。

接下来我们看看 ``Char::isWhitespace`` 的具体实现：[参考](<https://github.com/JetBrains/kotlin/blob/deb416484c5128a6f4bc76c39a3d9878b38cec8c/libraries/stdlib/jvm/src/kotlin/text/CharJVM.kt#L72>)
```
String.kt
expect fun Char.isWhitespace(): Boolean

实现在 charJVM.kt
public actual fun Char.isWhitespace(): Boolean = Character.isWhitespace(this) || Character.isSpaceChar(this)
```

可以看到, 在 String.kt 中只是通过 ``expect `` 声明了这个方法，具体实现还是在 charJVM.kt 中。关于 ``expect`` 和 ``actual`` 的用法可以参考 [kotlin 平台相关声明](https://www.kotlincn.net/docs/reference/platform-specific-declarations.html).

而在``Charactr`` 类中，``isWhitespace()`` 和 ``isSpaceChar()``的实现如下：
```
    public static boolean isWhitespace(char ch) {
        return isWhitespace((int)ch);
    }

    public static boolean isWhitespace(int codePoint) {
        return CharacterData.of(codePoint).isWhitespace(codePoint);
    }


    public static boolean isSpaceChar(char ch) {
        return isSpaceChar((int)ch);
    }

    public static boolean isSpaceChar(int codePoint) {
        return ((((1 << Character.SPACE_SEPARATOR) |
                  (1 << Character.LINE_SEPARATOR) |
                  (1 << Character.PARAGRAPH_SEPARATOR)) >> getType(codePoint)) & 1)
            != 0;
    }
```
这里 ``isWhitespace()`` 和 ``isSpaceChar()`` 的实现其实比较复杂，简单介绍一下其区别。

``isWhiteSpace()``符合下面条件的字符将会返回 ``true``


<li> It is a Unicode space character ({@code SPACE_SEPARATOR},
    {@code LINE_SEPARATOR}, or {@code PARAGRAPH_SEPARATOR})
    but is not also a non-breaking space ({@code '\u005Cu00A0'},
    {@code '\u005Cu2007'}, {@code '\u005Cu202F'}).
<li> It is {@code '\u005Ct'}, U+0009 HORIZONTAL TABULATION.
<li> It is {@code '\u005Cn'}, U+000A LINE FEED.
<li> It is {@code '\u005Cu000B'}, U+000B VERTICAL TABULATION.
<li> It is {@code '\u005Cf'}, U+000C FORM FEED.
<li> It is {@code '\u005Cr'}, U+000D CARRIAGE RETURN.
<li> It is {@code '\u005Cu001C'}, U+001C FILE SEPARATOR.
<li> It is {@code '\u005Cu001D'}, U+001D GROUP SEPARATOR.
<li> It is {@code '\u005Cu001E'}, U+001E RECORD SEPARATOR.
<li> It is {@code '\u005Cu001F'}, U+001F UNIT SEPARATOR.


``isSpaceChar()``用于检查``Unicodei``分割符和空格，参考[Unicode控制字符](https://zh.wikipedia.org/wiki/Unicode%E6%8E%A7%E5%88%B6%E5%AD%97%E7%AC%A6),分割符号包括
<li> U+2028 line separator ，HTML：&#8232;，LSEP
<li> U+2029 paragraph separator ，HTML：&#8233;，PSEP



# 总结
在 java 中和 kotlin 中，因为 ``trim`` 方法中使用到的匹配规则不一样，所以他们对同一个串进行处理之后，结果可能也是不一样的。所以为了保险起见，建议不要使用分别用 java 和 kotlin 语言 trim 之后的字符串在进行比较，可能会得有意料之外的结果。





# 参考链接
* [Unicode字符列表](https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%88%97%E8%A1%A8)
* [Unicode控制字符](https://zh.wikipedia.org/wiki/Unicode%E6%8E%A7%E5%88%B6%E5%AD%97%E7%AC%A6)
* [kotlin 平台相关声明](https://www.kotlincn.net/docs/reference/platform-specific-declarations.html)
* [Unicode 字符百科](https://unicode-table.com/cn/blocks/control-character/)