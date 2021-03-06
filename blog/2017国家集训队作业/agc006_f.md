## Description

	[题目链接](https://agc006.contest.atcoder.jp/tasks/agc006_f)





## Solution

	首先，把输入矩阵看成邻接矩阵，将问题转化到图上。

	现在的问题变成：给定一个有向图，如果存在$(u,v)$和$(v,w)$，则连边$(w,u)$，重复边不连。求最终状态下，整张图有多少条边。



	第一个思路是各个弱联通块互不影响，既然初始时它们之间无边，则按照题目的操作方式，在之后也不会有连边。因此，我们对每个弱联通块分开考虑。

	从连边数量级较大这一个方面看，猜想每个联通块的连边数量可能是结论相关。手玩一下图。

	如果对每个弱联通块进行三色染色（即从一个点开始染$c$，其到达点染$c+1\mod 3$），可以推出一些东西：

	（1）如果染色冲突。这种情况下，必然出现了自环或二元环。这两种环的出现带来的变动是巨大的——手玩一下可以发现，这整个弱联通块会变成完全图（包括自环边也会出现）。

	（2）如果颜色未用完。这种情况下，题目给出的扩展条件一次都使用不了。因此该弱联通块的边数与原来一样。

	（3）如果染色未冲突，且颜色使用完全。我们可以把点按颜色分成三块。其中块内的点互不连边，0色块通过若干边连向1色，1色连向2色，2色连向0色。且任意点有至少一条出边或入边，否则不属于该弱联通块。有结论：0色内的所有点会各自连向1色内的所有点，1色的所有点将连向2色的所有点，2色同理。因为任意一对处于$x$色的点$u$和$x+1 \mod 3$色的点$v$，都可以通过某种方式连上边。



	实现上，将图看成无向图进行联通块的搜索。每条边记录一下实际方向，以确定下一个点是$x+1 \mod 3$还是$x-1 \mod 3$

	

## Code

```c++
#include <cstdio>
#include <cstring>
using namespace std;
typedef long long ll;
const int N=100005;
int n,m;
int h[N],tot;
struct Edge{
	int v,f,next;
}e[N*2];
int col[N],cnt,ecnt,sum[3],type;
inline void addEdge(int u,int v,int f){
	e[++tot]=(Edge){v,f,h[u]}; h[u]=tot;
}
void readData(){
	scanf("%d%d",&n,&m);
	int u,v;
	for(int i=1;i<=m;i++){
		scanf("%d%d",&u,&v);
		addEdge(u,v,0);
		addEdge(v,u,1);
	}
}
int trans(int x,int f){
	if(f==0)
		x=(x+1)%3;
	else x=(x-1+3)%3;
	return x;
}
void dfs(int u){
	cnt++;
	for(int i=h[u],v,f;i;i=e[i].next){
		v=e[i].v; f=e[i].f;
		if(!f) ecnt++;
		if(col[v]!=-1){
			if(col[v]!=trans(col[u],f))
				type=-1;
		}
		else{
			col[v]=trans(col[u],f);
			if(type!=-1&&!sum[col[v]]) type++;
			sum[col[v]]++;
			dfs(v);
		}
	}
}
void paint(){
	ll ans=0;
	memset(col,-1,sizeof col);
	for(int i=1;i<=n;i++)
		if(col[i]==-1){
			type=1;
			sum[0]=1; sum[1]=sum[2]=0;
			col[i]=0;
			cnt=ecnt=0;
			dfs(i);
			if(type==3)
				ans+=1LL*sum[0]*sum[1]+1LL*sum[1]*sum[2]+1LL*sum[2]*sum[0];
			else if(type==-1)
				ans+=1LL*cnt*cnt;
			else ans+=ecnt;
		}
	printf("%lld\n",ans);
}
int main(){
	readData();
	paint();
	return 0;
}
```

