leetcode里写在类外面的数组，可能不会被成功初始化，原因是多个样例会多次调用同一个方法，而类的初始化只有一次。  
<!--more-->

```cpp
 int dp[10000] = {0};
 class Solution
 {
 public:
     bool canPartition(vector<int> &nums)
     {  

         memset(dp,0,sizeof(dp));
     }
 };
```
如果没有memset的话不一定为0。