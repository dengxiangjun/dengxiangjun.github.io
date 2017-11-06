---
title: 3. Longest Substring Without Repeating Characters
tags:
    - leetcode 
    - 动态规划
    - 双指针
---
Given a string, find the length of the longest substring without repeating characters.

Examples:

Given "abcabcbb", the answer is "abc", which the length is 3.

Given "bbbbb", the answer is "b", with the length of 1.

Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

最长不重复子序列
用一个hash表存放当前字符在串中的位置，两个指针i和j,j控制当前遍历到的字符，i为重复序列的尾指针 i = max(hash[c[j]],i),使用一个maxLen变量，存储遍历过程中的最大长度。
```java
 public int lengthOfLongestSubstring(String s) {
        char[] c = s.toCharArray();
        if(c.length == 0) return 0;
        int[] hash = new int[256];
        int maxLen = 1;
        for (int i = 0,j = 0;j<c.length;j++){
            i = Math.max(hash[c[j]],i);
            hash[c[j]] = j + 1;
            maxLen =  Math.max(j - i + 1,maxLen);
        }
        return maxLen;
    }
```

