---
title: 300. Longest Increasing Subsequence
tags:
    - leetcode 
    - 动态规划
---
Given an unsorted array of integers, find the length of longest increasing subsequence.

For example,
Given [10, 9, 2, 5, 3, 7, 101, 18],
The longest increasing subsequence is [2, 3, 7, 101], therefore the length is 4. Note that there may be more than one LIS combination, it is only necessary for you to return the length.

Your algorithm should run in O(n2) complexity.

Follow up: Could you improve it to O(n log n) time complexity?
## 递归， O（2^n）
有一个上升序列，针对每一个数字。要么它加入到上升子序列中（如果要加入明它大于之前的上升序列的最后一个值），要么不加入到上升子序列中。我们取这两种情况的最大值。
```java
    public static int lengthOfLIS(int[] a) {
        return lengthofLIS(a, Integer.MIN_VALUE, 0);
    }

     public static int lengthofLIS(int[] nums, int prev, int curpos) {
        if (curpos == nums.length) return 0;
        int taken = 0;
        if (nums[curpos] > prev) {
            taken = 1 + lengthofLIS(nums, nums[curpos], curpos + 1);
        }
        int notTaken = lengthofLIS(nums, prev, curpos + 1);
        return Math.max(taken, notTaken);
    }
```

## 动态规划， O（n^2）
dp存储长度，状态转移方程, dp[i]=max(dp[j])+1, j属于[0,i)
```java
    public static int lengthOfLIS(int[] nums) {
        int length = nums.length;
        if (length == 0) return 0;
        int[] dp = new int[length];
        dp[0] = 1;
        int maxAns = 0;
        for (int i = 0; i < length; i++) {
            int max = 0;
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j])
                    max = Math.max(dp[j], max);
            }
            dp[i] = max + 1;
            maxAns = Math.max(dp[i], maxAns);
        }
        return maxAns;
    }
```

## 动态规划， O（nlogn）
dp存储子序列![](/img/LeetCode/300-1.png)。如果当前元素大于子序列中最后一个元素，则加入到子序列末尾，否则，二分查找序列中大于等于当前元素的位置，用当前元素替代该位置对应的元素。
```java
 public static int lengthOfLIS(int[] nums) {
        int length = nums.length;
        if (length == 0) return 0;
        List<Integer> dp = new ArrayList<>();
        for (int num : nums) {
            int size = dp.size();
            if (size == 0 || dp.get(size - 1) < num) dp.add(num);
            else {
                int i = 0, j = size - 1;
                while (i < j) {
                    int mid = (i + j) >> 1;
                    if (num > dp.get(mid)) {
                        i = mid + 1;
                    } else j = mid;
                }
                dp.set(i, num);
            }
        }
        return dp.size();
    }
```