## Description

	[题目链接](https://arc079.contest.atcoder.jp/tasks/arc079_d)

	大意：给一张基环外向树。要求给每一个点确定一个值，其值为所有后继点的$\text{mex}$。求是否存在确定权值方案。



## Solution

	首先，对于叶子节点，其权值必定是0.

	对于每一棵外向树，树上的每个点的权值都是唯一确定的。可以通过DFS计算得到。

	然而，每棵外向树的根——环上的某个点$u$，其权值不是唯一确定的。因为它要考虑的后继，不仅包括在树上的后继，还有一个环上后继。

	根据$\text{mex}$的性质，我们发现不管$u$的环上后继是多少，$u$始终只有两种取值：分别是只考虑树上后继时的一级$\text{mex}$与二级$\text{mex}$。

	再一来可以发现，只要我们确定了环上的某个点$u$的权值，我们可以唯一确定地填出剩余环上点的权值。只需要模拟一圈计算回来，判断最终$u$的权值和开始确定的值是否相同即可。	

	任选一个环上点，两种值各试一次，只要一种OK则存在方案；否则无解。

	要对这个$\text{mex}$的取值比较敏感，才能很快地发现“两种取值”这一性质。确定一个点的值就能唯一确定方案这一点只要不要脑抽退却了应该能够比较顺利地发现。



## Code

```c++
#include <cstdio>
using namespace std;
const int N=200005;
int n,pre[N],cnex[N];
int h[N],tot;
struct Edge{
	int v,next;
}e[N*2];
int a[N],len,mex[N][2];
inline void addEdge(int u,int v){
	e[++tot]=(Edge){v,h[u]}; h[u]=tot;
}
void readData(){
	scanf("%d",&n);
	for(int i=1;i<=n;i++){
		scanf("%d",&pre[i]);
		addEdge(pre[i],i); 
	}
}
void findCircle(int u){
	static int stk[N],top=0;
	static bool vis[N];
	stk[++top]=u;
	vis[u]=true;
	if(vis[pre[u]]){
		int v=pre[u];
		cnex[v]=u;
		for(;stk[top]!=v;top--){
			cnex[stk[top]]=stk[top-1];
			a[++len]=stk[top];
		}
		a[++len]=v;
		return;
	}
	findCircle(pre[u]);
}
void dfs(int u,int dep){
	for(int i=h[u],v;i;i=e[i].next)
		if((v=e[i].v)!=cnex[u])
			dfs(v,dep+1);
	static int cnt[N];
	for(int i=h[u],v;i;i=e[i].next)
		if((v=e[i].v)!=cnex[u])
			cnt[mex[v][0]]++;
	for(mex[u][0]=0;cnt[mex[u][0]];mex[u][0]++);
	if(dep==0)
		for(mex[u][1]=mex[u][0]+1;cnt[mex[u][1]];mex[u][1]++);
	for(int i=h[u],v;i;i=e[i].next)
		if((v=e[i].v)!=cnex[u])
			cnt[mex[v][0]]--;
}
bool run(int x){
	int last=x;
	for(int i=len-1;i>=1;i--){
		int u=a[i];
		last=(mex[u][0]==last)?mex[u][1]:mex[u][0];
	}
	last=(mex[a[len]][0]==last)?mex[a[len]][1]:mex[a[len]][0];
	return x==last;
}
bool judge(){
	for(int i=1;i<=len;i++)
		dfs(a[i],0);
	return run(mex[a[len]][0])||run(mex[a[len]][1]);
}
int main(){
	readData();
	findCircle(1);
	puts(judge()?"POSSIBLE":"IMPOSSIBLE");
	return 0;
}
```

