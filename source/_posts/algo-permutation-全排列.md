---
title: permutation(全排列)
top: false
cover: false
toc: true
mathjax: true
date: 2020-06-29 16:36:47
password:
summary:
tags: algorithms
categories: algorithms
---


# 全排列的各种输出方法
## 最朴素的输出方法
* 和乘法原理相同的过程，先确定第一位，再确定第二位，以此类推。
* 回溯

<!--more-->


~~~cpp
#include<stdio.h>
#include<stdlib.h>
void permutation(int *ma,int n, int k)
{
    if(n-1==k)
    {
        for (int i = 0; i < n;i++)
        {
            printf("%d", ma[i]);
        }
        printf("\n");
        return;
    }
   
    for (int i = k; i < n;i++)
    {
        int temp = ma[i];
        ma[i]=ma[k];       
        ma[k] = temp;
        permutation(ma, n, k + 1);
        temp = ma[k];
        ma[k]=ma[i];
        ma[i] = temp;
    }
    return;
}
int main()
{
    int ma[100];
    int n;
    scanf("%d", &n);
    for (int i = 0; i < n;i++)
    {
        ma[i] = i + 1;
    }
    permutation(ma, n, 0);
    return 0;
}
~~~

## 按照字典序输出

* 也是先确定第一位，再确定第二位，以此类推
* 按照字典序要求输出
* 有约束的深度优先DFS
* 和朴素的输出方法不同点是：每次交换位置后，后面位置的数要重新排序，按照从小到大排好，由数学归纳法，假设每次交换位置前顺序都是从小到大，交换位置后，则只需要把交换出去的数放在第一位，后面的数依次往后移动一个位置，也保证了第一位最小。

~~~cpp
#include<stdio.h>
#include<stdlib.h>
void permutation(int *ma,int n, int k)
{
    if(n-1==k)
    {
        for (int i = 0; i < n;i++)
        {
            printf("%d", ma[i]);
        }
        printf("\n");
        return;
    }
    permutation(ma, n, k + 1);
    for (int i = k+1; i < n;i++)
    {
        int temp = ma[i];
        for (int j = i ; j > k;j--)
        {
            ma[j] = ma[j - 1];
        }
           
        ma[k] = temp;
        permutation(ma, n, k + 1);

        temp = ma[k];
        for (int j = k; j <i; j++)
        {
            ma[j] = ma[j + 1];
        }
        ma[i] = temp;
    }
    return;
}
int main()
{
    int ma[100];
    int n;
    scanf("%d", &n);
    for (int i = 0; i < n;i++)
    {
        ma[i] = i + 1;
    }
    permutation(ma, n, 0);

    return 0;
}
~~~


### 注意事项

* 阶乘增长速度比指数还快，很容易爆。
* 2^16=6,5536 
* 2^32=42,9496,7296 
* 2^64=1844,6744,0737,0955,1616
* 8!=40320 
* 9！=36,2880 (爆16位unsigned int) 
* 12!=4,7900,1600 
* 13!=62,2702,0800 (爆32位unsigned int) 
* 20!=243,2902,0081,7664,0000 
* 21!=5109,0942,1717,0944,0000 (爆64位unsigned long long)
### 康托展开及其逆





