---
title: Nim游戏
date: 2018-11-13 23:44:55
categories: algorithm
tags: lettcode
---

### [Nim游戏](https://leetcode-cn.com/problems/nim-game/)

难度 **简单**



你和你的朋友，两个人一起玩 ：桌子上有一堆石头，每次你们轮流拿掉 1 - 3 块石头。 拿掉最后一块石头的人就是获胜者。你作为先手。

你们是聪明人，每一步都是最优解。 编写一个函数，来判断你是否可以在给定石头数量的情况下赢得游戏。

**示例:**

```
**输入:** `4`
**输出:** false 
**解释:** 如果堆中有 4 块石头，那么你永远不会赢得比赛；
     因为无论你拿走 1 块、2 块 还是 3 块石头，最后一块石头总是会被你的朋友拿走。
```

### 结题思路
简单的罗列一下1-12个石子的情况，发现只有4的倍数才是false。


#### Solution

Language: **Java**

```java
class Solution {
    public boolean canWinNim(int n) {
          return n%4!=0;
    }
}
```
