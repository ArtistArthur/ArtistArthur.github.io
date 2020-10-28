---
title: 单调栈(monotone stack)
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-14 22:44:05
password:
summary:
tags:
categories:
---

#  单调栈（monostone）
* 单调栈顾名思义就是一个单调的栈，可以是单调递增也可以是单调递减的
* 单调栈可以解决的问题是：对于于一个数列，可以找到每一个数的下一个比自己大的或者小的数  
* 比如：$2 3 5 7 8 6 5 1$。对于$2$来说，下一个比它小的数是$1$，对于$8$来说下一个比它小的数是6.
* 朴素的算法是复杂度为$O(n^2)$。并且有两个：
  * $\alpha.$ 第一种方法，往后遍历：对于$2$来说，遍历$2$后面的所有数找出第一个比它小的数
  * $\beta.$ 第二种方法，往前遍历：对于$5$来说，往左遍历，找到第一个比它小或者相等的数$x$,此时$x$和$5$之间的数的下一个比自己小的数就是$5$,而对于$5$来说$x$之前的数被$x$屏蔽了。
* 而使用单调栈可以使复杂度降低为$O(n)$   
* 还是上面的数列，找第一个比自己小的数：
  * $\alpha.$维持一个单调递增的栈，当遇到一个新值 $x$ 时，如果这个值比栈顶的值 $y$ 大或者相等，压入;
  * $\beta.$如果 $x$ 比 $y$ 小，那么把 $y$ 弹出，并且把这个 $y$ 对应的输出置为 $x$ ，回到 $\alpha$
* 单调栈的原理在于：
  * 对于每一个数，它被选中之后就会被弹出不会被重复遍历，总体的查询次数最多是$2n$\(压栈的时候要主动去查询一次栈顶元素查询次数就是$n$，然后如果命中了，需要弹栈再查询,总共需要弹$n$次\)，保证了复杂度是$O(n)$
  * 对于 $x、y$，如果$x\leq y$那么对于$x$前面的数对于$y$来说就是不可见的，或者被$x$屏蔽了,所以不用遍历了
---
## 几道leetcode
$\alpha.$ $739 Daily Temperatures$ <https://leetcode.com/problems/daily-temperatures/>  
>Given a list of daily temperatures T, return a list such that, for each day in the input, tells you how many days you would have to wait until a warmer temperature. If there is no future day for which this is possible, put 0 instead.  
For example, given the list of temperatures T = [73, 74, 75, 71, 69, 72, 76, 73], your output should be [1, 1, 4, 2, 1, 1, 0, 0].  
Note: The length of temperatures will be in the range [1, 30000]. Each temperature will be an integer in the range [30, 100].
* 很显然的单调栈  
  
~~~cpp
#include<vector>
#include<stack>
#include<iostream>
using namespace std;
class Solution
{
public:
    vector<int> dailyTemperatures(vector<int> &T)
    {
        stack<int> small_stk;
        vector<int> ret(T.size(),1);
        for (int i=0;i<T.size();i++)
        {
            if(small_stk.empty())
            {
                small_stk.push(i);
                continue;
            }           
            else if(T[i]<=T[small_stk.top()])
            {
                small_stk.push(i);
                continue;
            }
            else if(T[i]>T[small_stk.top()])
            {
                ret[small_stk.top()] = i - small_stk.top();
                small_stk.pop();
                while(!small_stk.empty() &&T[i]>T[small_stk.top()])
                {
                    ret[small_stk.top()] = i - small_stk.top();
                  
                    small_stk.pop();
                }
                small_stk.push(i);
                continue;
            }
        }
        while (!small_stk.empty())
        {
            ret[small_stk.top()] = 0;
            small_stk.pop();
        }
        return ret;
    }
};
int main()
{
    vector<int >T={73, 74, 75, 71, 69, 72, 76, 73 };
    Solution A;
    vector<int>x=A.dailyTemperatures(T);
    for (int i = 0; i < x.size();i++)
        std::cout << x[i] << std::endl;

}  

~~~    


$\beta.$ $84.Largest\:Rectangle\:in\:Histogram$ <https://leetcode.com/problems/largest-rectangle-in-histogram/>  
* 还有几道题明天写
