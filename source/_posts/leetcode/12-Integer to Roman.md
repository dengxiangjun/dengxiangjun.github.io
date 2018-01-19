---
title: 12. Integer to Roman
tags:
    - leetcode 
    - 贪心算法
---
https://leetcode.com/problems/integer-to-roman/description/
Given an integer, convert it to a roman numeral.

Input is guaranteed to be within the range from 1 to 3999.
## 贪心算法
转换时，每次选取最大的一个值
```java
        public static String intToRoman(int num) {
        Map<Integer, String> map = new HashMap<Integer, String>();
        int[] a = {1000,900,500,400,100,90,50,40,10,9,5,4,1};
        String[] s = {"M","CM","D","CD","C","XC","L","XL","X","IX","V","IV","I"};
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i< a.length;i++){
            while (num > a[i]){
                num -= a[i];
                sb.append(s[i]);
            }
        }
        return sb.toString();
    }
```
## 方法2
好像没什么好说的。。。
```java
 public static String intToRoman(int num) {
        String[] M = {"", "M", "MM", "MMM"};
        String[] C = {"", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM"};
        String[] X = {"", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC"};
        String[] I = {"", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"};
        StringBuilder sb = new StringBuilder();
        sb.append(M[num / 1000]).append(C[num % 1000 / 100]).append(X[num % 100 / 10]).append(I[num % 10]);
        return sb.toString();
    }
```