---
title: Josephus problem(约瑟夫问题)
top: false
cover: false
toc: true
mathjax: true
date: 2020-06-26 04:48:29
password:
summary:
tags: algorithms
categories: algorithms

---
# Josephus problem（约瑟夫问题）
## 问题描述
链接：http://bailian.openjudge.cn/practice/2746/    
>约瑟夫问题：有ｎ只猴子，按顺时针方向围成一圈选大王（编号从１到ｎ），从第１号开始报数，一直数到ｍ，数到ｍ的猴子退出圈外，剩下的猴子再接着从1开始报数。就这样，直到圈内只剩下一只猴子时，这个猴子就是猴王，编程求输入ｎ，ｍ后，输出最后猴王的编号。
<!--more-->
来源：
* 约瑟夫是犹太军队的一个将军，在反抗罗马的起义中，他所率领的军队被击溃，只剩下残余的部队40余人，他们都是宁死不屈的人，所以不愿投降做叛徒。一群人表决说要死，所以用一种策略来先后杀死所有人。
* 于是约瑟夫建议：每次由其他两人一起杀死一个人，而被杀的人的先后顺序是由抽签决定的，约瑟夫有预谋地抽到了最后一签，在杀了除了他和剩余那个人之外的最后一人，他劝服了另外一个没死的人投降了罗马。

那么如何选择一个位置以使得你变成那剩余的最后一人？

这个问题可以转换成递推问题：$J(n,k)=(J(n-1,k)+k)\%n$

* $J(n,k)$表示人数为n，报数节点为k，最后剩下的人的编号。
* 对于$J(n,k)$,可以转换成 $J(n-1,k)$
* 当人数为$n$时，进行第一次报数，第$k$个人退出，此时剩余$n-1$个人。
* 若重新排号，第$k+1$个人为1号，那么此时问题为 $J(n-1,k)$,而此时求出的结果的编号是人数为$n-1$并且重新排号的情况，$J(n,k)$与$J(n-1,k)$编号的关系为 $J(n,k)=(J(n-1,k)+k)\%n$ \(自行体会\)。
    
注意事项：重新编号应该从0开始，最后要和题目的编号方式比较再输出。

```cpp  
#include <stdio.h>
#include <stdlib.h>
int main()
{
    int n, m;

    scanf("%d %d", &n, &m);
    
    while(n!=0&&m!=0)
    {
        int temp = 0;
 
        for(int i=1; i < n; i++)//从1推到n
        {
            temp = (temp + m) % (i + 1);
        }
  
        printf("%d\n", temp+1);
 
        scanf("%d %d", &n, &m);
    }
        return 0;
}


```

约瑟夫变体：求最后x个的编号

链接：https://vjudge.net/problem/UVA-1452

```cpp
#include<stdio.h>
#include<stdlib.h>
int main()
{
    int n, m;
    int T;
    scanf("%d", &T);
    for (int i = 0; i < T;i++)
    {
        scanf("%d %d", &n, &m);
        int temp ;
        temp =((m % 3)+2)%3;//剩3个时的编号
        
        for (int i = 3; i < n ; i++) //从3推到n
        {
            temp = (temp + m) % (i + 1);
        }
        printf("%d ", temp + 1);

        temp = ((m % 2)+1)%2;
        
        for (int i = 2; i < n; i++) //从2推到n
        {
            temp = (temp + m) % (i + 1);
        }
        printf("%d ", temp + 1);
        temp = 0;
        for (int i = 1; i < n; i++) //从1推到n
        {
            temp = (temp + m) % (i + 1);
        }
        printf("%d\n", temp + 1);
    }  
        return 0;
}

```








