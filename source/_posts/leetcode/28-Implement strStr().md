---
title: 28. Implement strStr()
tags:
    - leetcode 
    - 双指针
    - 字符串匹配
---
Implement strStr().

Return the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.

Example 1:

Input: haystack = "hello", needle = "ll"
Output: 2
Example 2:

Input: haystack = "aaaaa", needle = "bba"
Output: -1
## 蛮力法
双指针，i遍历主串，j遍历模式串，当haystack[i] != needle[j]时，回溯，i回溯到当前主串中的第一个元素的的下一个位置，j = 0;
```java
    public int strStr(String haystack, String needle) {
         int len1 = haystack.length(), len2 = needle.length();
        int i = 0, j = 0;
        while (i < len1 && j < len2) {
            if (haystack.charAt(i) == needle.charAt(j)) {
                i++;
                j++;
            } else {
                i = i - j + 1;
                j = 0;
            }
        }
        if (j == len2) return i-j;
        else return -1;
    }
```
