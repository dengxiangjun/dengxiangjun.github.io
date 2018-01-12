---
title: 6. ZigZag Conversion
tags:
    - leetcode 
    - 边界题
---
https://leetcode.com/problems/zigzag-conversion/description/
The string "PAYPALISHIRING" is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

P   A   H   N
A P L S I I G
Y   I   R
And then read line by line: "PAHNAPLSIIGYIR"
Write the code that will take a string and make this conversion given a number of rows:

string convert(string text, int nRows);
convert("PAYPALISHIRING", 3) should return "PAHNAPLSIIGYIR".
## 答案
每行是一个StringBuilder，先垂直向下转换，再斜向上转换。
```java
     public static String convert(String s, int numRows) {
        int len = s.length();
        char[] c = s.toCharArray();
        StringBuilder[] sb = new StringBuilder[len];
        for (int i = 0; i < numRows && i < len; i++) sb[i] = new StringBuilder();
        int i = 0;
        while (i < len) {
            for (int j = 0; j < numRows && i < len; j++) sb[j].append(c[i++]);//垂直向下
            for (int j = numRows - 2; j > 0 && i < len; j--) sb[j].append(c[i++]);
            //斜向上
        }
        StringBuilder res = new StringBuilder();
        for (int k = 0; k < numRows && k < len; k++) res.append(sb[k]);
        return res.toString();
    }
```

