# 图
边的定义：  
```cpp
struct Edge{
    int to, next,w;
};
```
用数组存储所有边:`int edgs[2*nums];`（无向边要双倍数组）  
一个全局变量记录当前边位置`int cnt=-1;`
添加边：  
```cpp
void add(int u, int v, int w)
{
    edges[++cnt]={v, head[u],w};
    head[u]=cnt;
}
```
`head[u]`表示当前节点`u`的最新边，此节点新加入的边的`next`为此边，再把`head[u]`设为新边，因此形成一个单向链表，可以通过`next`遍历完，当遇到`-1`时结束。  
当边存储完之后，`head[u]`存的是边`u`的头节点，可以通过这个结点遍历找到u上的边。  
`memset(dst,num,size);`把目标位置size字节的位置设为num，注意是单个字节设为num，因此如果是int，只能设为0或-1。  
深度优先搜索：  
```cpp
//u是到达的节点，f是其父节点，totalw是总的源节点到u的距离
//不应该带环，带环需要额外处理
//效果是找到最远点和距离
void dfs(int u,int f, int totalw)
{
    if(totalw>maxdis)
    {
        maxdis=totalw;
        maxv=u;//最远到达的点
    }
    //e如果是-1，则是末端节点，不用循环
    for(int e=head[u];e!=-1;e=edges[e].next)
    {
        int v=edges[e].to;//e这条边指向的另一个点
        //如果指向父节点，则不用遍历这个点
        if(v==f)
        continue;
        //否则需要遍历
        dfs(v ,u, totalw+edges[e].w);
    }
    return;
}
```
```cpp
#include <iostream>
#include <cstring>
using namespace std;

//maxv：源点能到的最远点，maxdis:最远点对应的距离, 
const int maxn = 1e4 + 5;
struct Edge { int to, next, w; }edges[2 * maxn];
int head[maxn], maxdis,maxv, ne; 

void add(int u, int v, int w) {
	edges[ne] = { v, head[u], w };
	head[u] = ne++;
}

//u：dfs的源点，f: u点的父节点，d2s：u点到源点的距离
void dfs(int u, int f, int d2s) {
	if (maxdis < d2s){
		maxdis = d2s;
		maxv = u;
	}
	for (int e = head[u]; e != -1; e = edges[e].next) {
		int v = edges[e].to, w = edges[e].w;
		if (v == f) continue;  //父节点已经访问过，防止重复遍历，相反孩子不会重复遍历。
		dfs(v, u, d2s + w);
	}
}

int main() {
	int e, u, v, w, s;
	cin >> e;
	memset(head, -1, sizeof(head));
	for (int i = 1; i <= e; i++) {
		cin >> u >> v >> w;
		add(u, v, w), add(v, u, w);
	}
	dfs(1, -1, 0); //从结点1开始遍历，找到最远点maxv及对应的最远距离maxdis
	maxdis = 0;
	dfs(maxv, -1, 0);//从结点maxv开始遍历，找到最远点对应的距离maxdis
	cout << maxdis << endl;
	return 0;
}

```