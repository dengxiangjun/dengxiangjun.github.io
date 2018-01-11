---
title: 11. Container With Most Water
tags:
    - leetcode 
    - 双指针
---
Given n non-negative integers a1, a2, ..., an, where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

Note: You may not slant the container and n is at least 2.


## 双指针
使用左右两个指针，当左边指针对应高度小于右边指针对应高度时，左边指针跳到下一个位置；反之，当右边指针对应高度小于左边指针对应高度时，右边指针跳到下一个位置。这样跳过较低高度的方法使得更低的容积不在考虑范围之内。
```java
      public int maxArea(int[] height) {
        int i = 0, j = height.length - 1;
        int max = Integer.MIN_VALUE;
        while (i < j) {
            int area = Math.min(height[i],height[j]) * (j - i);
            max = Math.max(max, area);
            if (height[i] < height[j]) i++;
            else j--;
        }
        return max;
    }
```

