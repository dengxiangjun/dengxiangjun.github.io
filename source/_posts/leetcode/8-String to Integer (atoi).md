---
title: 8. String to Integer (atoi)
tags:
    - leetcode 
    - 边界题
---
Implement atoi to convert a string to an integer.

Hint: Carefully consider all possible input cases. If you want a challenge, please do not see below and ask yourself what are the possible input cases.

Notes: It is intended for this problem to be specified vaguely (ie, no given input specs). You are responsible to gather all the input requirements up front.

Update (2015-02-10):
The signature of the C++ function had been updated. If you still see your function signature accepts a const char * argument, please click the reload button  to reset your code definition.
## 答案
在判断数字是否越界的时候，若假设int数字范围是[-128,127],对于数字130，当读到13时，13 > 127/10，因此第一个判定条件是result > Integer.MAX_VALUE / 10；对于数字128，当读到12时，8 > 127 % 10,因此第二个判定条件是tail > Integer.MAX_VALUE % 10。
```java
      public static int myAtoi(String str) {
        if ("".equals(str)) return 0;
        char[] c = str.toCharArray();
        int len = c.length;
        int i = 0;
        int sign = 1;
        while (c[i] == ' ') i++;
        if (c[i] == '-') {
            sign = -1;
            i++;
        } else if (c[i] == '+') i++;

        int result = 0;
        while (i < len) {
            if (c[i] < '0' || c[i] > '9') break;
            int tail = c[i] - '0';
            if (sign == 1) {
                if (result > Integer.MAX_VALUE / 10
                        || result == Integer.MAX_VALUE / 10 && tail > Integer.MAX_VALUE % 10)
                    return sign * Integer.MAX_VALUE;
            } else {
                if (result > Integer.MAX_VALUE / 10 || result == Integer.MAX_VALUE / 10 && tail > Integer.MIN_VALUE % 10 + 1)
                    return Integer.MIN_VALUE;
            }

            result = result * 10 + tail;
            i++;
        }
        return sign * result;
    }
```

