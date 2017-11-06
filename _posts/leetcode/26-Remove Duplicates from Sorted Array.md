---
title: 26. Remove Duplicates from Sorted Array
tags:
    - leetcode 
    - 双指针
---
Given a sorted array, remove the duplicates in-place such that each element appear only once and return the new length.

Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.

Example:

Given nums = [1,1,2],

Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively.
It doesn't matter what you leave beyond the new length.

双指针，i为计数器，j为迭代器，当i与j不相等时，计数器+1，同时将j对应元素移动到i的位置
```java
 public int removeDuplicates(int[] a) {
       if (a.length == 0) return 0;
        int i = 0;
        for (int j = 1; j < a.length; j++) {
            if (a[i] != a[j]) {
                i++;
                a[i] = a[j];
            }

        }
        return i + 1;
    }
```

