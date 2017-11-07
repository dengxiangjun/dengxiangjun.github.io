---
title: 27. Remove Element
tags:
    - leetcode 
    - 双指针
---
Given an array and a value, remove all instances of that value in-place and return the new length.

Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.

The order of elements can be changed. It doesn't matter what you leave beyond the new length.

Example:

Given nums = [3,2,2,3], val = 3,

Your function should return length = 2, with the first two elements of nums being 2.
## 方法1
快慢双指针，i为计数器，j为迭代器，当a[j]与val不相等时，计数器+1，同时将j对应元素复制到i的位置
```java
 public static int removeElement(int[] a, int val) {
        if (a.length == 0) return 0;
        int i = 0;
        for (int j = 0; j < a.length; j++) {
            if (a[j] != val) {
                a[i] = a[j];
                i++;
            }

        }
        return i;
    }
```

## 方法2
左右双指针，当左指针a[i] == val时，将右指针的值拷贝到作指针交换
```java
 public static int removeElement(int[] a, int val) {
        if (a.length == 0) return 0;
        int i = 0;
        int n = a.length;
        while (i< n) {
            if (a[i] == val) {
                a[i] = a[n-1];
                n--;
            }else i++;

        }
        return i;
    }
```
