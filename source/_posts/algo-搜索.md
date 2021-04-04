# 广度优先搜索
树的广度优先遍历的写法模式相对固定： 
* 使用队列；
* 在队列非空的时候，动态取出队首元素；
* 取出队首元素的时候，把队首元素相邻的结点（非空）加入队列。  
<!--more-->

```cpp
 while (!deq.empty())
         {
             int len = deq.size();
             ret.push_back({});
             for (int i = 0; i < len;i++)
             {
                 auto tmp = deq.front();
                 deq.pop_front();
                 ret.back().push_back(tmp->val);
                 if (tmp->left != nullptr)
                 {
                     deq.push_back(tmp->left);
                 }
                 if (tmp->right != nullptr)
                 {
                     deq.push_back(tmp->right);
                 }
             }
            }
```
在无权图中，由于广度优先遍历本身的特点，假设源点为 source，只有在遍历到所有距离源点source的距离为d的所有结点以后，才能遍历到所有距离源点 source的距离为d + 1的所有结点。也可以使用「两点之间、线段最短」这条经验来辅助理解如下结论：从源点 source 到目标结点 target 走直线走过的路径一定是最短的。  
对于图的广度优先遍历，先建立链式邻接表，然后从第一个链开始搜索，完成之后搜索第二个链，如果已被访问，则跳过。  