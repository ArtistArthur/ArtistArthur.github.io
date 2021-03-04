---
title: 最大子序和
top: false
cover: false
toc: true
mathjax: true
date: 2020-11-01 14:21:48
password:
summary:
tags: [DP,线性DP]
categories: [DP, 线性DP]
---
## 简单的线性规划 
* https://leetcode-cn.com/problems/maximum-subarray/
* $dp[i]$代表以当前数结尾的序列最大和
* 转移方程是$dp[i]=max\{dp[i-1]+xi , xi\}$
<!--more-->
* 动态规划代码$O(n)$  

~~~cpp

class Solution
{
public:
    int maxSubArray(vector<int> &nums)
    {
        int dp[nums.size()];
        int maxn = nums[0];
        for (int i = 0; i < nums.size();i++)
        {
            if(i==0)
            {
                dp[i] = nums[i];
                continue;
            }
            dp[i] = dp[i - 1] + nums[i] > nums[i] ? dp[i - 1] + nums[i] : nums[i];
            if(dp[i]>maxn)
            {
                maxn = dp[i];
            }
        }
        return maxn;
    }
};

~~~