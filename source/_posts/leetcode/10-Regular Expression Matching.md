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

## 动态规划
利用一个二维数组存储匹配状态。数组维度为m + 1和n + 1。
### 初始化
1. 第0行表示s是空字符串，第0列表示p是空字符串。当s和p都是空串时，匹配成功，因此dp[0][0] = 1.
2. 当s为空串，p为"a\*b\*"类的字符串时，是可以匹配成功的，但是当p为"a\*bc\*"这种类型的字符串匹配失败，所以我们令dp[0][i] = dp[0][i - 2].表示匹配0个\*号。
### 状态转移方程
1. sc[i - 1] == pc[j - 1] || pc[j - 1] == '.'时，状态与前一个字符串的匹配情况一样。
2. 当遇到\*号时，分两种情况讨论：
    1. p中\*号前面的字符与s中字符匹配或者为'.'字符时，那么可以匹配0次、1次、多次该字符。举例来说:s = "aaac", p = "ab\*a\*c\*d\*".匹配0次，如b\*,说明,模式串中的b\*要忽略，那么当前的状态与dp[i][j - 2]相同；匹配一次，如c\*，那么当前的状态与dp[i][j - 1]相同，表示用第j - 1个字符与当前字符去匹配；匹配多个如a\*，当前的状态与dp[i - 1][j]相同，表示前面的字符也用到了模式串a\*.
    2. 否则，匹配0个如d\*，dp[i][j] = dp[i][j - 2].
3. 否则，匹配失败。
```java
 public static boolean isMatch(String s, String p) {
        int m = s.length(), n = p.length();
        char[] sc = s.toCharArray(), pc = p.toCharArray();
        boolean[][] dp = new boolean[m + 1][n + 1];
        dp[0][0] = true;
        for (int i = 2; i <= n; i++) {
            if (pc[i - 1] == '*') {
                dp[0][i] = dp[0][i - 2]; // *可以消掉c*，s="a",
            }
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (sc[i - 1] == pc[j - 1] || pc[j - 1] == '.') {
                    dp[i][j] = dp[i - 1][j - 1];
                } else if (pc[j - 1] == '*') {
                    if (sc[i - 1] == pc[j - 2] || pc[j - 2] == '.') {// 当*的前一位是'.'， 或者前一位的pc等于sc的话，
                        dp[i][j] = dp[i][j - 2] || dp[i][j - 1] || dp[i - 1][j];// *代表1个(dp[i][j - 1])，*代表多个(dp[i - 1][j])，或者用*消掉c*(dp[i][j - 2])
                    } else
                        dp[i][j] = dp[i][j - 2]; // 用*消掉c*
                } else
                    dp[i][j] = false;

            }
        }
        return dp[m][n];
    }
```

## 类似题目
Implement wildcard pattern matching with support for '?' and '*'.

'?' Matches any single character.
'*' Matches any sequence of characters (including the empty sequence).

The matching should cover the entire input string (not partial).

The function prototype should be:
bool isMatch(const char *s, const char *p)

Some examples:
isMatch("aa","a") → false
isMatch("aa","aa") → true
isMatch("aaa","aa") → false
isMatch("aa", "*") → true
isMatch("aa", "a*") → true
isMatch("ab", "?*") → true
isMatch("aab", "c*a*b") → false

```java
public static boolean isMatch(String s, String p) {
        int m = s.length(), n = p.length();
        char[] sc = s.toCharArray(), pc = p.toCharArray();
        boolean[][] dp = new boolean[m + 1][n + 1];
        dp[0][0] = true;
        for (int i = 1; i <= n; i++) {
            if (pc[i - 1] == '*') dp[0][i] = dp[0][i - 1];
        }

        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++) {
                if (sc[i - 1] == pc[j - 1] || pc[j - 1] == '?') dp[i][j] = dp[i - 1][j - 1];
                else if (pc[j - 1] == '*') dp[i][j] = dp[i][j - 1] || dp[i - 1][j - 1] || dp[i - 1][j];
                else dp[i][j] = false;
            }
        return dp[m][n];
    }
```