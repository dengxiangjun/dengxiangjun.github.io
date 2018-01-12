---
title: 9. Palindrome Number
tags:
    - leetcode 
    - 边界题
---
Determine whether an integer is a palindrome. Do this without extra space.

click to show spoilers.

Some hints:
Could negative integers be palindromes? (ie, -1)

If you are thinking of converting the integer to string, note the restriction of using extra space.

You could also try reversing an integer. However, if you have solved the problem "Reverse Integer", you know that the reversed integer might overflow. How would you handle such case?

There is a more generic way of solving this problem.
## 答案
将数字的末尾一半反转
```java
      public static boolean isPalindrome(int x) {
        if (x < 0) return false;
        if (x >= 0 && x < 10) return true;
        if (x % 10 == 0) return false;
        int result = 0;
        while (x > result) {
            int tail = x % 10;
            result = result * 10 + tail;
            x /= 10;
        }
        return x == result || result / 10 == x;
    }
```

