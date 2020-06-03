---
title: 两个数组的交集 II(
date: 2018-07-11 10:55:48
categories: algorithm
tags: leetcode 
---

### [350\. 两个数组的交集 II(Intersection of Two Arrays II)](https://leetcode-cn.com/problems/intersection-of-two-arrays-ii/description/)

题目难度： **简单**



给定两个数组，写一个方法来计算它们的交集。

**例如:**  
给定_nums1_ = `[1, 2, 2, 1]`, _nums2_ = `[2, 2]`, 返回 `[2, 2]`.

**注意：**

*   输出结果中每个元素出现的次数，应与元素在两个数组中出现的次数一致。
*   我们可以不考虑输出结果的顺序。

****跟进:****

*   如果给定的数组已经排好序呢？你将如何优化你的算法？
*   如果 _nums1 _的大小比 _nums2 _小很多，哪种方法更优？
*   如果_nums2_的元素存储在磁盘上，内存是有限的，你不能一次加载所有的元素到内存中，你该怎么办？



#### Solution
**解题思路:**
先将两个数组排好序，我们需要维护两个指针，指针i用于指向nums1，指针j指向nums2。当``nums1[i] > nums2[j]``时，j++,当``nums1[i] < nums2[j]``，i++。我们需要寻找的情况就是有多少对nums1[i] == nums2[j]的情况，记录下来即可，记得把指针i和j往前移一位。

Language: **Java**

```java
class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
      if(nums1.length ==0 || nums2.length == 0)
            return new int[0];
        Arrays.sort(nums1);
        Arrays.sort(nums2);
        int i=0,j=0;
        List res = new ArrayList<Integer>();
        while(i<nums1.length && j<nums2.length){
            if(nums1[i]==nums2[j]){
                res.add(nums1[i]);
                i++;
                j++;
            }else if(nums1[i]>nums2[j]){
                j++;
            }else if(nums1[i]<nums2[j]){
                i++;
            }
        }
        int a[] = new int[res.size()];
        for(int k=0;k<res.size();k++){
            a[k] = (int) res.get(k);
        }
        return a;
    }
}
```
