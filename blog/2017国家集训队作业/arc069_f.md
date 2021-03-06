## Description

​	数轴上有 $n$个旗子，第$i$个可以插在坐标$x_i$或者$y_i$。	

​	请最大化两两旗子之间的最小距离。 

​	$2 \le n \le 10^4$，$1 \le x_i,y_i \le 10^9$





## Solution

​	这种有若干组决策，每种决策二选一，然后要求所有决策互相合法的问题，习惯性的要往2-sat想一想。

​	我们把每个旗子$i$插在$x_i$还是$y_i$看做一种决策，分别用两个点表示。

​	看起来二分答案比较靠谱。

​	问题转变为：判定是否存在一种方案，使得两两旗子之间的最小距离不小于$len$。

​	有了$x$，我们就很方便地进行逻辑连边了。

​	考虑每个旗子$i$的两个位置。如果该旗子选择了$x_i$，则对于所有处于$(x_i-len,x_i+len)$的位置（**除了$x_i$**），它们对应的旗子在决策时必须选择另一个可选位置，它们必须选择另外一个可选位置$p$。因此，从$x$向所有的$p$连一条边。

​	如果该旗子选择了$y_i$同理。

​	之后进行2-sat，即用Tarjan缩点。如果一个旗子的两个位置处于同一个强联通分量，则无解。否则一定可以构造出一种方案，由于这是一个判定问题，我们不需要真正构造。

​	由于点数较大无法$\mathcal O(n^2)$连边，我们使用线段树进行连边优化。

​	总时间复杂度为$\mathcal O(n \log_2 n \log_2 1e9)$

## Code

​	

```c++
#include <cstdio>
#include <vector>
#include <algorithm>
#include <unordered_map>
#define pb push_back
using namespace std;
const int N=10005,NUM=N*6;
int n,p[N][2];
int dcnt,d[N*2];
unordered_map<int,int> tid;
int lis[N*2];
int nn;
int dfntm,dfn[NUM],low[NUM],sta[NUM],top,bl[NUM],blcnt;
bool ins[NUM];
int h[NUM],tot;
struct Edge{int v,next;}e[N*100];
inline void addEdge(int u,int v){
	//printf("%d %d\n",u,v);
	e[++tot]=(Edge){v,h[u]}; h[u]=tot;
}
void readData(){
	scanf("%d",&n);
	for(int i=1;i<=n;i++)
		scanf("%d%d",&p[i][0],&p[i][1]);
}
void Diz(){
	for(int i=1;i<=n;i++) 
		d[++dcnt]=p[i][0], d[++dcnt]=p[i][1];
	sort(d+1,d+1+dcnt);
	for(int i=1;i<=dcnt;i++)
		if(d[i]!=d[i-1]) tid[d[i]]=i;
	for(int i=1;i<=n;i++){
		int x=p[i][0];
		p[i][0]=tid[p[i][0]];
		tid[x]++;
		x=p[i][1];
		p[i][1]=tid[p[i][1]];
		tid[x]++;
		lis[p[i][0]]=n+i;
		lis[p[i][1]]=i;
	}
}
namespace SEG{
	const int S=N*4;
	int rt,sz;
	int ch[S][2];	
	void reset(){
		rt=sz=0;
	}
	void build(int &u,int l,int r){
		if(l==r){
			u=lis[l];
			return;
		}
		u=++sz;
		int mid=(l+r)>>1;
		build(ch[u][0],l,mid);
		build(ch[u][1],mid+1,r);
		addEdge(u,ch[u][0]);
		addEdge(u,ch[u][1]);
	}
	void link(int u,int l,int r,int L,int R,int v){
		if(L<=l&&r<=R){
			addEdge(v,u);
			return;
		}
		int mid=(l+r)>>1;
		if(R<=mid) link(ch[u][0],l,mid,L,R,v);
		else if(mid<L) link(ch[u][1],mid+1,r,L,R,v);
		else{
			link(ch[u][0],l,mid,L,mid,v);
			link(ch[u][1],mid+1,r,mid+1,R,v);
		}
	}
}
void reset(){//!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
	tot=0;
	dfntm=blcnt=0; top=0;
	for(int i=1;i<=nn;i++){
		h[i]=0;
		dfn[i]=low[i]=bl[i]=0;
	}
	SEG::reset();
}
void tarjan(int u){/*{{{*/
	dfn[u]=low[u]=++dfntm;
	sta[++top]=u;
	ins[u]=true;
	for(int i=h[u],v;i;i=e[i].next){
		v=e[i].v;
		if(!dfn[v]){
			tarjan(v);
			low[u]=min(low[u],low[v]);
		}
		else if(ins[v])
			low[u]=min(low[u],dfn[v]);
	}
	if(dfn[u]==low[u]){
		blcnt++;
		int x;
		do{
			x=sta[top];
			sta[top--]=0;
			ins[x]=false;
			bl[x]=blcnt;
		}while(x!=u);
	}
}/*}}}*/
bool judge(int x){
	reset();
	SEG::sz=n*2;
	SEG::build(SEG::rt,1,dcnt);
	for(int i=1;i<=n;i++){
		int l=lower_bound(d+1,d+1+dcnt,d[p[i][0]]-x+1)-d;
		int r=upper_bound(d+1,d+1+dcnt,d[p[i][0]]+x-1)-d-1;
		if(l<=r){
			if(l<p[i][0]) 
				SEG::link(SEG::rt,1,dcnt,l,p[i][0]-1,i);
			if(p[i][0]<r)
				SEG::link(SEG::rt,1,dcnt,p[i][0]+1,r,i);
		}
		l=lower_bound(d+1,d+1+dcnt,d[p[i][1]]-x+1)-d;
		r=upper_bound(d+1,d+1+dcnt,d[p[i][1]]+x-1)-d-1;
		if(l<=r){
			if(l<p[i][1]) 
				SEG::link(SEG::rt,1,dcnt,l,p[i][1]-1,n+i);
			if(p[i][1]<r)
				SEG::link(SEG::rt,1,dcnt,p[i][1]+1,r,n+i);
		}
	}
	nn=SEG::sz;
	for(int i=1;i<=nn;i++)
		if(!dfn[i])
			tarjan(i);
	for(int i=1;i<=n;i++)
		if(bl[i]==bl[n+i]) return false;
	return true;
}
void solve(){
	int l=0,r=1e9,mid;
	while(l<=r){
		mid=(l+r)>>1;
		if(judge(mid)) 
			l=mid+1;
		else r=mid-1;
	}
	printf("%d\n",r);	
}
int main(){
	readData();
	Diz();
	solve();
	return 0;
}
```

