---
title: time33
top: false
cover: false
toc: true
mathjax: true
date: 2021-02-04 16:40:14
password:
summary:
tags:
categories:
---

### 字符串hash算法time33
time33是一种字符串hash算法,其中使用了magic number 5381(001 010 100 000 101)原因是:
```
Magic Constant 5381:
1. odd number
2. prime number
3. deficient number
```
这些特性使得它hash后的结果分布更好.
<!-- more -->
```c

#include<stdio.h>

unsigned int time33(const char*);
int main()
{
    char str[]="hash_func";
    unsigned int res=time33(str);
    printf("%ud",res);
    return 0;
}
unsigned int time33(const char* strx)
{

    unsigned int hash=5381;
    while(*strx)
    {
        hash=hash*33+(unsigned int)*strx;//或者用位运算
        strx++;
    }
    return hash;
}

```

https://www.cnblogs.com/napoleon_liu/articles/1911571.html   


## 数字hash
哈希冲突解决办法最常用的就是开发定址法和链地址法。链地址法的原理是如果遇到冲突，就在原地址新建一个空间，然后以链表结点的形式插入到该空间。  
最主要两个函数:`insert`和`find`
```c
struct node
{
    struct node *next;
    int key;
    int value;
    int valid;
};

int hash(int key)
{
    if (key < 0)
        key = -1 * key;
    return key % 997;
}
struct node hash_table[997];
void insert(struct node *head, int value, int key)
{
    if (head->valid == 0)
    {
        head->key = key;
        head->value = value;
        head->valid = 1;
        head->next = NULL;
        return;
    }
    while (head->next != NULL)
    {
        head = head->next;
    }
    head->next = (struct node *)malloc(sizeof(struct node));
    head->next->key = key;
    head->next->value = value;
    head->next->valid = 1;
    head->next->next = NULL;
    return;
}
int find(struct node *head, int value)
{
    while (head != NULL)
    {
        if (head->valid == 1)
        {
            if (head->value == value)
                return head->key;
        }
        head = head->next;
    }
    return -1;
}

```
cpp中STL的hash表:C++ STL unordered_map容器  
unordered_map<key,T> tmp;  
auto it=tmp.find(key);//如果key存在,则返回一个迭代器,如果不存在,则返回尾迭代器tmp.end()  
it->second;//表示T  
it->first;//表示key

```cpp

class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        std::unordered_map<int,int> hashtable;
        for(int i=0;i<nums.size();i++)
        {
            auto it=hashtable.find(target-nums[i]);
            if(it!=hashtable.end())
            {
                return {it->second,i};
            }
            hashtable[nums[i]]=i;
        }
    }
};


```