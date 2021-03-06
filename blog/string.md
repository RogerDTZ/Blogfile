##### Time Limit: 1000 ms   Memory Limit: 1024 MB

## Description

给你一个字符串S，你需要维护这个字符串S并支持两种操作：

1、在字符串S末尾插入一个字符。

2、记字符串T为字符串S从第 ll 个字符到第 rr 个字符所构成的子串。询问字符串T中最长的子串使得该子串在T中出现过至少两次（例如：T="ababa"，最长的子串应为aba，长度为3），并输出它的长度。如果不存在这样的子串，则输出0。

## Input

第一行为一个整数 online(0≤online≤1)online(0≤online≤1) ，在下文中用到该值。

第二行为一个字符串和一个整数m，分别表示原始的字符串（仅含小写字母）和操作个数。

记 lenlen 为字符串的长度（注意它会随时改变）。

记 lastanslastans 为上次询问的答案，初始为0。

接下来m行，每行一个操作：

若操作为"1 c"格式（c为字符），表示在字符串末尾新添一个字符。新增的字符为 (c−′a′+lastans×online) mod 26+′a′(c−′a′+lastans×online) mod 26+′a′

若操作为"2 l r"格式，表示询问。记 l′=(l−1+lastans×online) mod len+1,r′=(r−1+lastans×online) mod len+1l′=(l−1+lastans×online) mod len+1,r′=(r−1+lastans×online) mod len+1 ，真正的询问区间为 [l′,r′][l′,r′] 。我们保证 1≤l′≤r′≤len1≤l′≤r′≤len （前提是你的lastans没有错）。

## Output

对于每一个询问，输出一行表示答案。

## Sample Input

```
【样例输入1】
0
aabda 6
2 1 4
1 b
2 2 6
1 d
2 1 7
2 3 7
【样例输入2】
1
aabda 6
2 1 4
1 a
2 1 5
1 b
2 6 5
2 7 4
```

## Sample Output

```
【样例输出1】
1
2
3
2
【样例输出2】
1
2
3
2
```

## HINT

记字符串的初始长度为 nn ，操作和询问的个数之和为 qq 。

子任务1（5分）： 1≤n,q≤50,online=11≤n,q≤50,online=1

子任务2（10分）： 1≤n,q≤250,online=11≤n,q≤250,online=1

子任务3（15分）： 1≤n,q≤1500,online=01≤n,q≤1500,online=0

子任务4（10分）： 1≤n,q≤1500,online=11≤n,q≤1500,online=1

子任务5（20分）： 1≤n,q≤30000,online=01≤n,q≤30000,online=0

子任务6（20分）： 1≤n,q≤30000,online=11≤n,q≤30000,online=1

子任务7（20分）： 1≤n,q≤50000,online=1



# Solution

​	题目比较特殊，问的是出现至少两次，这样可以少考虑一点东西。

​	一个一个地加入字符，在加入到第$r$个字符的时候，回答$[l,r]$的询问。

​	用后缀自动机来维护字符串，对每一个状态，维护一个变量$right$表示该状态在字符串最后出现位置的右端点，每新加入一个字符$str_r$，设在后缀自动机上的新节点是$x$，那么在parent树上，$x$到根节点1的$right$都要更新为$r$。这个操作用LCT维护parent树实现。

​	我们发现，在更新$right$的时候，旧的$right$被赋值成了新的$r$，但其实$right$和$r$其实刚好代表了这个状态在字符串中最后出现的两个位置。记该状态的最长后缀长度是$len$（后缀自动机里面的$len$)，那么对于$r$时间点的所有询问，用$len$更新$l$在$[1,right-len+1)$的询问；用$right-l+1$更新$l$在$[right-len+1,right]$的询问，由于询问的时候$-l+1$是固定的，所以用$right$更新就好了，查询的时候$-l+1$。以上两个操作用两棵线段树记录。

​	我们发现这样更新好像忽略了很多情况，但为什么可以这样更新呢？对于同一个状态，虽然它们可能会相同地在字符串中重复很多次，但是如果我们用最右边的$right$更新，是可以覆盖到所有其他位置的更新操作的，所以用最右边的$right$来更新即可。

​	但是这样依然不够快，有可能$x$到根节点的路径比较长，这怎么办呢？

​	这里有个巧合：lct中，在同一条重链里面的状态，它们的$right$是相同的，因为这是维护时候自然体现的嘛。

我们发现可以用一条重链里面$len$最大的状态来代表这条链中所有的状态，按照上面说的方法更新，因为它一定能覆盖其他状态的更新。

​	题目要求动态加入，那么把线段树可持久化。对于询问$[l,r]$，在$r$的时间点查询就好。



## Tips

​	LCT的access操作不可调用，因为这样会破坏"同一条链的$right$相同的性质"，恰好连边和删边比较好做，是树上的，特殊写一下，具体见代码，真的调了好久......

​	对于更新操作，重写一个access操作来实现，大框架是一样的，但是注意要先断开，再更新，再连接；而不是像原版access直接换右儿子。



```c++
#include <cstdio>
#include <cstring>
using namespace std;
const int N=400005;
int online,n,_n,m,ans;
char str[N];
inline void swap(int &x,int &y){int t=x;x=y;y=t;}
inline int max(int x,int y){return x>y?x:y;}
inline int rd(){
	char c=getchar(); int x=0,f=1;
	while(c<'0'||c>'9'){if(c=='-')f=-1;c=getchar();}
	while('0'<=c&&c<='9'){x=x*10+c-'0';c=getchar();}
	return x*f;
}
inline void write(int x){
	if(x>9) write(x/10);
	putchar('0'+(x%10));
}
struct SegmentTree{/*{{{*/
	int rt[N*2*19],sz,ch[N*2*19][2],maxs[N*2*19],tm[N*2*19],tmcnt;	
	void step(){tmcnt++;}
	void insert(int u1,int &u2,int l,int r,int L,int R,int val){
		if(L>R) return;
		if(!u2) u2=copy(u1);
		else if(tm[u2]!=tmcnt) u2=copy(u1);
		if(L<=l&&r<=R){
			maxs[u2]=max(maxs[u2],val);
			return;
		}
		int mid=l+r>>1;
		if(L<=mid) insert(ch[u1][0],ch[u2][0],l,mid,L,R,val);
		if(mid<R) insert(ch[u1][1],ch[u2][1],mid+1,r,L,R,val);
	}
	int query(int u,int l,int r,int pos,int ret){
		if(!u) return ret;
		ret=max(ret,maxs[u]);
		if(l==r) return ret;
		int mid=l+r>>1;
		if(pos<=mid) return query(ch[u][0],l,mid,pos,ret);
		else return query(ch[u][1],mid+1,r,pos,ret);
	}
	inline int copy(int u){
		int v=++sz;
		maxs[v]=maxs[u];  tm[v]=tmcnt;
		ch[v][0]=ch[u][0]; ch[v][1]=ch[u][1];
		return v;
	}
}seg1,seg2;/*}}}*/
struct Lct{/*{{{*/
	int rev[N*2],ch[N*2][2],fa[N*2],val[N*2],tag[N*2],len[N*2],lenmax[N*2];
	inline bool isroot(int u){return ch[fa[u]][0]!=u&&ch[fa[u]][1]!=u;}
	inline int who(int u){return ch[fa[u]][1]==u;}
	inline void update(int u){
		lenmax[u]=max(len[u],max(lenmax[ch[u][0]],lenmax[ch[u][1]]));
	}
	inline void reverse(int u){
		rev[u]^=1;
		swap(ch[u][0],ch[u][1]);
	}
	inline void maketag(int u,int x){
		tag[u]=x; val[u]=x;
	}
	inline void rotate(int u){
		int f=fa[u],g=fa[f],c=who(u);
		if(!isroot(f)) ch[g][who(f)]=u;
		fa[u]=g;
		ch[f][c]=ch[u][c^1]; 
		if(ch[f][c]) fa[ch[f][c]]=f;
		ch[u][c^1]=f; fa[f]=u;
		update(f); update(u);
	}
	inline void pushdown(int u){
		if(rev[u]){
			if(ch[u][0]) reverse(ch[u][0]);
			if(ch[u][1]) reverse(ch[u][1]);
			rev[u]=0;
		}
		if(tag[u]){
			if(ch[u][0]) maketag(ch[u][0],tag[u]);
			if(ch[u][1]) maketag(ch[u][1],tag[u]);
			tag[u]=0;
		}
	}
	inline void pd(int u){
		if(!isroot(u)) pd(fa[u]);
		pushdown(u);
	}
	inline void splay(int u){
		pd(u);
		for(;!isroot(u);rotate(u))
			if(!isroot(fa[u]))
				rotate(who(fa[u])==who(u)?fa[u]:u);
	}
	inline void link(int a,int b){
		splay(a);
		fa[a]=b;
	}
	inline void cut(int a){
		splay(a);
		if(ch[a][0]){
			fa[ch[a][0]]=fa[a]; //!!!
			ch[a][0]=0;
			update(a);
		}
		else
			fa[a]=0;
	}
	inline int query(int u){
		pd(u);
		return val[u];
	}
	void access_special(int u,int nval){
		int v;
		for(v=0;u;v=u,u=fa[u]){
			splay(u);
			ch[u][1]=0; update(u); //!!!
			int r2=val[u],maxlen=lenmax[u];
			if(r2&&maxlen){
				seg1.insert(seg1.rt[nval-1],seg1.rt[nval],1,_n+m,1,r2-maxlen,maxlen);
				seg2.insert(seg2.rt[nval-1],seg2.rt[nval],1,_n+m,r2-maxlen+1,r2,r2);
			}
			ch[u][1]=v;
			update(u);
		}
		maketag(v,nval);
	}
}lct;/*}}}*/
struct Sam{/*{{{*/
	int last,sz,len[N*2],trans[N*2][26],link[N*2];
	void init(){
		last=sz=1;
		len[1]=link[1]=0;
	}
	void expand(int c){
		int u=++sz,p=last;
		len[u]=len[p]+1; lct.len[u]=len[u];
		for(;p&&!trans[p][c];p=link[p]) trans[p][c]=u;
		if(!p) link[u]=1,lct.link(u,1);
		else{
			int q=trans[p][c];
			if(len[q]==len[p]+1) link[u]=q,lct.link(u,q);
			else{
				int v=++sz;
				len[v]=len[p]+1; lct.len[v]=len[v]; lct.val[v]=lct.query(q);
				for(int i=0;i<26;i++) trans[v][i]=trans[q][i];
				lct.cut(q); 
				lct.link(q,v); lct.link(u,v);
				lct.link(v,link[q]);
				link[v]=link[q]; 
				link[q]=link[u]=v;
				for(;trans[p][c]==q;p=link[p]) trans[p][c]=v;
			}
		}
		last=u;
	}
}sam;/*}}}*/
inline void addchar(int c){
	n++;
	sam.expand(c);
	seg1.rt[n]=seg1.rt[n-1];
	seg2.rt[n]=seg2.rt[n-1];
	seg1.step(); seg2.step();
	lct.access_special(sam.last,n);
}
inline int query(int l,int r){
	int ret=0;
	ret=max(ret,seg1.query(seg1.rt[r],1,_n+m,l,0));
	ret=max(ret,seg2.query(seg2.rt[r],1,_n+m,l,0)-l+1);
	return ret;
}
int main(){
	freopen("input.in","r",stdin);
	online=rd();
	scanf("%s",str+1); n=strlen(str+1);
	m=rd();
	_n=n; n=0;
	sam.init();
	for(int i=1;i<=_n;i++) 
		addchar(str[i]-'a');
	int opt,x,y;
	char inp[2];
	for(int i=1;i<=m;i++){
		opt=rd();
		if(opt==1){
			scanf("%s",inp);
			inp[0]=(inp[0]-'a'+ans*online)%26+'a';
			addchar(inp[0]-'a');
		}
		else{
			x=(rd()-1+ans*online)%n+1;
			y=(rd()-1+ans*online)%n+1;
			write(ans=query(x,y)); putchar('\n');
		}
	}
	return 0;
}
```



