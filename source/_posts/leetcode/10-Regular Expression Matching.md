---
title: 10. Regular Expression Matching
tags:
    - leetcode 
    - 回溯
    - 动态规划
---
https://leetcode.com/problems/regular-expression-matching/description/
Implement regular expression matching with support for '.' and '*'.

'.' Matches any single character.
'*' Matches zero or more of the preceding element.

The matching should cover the entire input string (not partial).

The function prototype should be:
bool isMatch(const char *s, const char *p)

Some examples:
isMatch("aa","a") → false
isMatch("aa","aa") → true
isMatch("aaa","aa") → false
isMatch("aa", "a\*") → true
isMatch("aa", ".\*") → true
isMatch("ab", ".\*") → true
isMatch("aab", "c\*a\*b") → true
## 回溯
每次考察模式串中两个字符，讨论模式串第二个字符不是'\*'的时候，如果字符串中第一个字符与模式串中的第一个字符匹配的时候，字符串和模式串都向后移动一个字符；如果如果字符串中第一个字符与模式串中的第一个字符不匹配，那么直接返回false。
当模式串第二个字符是'\*'的时候，如果字符串和模式串的第一个字符相同，或者模式串中的第一个字符是'.',那么可以我们可以让模式串匹配0个字符，一个字符或者多个字符。如果字符串和模式串的第一个字符不相同。那就是匹配0个字符。
对于递归终止条件，
```java
     public static boolean isMatch(String s, String p) {
        if (s == null || p == null) return false;
        else return isMatch(s, p, 0, 0);
    }

    public static boolean isMatch(String s, String p, int i, int j) {
        if (i == s.length() && j == p.length()) return true;
        if (i != s.length() && j == p.length()) return false;
        if (i == s.length() && j != p.length()) {
            //只有pattern剩下的部分类似a*b*c*的形式，才匹配成功
            if (j + 1 < p.length() && p.charAt(j + 1) == '*') {
                return isMatch(s, p, i, j + 2);//匹配0个字符
            }
            return false;
        }
        if (j + 1 < p.length() && p.charAt(j + 1) == '*') {
            if (s.charAt(i) == p.charAt(j) || p.charAt(j) == '.')
                return isMatch(s, p, i, j + 2)//匹配0个字符
                 || isMatch(s, p, i + 1, j + 2) //匹配一个字符
                 || isMatch(s, p, i + 1, j);//匹配多个字符
            else return isMatch(s, p, i, j + 2);//匹配0个字符
        }
        if (p.charAt(j) == '.' || s.charAt(i) == p.charAt(j))
            return isMatch(s, p, i + 1, j + 1);
        return false;
    }
```

