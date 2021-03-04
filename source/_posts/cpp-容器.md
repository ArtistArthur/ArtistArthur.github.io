---
title: cpp 容器
top: false
cover: false
toc: true
mathjax: true
date: 2021-02-16 00:04:57
password:
summary:
tags:
categories:
---
一个容器就是一些特定类型对象的集合.  
顺序容器为程序员提供了控制元素存储和访问顺序的能力.这种顺序不依赖于元素的值,而是与元素加入容器时的位置相对应.  
相对的,有序和无序关联容器,则根据关键字的值来存储元素.  
标准库还提供了三种容器适配器,分别为容器操作定义了不同的接口,来与容器类型适配.容器适配器是标准库中的一个通用概念.容器,迭代器和函数都有适配器.  
本质上,一个适配器是一种机制,能使某种事物的行为看起来像另一种事物一样.   
一个容器适配器接受一种已有的容器类型,使其行为看起来像一种不同的类型.例如`stack`适配器接受一个顺序容器(`array`或`forward_list`除外),并使其操作起来像一个stack一样.
# 顺序容器
<!-- more -->
## 容器的选择
## 容器公有的操作
一般来说每个容器都定义在一个头文件中,文件名与类型名相同.容器均为模板类,需要额外的类型信息生成特定的容器.  
顺序容器几乎可以保存任意类型的元素,特别是我们可以定义一个容器,其元素类型是另一个容器.`vector<vector<int>>`  
但是某些容器操作对元素类型有自己的特殊要求,如果某种类型不支持这种特殊操作,也可以定义这种容器,但是只能使用没有特殊要求的操作了:比如顺序容器的某个构造函数可以接受容器大小参数,它使用了元素的默认构造函数,而如果容器的元素没有实现这个默认构造函数,将不能用这个操作.  
```cpp
vector<noDefault> v1(10,init);//正确,提供了元素初始化器
vector<noDefault> v2(10);//错误,必须提供一个元素初始化器,或者提供一个默认构造函数
```
共有的操作:
* iterator 此容器类型的迭代器类型
* const_iterator 可以读取元素,但是不能修改元素,是一个迭代器类型
* size_type 无符号整数类型,足够保存此种容器类型最大可能大小
## 迭代器
迭代器类似与指针，提供了对对象的间接访问，并可以通过`++`和`--`从一个元素移动到另一个元素，通过解引用`*`来获取元素。  
有迭代器的类型同时拥有返回迭代器的成员，这些类型都有名为`begin`和`end`的成员。  
`begin`成员负责返回指向第一个元素的迭代器，`end`负责返回指向容器“尾元素的下一个位置”的迭代器，也就是说该迭代器指向的是容器的一个本不存在的“尾后”元素。  
尾后迭代器没有什么实际含义，仅仅是个标记，对尾后迭代器解引用会出现未定义的行为。  
如果容器为空，`begin`和`end`返回的都是尾后迭代器。  
### 迭代器的运算符
`*iter`返回迭代器iter指向元素的引用（不可对尾后迭代器解引用）  
`iter->mem`解引用iter并获取该元素名为mem的成员，等价于`(*iter).mem`  
`++`、`--`指向容器中的上一个（下一个）元素，`forward_list`不支持递减  
`==`、`!=`判断两个迭代器是否相等（指向同一个元素）。  
### 迭代器类型
我们并不需要知道迭代器的准确类型（即返回的类型，int* char*等)，而可以直接通过`类名::iterator`来获取：  
`vector<int>::iterator it;`  
`string::iterator it2;`  
`vector<int>::const_iterator it3`    
`string::const_iterator it4;`  
const_iterator和常量指针类似，能读取但是不能修改元素。  
`cbegin`和`cend`来返回`const_iterator`。  
### 迭代器失效
某些对vector对象的操作会使迭代器失效，当vector容量变化时，迭代器都有可能失效。  
任何能引起vector容量变化的操作，比如`push_back`都可能使迭代器失效。  
**谨记**：任何使用迭代器的循环体，都不要向迭代器所属的容器中添加元素，这有可能会导致迭代器失效。（因为迭代器和容器元素指针有关，容器扩容时可能会改变元素地址）   
向容器中添加元素和从容器中删除元素的操作可能会使指向容器元素的指针引用或迭代器失效，一个失效的指针、引用或迭代器将不再表示任何元素，使用它们会引起和未初始化指针一样的问题。  

 

## vector 
vector:可变大小数组,支持快速随机访问,在尾部之外的位置插入或者删除元素可能很慢.  
vector的实现:

## deque
双端队列.支持快速随机访问,在头尾插入/删除速度很快

## list
双向链表.只支持shuang'xiang顺序访问.在任何位置插入删除速度都很快.

## forward_list
单向链表.只支持单向顺序访问.在链表任何位置进行插入删除速度都很快.

## array
固定大小数组.支持随机快速访问.不能添加或删除元素.

## string
与`vector`相似的容器.但专门用于保存字符.随机访问快.在尾部插入删除速度快.

# 关联容器
关联容器和顺序容器有着根本的不同:关联容器中的元素是按关键字来保存和访问的.  
关联容器支持高效的关键字查找和访问.两个主要的关联容器类型是`map`和`set`.  
`map`中的元素是一些关键字-值对:关键字起索引的作用,值则表示与索引相关联的数据.   
`set`中每个元素只包含一个关键字,它支持高效的关键字查询操作,检查一个给定关键字是否在set中.  
在某些文本处理过程中,可以用一个set来保存想要忽略的单词.  
map可以实现字典:用单词作为关键字,单词释义作为值.  
标准库提供8个关联容器.这8个关联容器的不同体现在三个方面:  
1. 每个容器要么是一个set,要么是一个map
2. 要么允许重复关键字,要么不允许重复关键字.允许重复关键字的容器的名字中都包含单词`multi`
3. 要么按顺序保存元素,要么无序保存元素.不保持关键字顺序的容器都以单词`unordered`开头.    
因此:  
1. `map`:保持元素顺序不允许重复的map
2. `set`:保持元素顺序不允许重复的set
3. `multimap`:保持元素顺序允许重复的map
4. `multimap`:保持元素顺序允许重复的set
5. `unordered_map`:不保持元素顺序不允许重复的map
6. `unordered_set`:不保持元素顺序不允许重复的set
7. `unordered_multimap`:不保持元素顺序允许重复的map
8. `unordered_multiset`:不保持元素顺序允许重复的set
`map`和`multimap`在头文件`map`里,`set`和`multiset`在头文件`set`里.无序容器在头文件`unordered_map`和`unordered_set`里.    
### map
经典的使用`map`的例子是单词计数器:  
```cpp
//统计每个单词在输入中出现的次数
map<string, size_t> word_count;//string到size_t的空map
string word;
while(cin>>word)
++word_count[word];
for(const auto& w:word_count)
{
    cout<<w.first<<" occurs "<<w.second<<((w.second>1?" times":" time")<<endl;
}
```
此程序读取输入,报告每个单词出现多少次.  
当对word_count进行下标操作时,word还未在map中,下标运算会创建一个新元素,关键字为word,值为0(值初始化).  
`for(const auto&w:word_count)`中w会得到一个`pair`类型的对象.  
### set
`pair<set<int>::iterator, bool> set<int>::insert(int key_value);`插入函数，通过键值插入，返回一个pair，first代表插入元素的迭代器，second代表是否插入成功，插入失败是指此元素以及在set里了。  

单词统计程序的合理拓展是:忽略常见单词,如"the","and","or"等.可以使用set保存想忽略的单词,只对不在集合中的单词统计出现次数:  
```cpp
//统计每个单词在输入中出现的次数
map<string, size_t> word_count;//string到size_t的空map
set<string> exclude={"the","but", "or", "and", "an", "a"};
string word;
while(cin>>word)
//只统计不在set中的
if(exclude.find(word)==exclude.end())
++word_count[word];
for(const auto& w:word_count)
{
    cout<<w.first<<" occurs "<<w.second<<((w.second>1?" times":" time")<<endl;
}
```
与其他容器类似,set也是模板.为了定义一个set,必须指定其元素类型.  
`find()`调用返回一个迭代器.如果给定关键字在set中,迭代器指向该关键字.否则,返回尾后迭代器.  

## pair
`pair`是一个标准库类型,定义在头文件`utility`中.  
一个`pair`,保存两个数据成员.类似容器,`pair`是一个用来生成特定类型的模板.当创建一个`pair`时,必须提供给它两个类型名,pair的数据成员将具有对应的类型.两个类型要求不同:  
* `pair<string, string> anon;`保存两个string 
* `pair<string, size_t> word_count;`保存一个string和一个size_t
* `pair<string, vector<int>> line;`保存一个string和`vector<int>`     

