---
title: 5. Longest Palindromic Substring
tags:
    - leetcode 
    - 双指针
---
Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.

Example:

Input: "babad"

Output: "bab"

Note: "aba" is also a valid answer.
Example:

Input: "cbbd"

Output: "bb"

## 双指针
由中心点向外扩散,一种是奇数个字符的回文子串，一种是偶数个字符的回文子串。![](/img/LeetCode/5-1.jpg)
```java
       public static String longestPalindrome(String s) {
        int length = s.length();
        if (length == 1) return s;
        int max = 0;
        String res = null;
        for (int i = 0; i < length; i++) {
            int low = i, high = i;
            while (low >= 0 && high < s.length()) {
                if (s.charAt(low) == s.charAt(high) && (high - low + 1) > max) {
                    max = high - low + 1;
                    res = s.substring(low, high + 1);
                } else break;
                low--;
                high++;
            }
            low = i;
            high = i + 1;
            while (low >= 0 && high < s.length()) {
                if (s.charAt(low) == s.charAt(high) && (high - low + 1) > max) {
                    max = high - low + 1;
                    res = s.substring(low, high + 1);
                } else break;
                low--;
                high++;
            }
        }
        return res;
    }
```

## Manacher算法
