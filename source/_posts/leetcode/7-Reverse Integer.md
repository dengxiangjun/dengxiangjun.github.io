---
title: 7. Reverse Integer
tags:
    - leetcode 
    - 简单题
---
Given a 32-bit signed integer, reverse digits of an integer.

Example 1:

Input: 123
Output:  321
Example 2:

Input: -123
Output: -321
Example 3:

Input: 120
Output: 21
## 答案
使用tail存储数字的最后一位数字的值，每处理掉一个末尾数字，结果扩大十倍，当扩大了十倍后再求一次逆，如果能得到原来的值，说明数字没有溢出。
```java
      public int reverse(int x) {
        int result = 0;
        while (x != 0){
            int tail = x % 10;
            int newResult = result * 10 + tail;
            if ((newResult - tail) / 10 == result) return 0;
            result = newResult;
            x /= 10;
        }
        return result;
    }
```

