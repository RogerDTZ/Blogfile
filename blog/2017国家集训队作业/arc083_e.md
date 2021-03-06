## Description

​	给一棵$n$个节点的树，和一个长度同样为$n$的非负整数序列$x_i$。

​	请尝试对每个节点染黑或白两种颜色，并确定一个非负整数权值。

​	问是否存在一种方案，使得每个点$i$满足其子树内与其同色的点的权值之和恰好为$x_i$。

​	$1 \le n \le 1000$

​	$0 \le x_i \le 5000$



## Solution

​	一道好题。

​	我们自底向上逐步确定权值。假设当前正在考虑$u$这个点。

​	我们发现，$u$本身选黑还是白并不重要，其子树中的点只有两种：颜色和$u$相同的点，以及颜色和$u$不相同的点。

​	能确定的是，前一类的点的权值值和必须恰好等于$x_u$，而在这个条件下，后者的点的权值值和可以有多种情况。

​	回头看一下，对于一种合法方案，单看每一种颜色形成的树的话，每个点$u$都要满足“后继”（单看一个颜色形成的树意义下的后继）的$x$之和小于等于$x_u$，这样一来$u$的权值可以设为一个恰好的值，使得和调整为$x_u$。

​	显然我们可以对每个点$u$维护一个值$f_u$，表示在满足与$u$颜色相同的点的权值值和恰好为$x_u$时，与$u$颜色不同的点的权值之和最小是多少。这是一个贪心的思想，由于一种颜色的权值和只能是$x_u$，因此我们尽量保证另一种颜色的权值之和最小，以最大化这种颜色在之后考虑父亲节点时满足“小于等于”的可能性。

​	设$g_i$表示当前与$u$同色权值值和为$i$时，另一个颜色的最小值是多少。

​	初始有$g_0=0,\;g_{i}=\infty (i>0)$

​	对于每个后继$v$，转移
$$
g_i+f_v\rightarrow g_{i+x_v}\\g_i+x_v\rightarrow g_{i+f_v}
$$
​	注意是非继承转移，即每层强制转移。

​	最后$f_u=\min g_i\;\;(0 \le i \le x_u)$

​	如果$f_1=\infty$就无解，否则有解。

​	此题关键是想到贪心的那一步。如果不贪心的话，相当于每个点有多个状态（另一个颜色有多种和）等待转移，根本做不了。

​	



## Code

```c++
#include <cstdio>
using namespace std;
const int N=1005,X=5005,INF=1e9;
int n,a[N];
int f[N],g1[X],g2[X];
int h[N],tot;
struct Edge{int v,next;}e[N*2];
inline void addEdge(int u,int v){
	e[++tot]=(Edge){v,h[u]}; h[u]=tot;
	e[++tot]=(Edge){u,h[v]}; h[v]=tot;
}
inline int min(int x,int y){return x<y?x:y;}
void readData(){
	scanf("%d",&n);
	int fa;
	for(int i=2;i<=n;i++){
		scanf("%d",&fa);
		addEdge(fa,i);
	}
	for(int i=1;i<=n;i++) scanf("%d",a+i);							
}
void dfs(int u,int fa){
	bool have=false;
	for(int i=h[u],v;i;i=e[i].next)
		if((v=e[i].v)!=fa){
			have=true;
			dfs(v,u);
		}
	if(!have){
		f[u]=0;
		return;
	}
	for(int i=0;i<=a[u];i++) g1[i]=INF;
	g1[0]=0;
	for(int i=h[u],v;i;i=e[i].next)
		if((v=e[i].v)!=fa){
			for(int j=0;j<=a[u];j++) g2[j]=INF;
			for(int j=0;j<=a[u]&&(j+a[v]<=a[u]||j+f[v]<=a[u]);j++)
				if(g1[j]!=INF){
					if(j+a[v]<=a[u])
						g2[j+a[v]]=min(g2[j+a[v]],g1[j]+f[v]);
					if(j+f[v]<=a[u])
						g2[j+f[v]]=min(g2[j+f[v]],g1[j]+a[v]);
				}
			for(int j=0;j<=a[u];j++) g1[j]=g2[j];
		}
	f[u]=INF;
	for(int i=0;i<=a[u];i++) f[u]=min(f[u],g1[i]);
}
int main(){
	readData();
	dfs(1,0);
	puts(f[1]==INF?"IMPOSSIBLE":"POSSIBLE");
	return 0;
}
```

​	

​	