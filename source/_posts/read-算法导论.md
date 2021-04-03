# 算法导论
## 并查集
无向图中的连通分量的计算[leetcode 323](https://leetcode-cn.com/problems/number-of-connected-components-in-an-undirected-graph/) 
[并查集模板](https://www.luogu.com.cn/problem/P3367)   
要记得初始化：
```cpp
void initf(int n)
{
    for(int i=0;i<n;i++)
    {
        f[i]=i;
    }
    return;
}
```

```cpp
int f[maxn];
int find(int k) 
{
    //带路径压缩
    return f[k]==k?k:f[k]=find(f[k]);
}
void merge(int m, int n)
{
    f[find(m)]=find(n);
    return ;
}
```
链式并查集的一种表示方法是通过数组，`f[k]`表示第k个元素的所属集合（有时未更新，需要通过`f[k]=find(f[k])`更新），这个集合由集合中的一个节点表示，同一个集合的代表结点应该相同，这个代表节点很特殊，以它自己为下标的数组元素值为他自己：`f[head]==head`。  
并查集主要是两个操作：查找和合并。查找通过递归完成：
```cpp
int find(int k)
{
    if(f[k]==k)
    return k;
    return find(k);
}
```
这种查找在集合连接方式变成一条链时效率很低，复杂度为`O(n)`，此时可以通过路径压缩优化：在寻找代表结点时把路径上的结点的代表结点一并更新：
```cpp
int find(int k)
{
    if(f[k]==k)
    return k;
    return f[k]=find(k);
}
```
优化后可以使得后续的查询的复杂度变为`O(lgn)`
合并比较简单，把代表结点的代表结点值设为另一个集合就行：
```cpp
void merge(int m, int n)
{
    f[find(m)]=find(n);
    return;
}
```
最后怎么计算有多少个连通分量呢？  
遍历所有结点，记下祖先是自己的结点。  
或者通过set记录。  

## 图
### 广度优先
22.2-7 职业摔跤手可以分为两种各类型：“娃娃脸”（“好人”）型和“高跟鞋”（“坏人”）型。在任意一堆职业摔跤手之间都有可能存在竞争关系。假定有 n 个职业摔跤手，并且有一个给出竞争关系的 r 对摔跤手的链表。请给出一个时间为 O( n+r ) 的算法来判断是否可以将某些摔跤手划分为“娃娃脸”型，而剩下的划分为“高跟鞋”型，使得所有的竞争关系均只存在于娃娃脸型和高跟鞋型选手之间。如果可以进行这种划分，则算法还应当生成一种这样的划分。  
ANSWER：相当于划分成二分图。  
1. 二分图判断：给每个结点增加一个属性，定为 good（好人）或者 bad（坏人）。链表中的结点的属性必与该链表头结点属性相反。按照这个规则遍历每个链表，第一个链表表头结点属性设为 good，若在遍历过程，发现新定义的属性与先前定义的属性冲突，则不可以划分；反之，则可以划分，并且属性为 good 和 bad 的结点就是相应的两类。
2. 用染色法，即从其中一个顶点开始，将跟它邻接的点染成与其不同的颜色，如果邻接的点有相同颜色的，则说明不是二分图，每次用bfs遍历即可。  
```cpp
#include <queue>
#include <cstring>
#include <iostream>
using namespace std;

const int N = 510;
int color[N], graph[N][N];

//0为白色，1为黑色 
bool bfs(int s, int n) {
    queue<int> q;
    q.push(s);
    color[s] = 1;
    while(!q.empty()) {
        int from = q.front();
        q.pop();
        for(int i = 1; i <= n; i++) {
            if(graph[from][i] && color[i] == -1) {
                q.push(i);
                color[i] = !color[from];//染成不同的颜色 
            }
            if(graph[from][i] && color[from] == color[i])//颜色有相同，则不是二分图 
                return false;
        }
    }
    return true;     
}

int main() {
    int n, m, a, b, i;
    memset(color, -1, sizeof(color));
    cin >> n >> m;
    for(i = 0; i < m; i++) {
        cin >> a >> b;
        graph[a][b] = graph[b][a] = 1; 
    }
    bool flag = false;
    for(i = 1; i <= n; i++)
        if(color[i] == -1 && !bfs(i, n)) {//遍历各个连通分支 
            flag = true;
            break;  
        }
    if(flag)
        cout << "NO" <<endl;    
    else
        cout << "YES" <<endl;
    return 0;
}
```




* 22.2-8 我们将一棵树 T = (V, E) 的直径定义为 max (u, v∈V) δ(u, v)，也就是说，树中所有最短路径距离的最大值即为树的直径。请给出一个有效算法来计算树的直径，并分析算法的运行时间。  
ANSWER：根据树的直径的性质：树的直径的路径的两个端点之一必定是以任一结点为源结点的最大路径的末结点。所以运行两次 BFS 搜索，第一次随机选择一个结点作为源结点；第二次选择第一次搜索结果中路径最大值的结点作为源结点，第二次 BFS 搜索得出的最大路径即为树的直径。时间复杂度为O(V+E)  
证明：假设路径v-w为树的直径
1）`u`位于`v-w`所在的路径上，那么从`u`点做DFS能访问到的最远点必然是`v`或`w`, 否则假设访问到的最远点为x, 有`dist(u,x)≥dist(u,v)`，`dist(u,x)≥dist(u,w)`,分两种情况讨论：  
a)如果只取大于号，`dist(u,x)>dist(u,v)`，`dist(u,x)>dist(u,w)`，那么d`ist(u,x)+dist(u,v)=dist(v,x)&gt;dist(u,v)+dist(u,w)=dist(v,w)`那么v-w不是树的是直径，跟假设矛盾。
b)如果取大于等于号，`dist(u,x)≥dist(u,v)`，`dist(u,x)≥dist(u,w)`假设`dist(u,x)=dist(u,v)``dist(u,x)=dist(u,v)`那么`dist(x,w)=dist(v,w)`，这样也没问题，树的直径不唯一而已，那么x依然位于树的直径的一个端点上。
2）`u`不位于`v-w`所在的路径上，那么有`dist(u,x)>dist(u,y,v)`，这里`y`是`u`到路径`v-w`的任意点，那么就有`dist(u,x)+dist(u,y,w)=dist(x,v)>dist(u,v)+dist(u,y,w)=dist(v,w)`那么说明那么v-w不是树的是直径，跟假设矛盾。
![在最大路径上](https://i.bmp.ovh/imgs/2021/03/db18a1effaf4118d.png)  
![不在最大路径上](https://i.bmp.ovh/imgs/2021/03/632f2031fb5328e6.png)  

### 深度优先

### 强连通分量
图`G=(V,E)`的转置：