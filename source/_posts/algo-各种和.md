# 分割和
[leetcode 416.分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/)`01`背包，指定背包容量。  
背包递归写法：
```cpp
int va[1005];
int n;
int ma[1005][1005];
int dp(int nums,int cap)    
{
    if(nums<0||cap<=0)
        return 0;
    if(ma[nums][cap]!=-1)
        return ma[nums][cap];
    return dp(nums - 1, cap - va[nums]) + va[nums] < dp(nums - 1, cap) ? dp(nums - 1, cap) : dp(nums - 1, cap - va[nums]) + va[nums];
}
```
<!--more-->

背包循环写法：  
```cpp
int ma[1005][1005];
int ca[1005];
int w[1005];
memset(ma,0,sizeof(ma));
for(int i=1;i<=n;i++)
{
    for(int j=1;j<=cap;j++)
    {
        if(j<ca[i])
        ma[i][j]=ma[i-1][j];
        ma[i][j]=max(ma[i-1][j],ma[i-1][j-ca[i]]+w[i]);
    }
    //一种更好的for写法,i表示第i件物品，j表示容量，当j小于当前物品容量时，不改变值，是原来的值，即i-1的值
    //但是使用二维数组会出错，以为此时没有更新i时的值，因此需要用一维数组，表示上一轮的最优解，因为每次只需要用到上一轮的信息
    for(int j=cap;j>=va[i];j--)
    {
        ma[i][j]=max(ma[i-1][j],ma[i-1][j-ca[i]]+w[i]);
    }
    //因此应该使用空间优化后的一维数组
    for(int j=cap;j>=va[i];j--)
    {
        ma[j]=max(ma[j],ma[j-ca[i]]+w[i]);
    }
}
```