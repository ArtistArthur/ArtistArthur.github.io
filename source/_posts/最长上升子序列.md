---
title: 最长上升子序列
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-30 15:35:19
password:
summary:
tags: [DP,线性DP]
categories: [DP,线性DP]
---

# 最长上升子序列
* leetcode300 https://leetcode-cn.com/problems/longest-increasing-subsequence/  
* 动态规矩、线性动态规划（一维）
<!--more-->
## 思路
* 暴力：遍历每一个子集中的元素,遍历$2^n$次
* $O(n^2)$复杂度的动态规划：
  * $dp[i-1]$代表第$i$个元素作为序列最后一个数时，最大的上升子序列长度；当遍历第$i$个数时$dp[i]=max\{dp[j]+1(j<i,xj<xi)\}$
  * 证明，归纳法（假设序号从1开始）：
    * $i=1$时显然成立；
    * 当$i<n$时假设成立：$dp[i]$表示$i$个元素作为序列最后一个数时，最大的上升子序列长度;
    * $i=n$时，若$dp[n]$不是以$xn$结尾的最大上升序列，即存在另一个以$xn$结尾的最大上升序列，而这个序列去掉$xn$后一定属于$dp[n]$之前的一个序列，而$dp[n]$是里面符合情况最好的，矛盾。
* $O(nlgn)$复杂度的动态规划：
  * $dp[i]$代表$i+1$长度的上升序列中，末尾最小的那个序列的末尾数，显然$dp[i]$是单调上升的（假设不单调上升，存在一个$i$,$dp[i]>=dp[i+1]$则让$dp[i+1]$代表的那个序列去掉末尾，则新的末尾小于$dp[i]$矛盾），当遍历到$xj$时，用二分法找到$dp[i-1]<xj<=dp[i]$插入并把$dp[i]$的值更新为$xj$（新的长度为$i+1$符合条件的序列是：长度为$i$的末尾最小序列$+xj$）,如果$xj$是最大的，$maxlen++$,$dp[maxlen]=xj$这个代表的序列是$dp[maxlen-1]$代表的那个序列$+xj$。
  * 证明：归纳法+反证法或者推理。
  * 二分查找：begin=mid+1, end=mid;

~~~cpp

              while (lo < hi)
            {
                mid = (lo + hi) >> 1;
                if (dp[mid] == nums[i])
                {
                    hi = mid;
                    break;
                }
                else if (dp[mid] < nums[i])
                    lo = mid + 1;
                    //mid的比较方向对最后的结果也会有影响：
                    //如果先判断目标数是在mid-hi之间
                    //那么最后结果会是最终的hi的位置不小于目标数
                else
                {
                    hi = mid;
                }
            }
~~~
## 代码
* $O(n^2)$

~~~cpp

#include<vector>
#include<iostream>
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
    using namespace std;
    vector<int> dp;
    int max_LIS=0;
    for(int i=0;i<nums.size();i++)
    {
        int maxn=0;
           for(int j=0;j<i;j++)
           {
               if(nums[j]<nums[i])
               {
                   if(dp[j]>maxn)
                   {
                       maxn=dp[j];
                   }
               }
           }
           dp.push_back(maxn+1);
           if(max_LIS<dp[i])
           max_LIS=dp[i];
    }
    return max_LIS;

    }
};

~~~

* $O(nlgn)$

~~~cpp

class Solution
{
public:
    int lengthOfLIS(vector<int> &nums)
    {
        vector<int> dp;
        for (int i = 0; i < nums.size(); i++)
        {

            int lo = 0, hi = dp.size();
            int mid;

            while (lo < hi)
            {
                mid = (lo + hi) >> 1;
                if (dp[mid] == nums[i])
                {
                    hi = mid;
                    break;
                }
                else if (dp[mid] < nums[i])
                    lo = mid + 1;
                else
                {
                    hi = mid;
                }
            }
            if (hi == dp.size())
            {
                dp.push_back(nums[i]);
            }
            else
            {
                dp[hi] = nums[i];
            }
        }

        return dp.size();
    }
};

~~~