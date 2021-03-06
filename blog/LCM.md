## Description

​	题目描述

　　给你$n,k$，要你选一些互不相同的正整数，满足这些数的lcm为$n$，且这些数的和为$k$的倍数。

　　求选择的方案数。对$232792561$取模。

　　$n\le10^{18},k\le20，n的全部质因子都\le100​$



## Solution

​	$n\le10^{18}$时其质因数个数最多只有15个。记$n$的质因子个数为$m$。	

​	设$g_{i,j}$表示有多少个$n$的因数$d$，满足$d$中$i$状态的质因子的指数与$n$相同，且$d\%k=j$。

​	再设$f_{i,j}$，表示从$g_{i}$所涉及的数组成的数集$A_i$中有多少个子集，使得子集中的数之和模$k$为$j$。

​	这点可以用生成函数解决：
$$
G_i(x)=\prod_{a\in A_i}(1+x^a)
$$
​	则$f_{i,j}=G_i(x)[j]$。

​	考虑许多个$f_{i,j}$怎样组合才对答案有贡献。当下列条件同时满足时：

​	（1）$i_1|i_2|...|i_s=2^m-1$	

​	（2）$i_1\neq i_2\neq...\neq i_s$	

​	（3）$j_1+j_2+...+j_s\equiv0\pmod k$

​	则会对$Ans$有$\prod f_{i,j}$的贡献。

​	其实我们要求一个数组$h_{i,j}$表示任选所有数字，使得$i$状态表示的每个质因子$p$，至少存在一个数满足其$p$的指数与$n$中相同，且所有数加起来模$k$等于$j$的方案数。显然答案是$h_{2^m-1,0}$。

​	对每一行$f_i$单独做一次DFT，此时得到每行$f_i$的点值表达。我们要做的，是**选择那些编号或起来恰好等于$2^m-1$的行，将它们的点值表达对位相乘，累加到一个新数组$tmp$**，这相当于枚举哪些行参与贡献、枚举这些行的哪一个$f_{i,j}$参与贡献。对此$tmp$做逆DFT，就可以得到$h_{i}$了！

​	所以这部分如何操作？我们发现每一列的操作模式是相同的且互不影响。那么现在我们只看第一列即$f_{i,0}$，将这一列数拉出来方便看，设为$a_i$，其中$a_i=DFT(f_i)[0]$。

​	另起一个命名空间：$f_i$表示从$a$中选择若干个数使得编号或起来等于$i$时所有选择数的乘积，的所有情况之和。

​	构造莫比乌斯变换$F_S=\sum_{T\in S}{f_T}=\prod_{i\in S}(1+a_i)$。最后一个式子只要将括号展开，会得到许多乘积之和，每一个乘积都属于某一个$f$，因此是成立的。

​	根据莫比乌斯反演，有$f_S=\sum_{T\in S}(-1)^{|S|-|T|}F_T$，因此若我们知道$F$，就可以暴力地求$f_{2^m-1}$。那么$f_{2^m-1}$其实就是$tmp$的第0项，因为它是一种或的组合形式，满足粗体字，解决了一列的操作，那么其他列同理。

​	$F$怎么求呢？这里要用到快速莫比乌斯变换FMT，即求解$F_S=\prod_{T\in S}g(T)$，其中$g$是一个已知函数。

​	初始时$F_S=g(S)$。考虑补全$F_S$的内容，即考虑自己$F_i$如何去补全别人的$F_j$。有一种方式可以**无重复**地做到这点。$F_i$能更新$F_j$当且仅当$j$是由$i$的某一位0改成1得到。从低到高枚举改第$x$位，之后从小到大循环$i$，如果$i$的第$x$位是0，那么更新$F_{i+2^x}+=F_i$。为什么要这样做？因为这样每个$F_i$所包含的元素都会只统计一次。假设$F_i$中$g(j)$被统计了多次，那就说明$g(j)$贡献去$F_i$的路径出现分叉和合并，即有人先改了高位、有人又先改了低位，两个过程又是同时进行的，这与我们算法设计的“逐位修改”有矛盾。因此可以这么做。

​	总时间复杂度$\mathcal O(k^22^m+km2^m)$。

​	交题不小心把yww的std交上去了，常数太优秀，我的代码覆盖不掉他。qwq



## Code

```c++
#include <cstdio>
#include <cstring>
using namespace std;
typedef long long ll;
const int PN=26,MOD=232792561,G=71;
ll n;
int m;
int d[PN],id[PN],idcnt,w[410][2];
int p[PN]={0,2,3,5,7,11,13,17,19,23,29,31,37,41,43,47,53,59,61,67,71,73,79,83,89,97};
int f[1<<15][20];
inline int bit(int i){return 1<<(i-1);}
int fmi(int x,int y,int mod=MOD){
	int res=1;
	for(;y;x=1LL*x*x%mod,y>>=1)
		if(y&1) res=1LL*res*x%mod;
	return res;
}
void init_rt(){
	int rt=fmi(G,(MOD-1)/m);
	w[0][0]=1;
	for(int i=1;i<=400;i++) w[i][0]=1LL*w[i-1][0]*rt%MOD;
	for(int i=0;i<=400;i++) w[i][1]=fmi(w[i][0],MOD-2);
}
void dft(int n,int *a,int t){
	static int b[30];
	for(int i=0;i<n;i++) b[i]=0;
	for(int i=0;i<n;i++)
		for(int j=0;j<n;j++)
			(b[i]+=1LL*a[j]*w[i*j][t]%MOD)%=MOD;
	if(t){
		int invn=fmi(n,MOD-2);
		for(int i=0;i<n;i++) b[i]=1LL*b[i]*invn%MOD;
	}
	for(int i=0;i<n;i++) a[i]=b[i];
}
void depart(){
	ll x=n;	
	for(int i=1;i<PN;i++){
		d[i]=0;
		while(x%p[i]==0) x/=p[i],d[i]++;
		if(d[i]) id[i]=++idcnt;
	}
}
void add(int x,int y){
	static int t[30];
	for(int i=0;i<m;i++) t[i]=f[x][i];
	for(int i=0;i<m;i++)
		(f[x][(i+y)%m]+=t[i])%=MOD;
}
void dfsdiv(int x,int state,int mulmodk){
	if(x==PN){
		add(state,mulmodk);
		return;
	}
	for(int i=0;i<=d[x];i++){
		if(i&&(i==d[x]))
			dfsdiv(x+1,state|bit(id[x]),1LL*mulmodk*fmi(p[x],i,m)%m);
		else
			dfsdiv(x+1,state,1LL*mulmodk*fmi(p[x],i,m)%m);
	}
}
void reset(){
	memset(f,0,sizeof f);
}
void Main(){
	scanf("%lld%d",&n,&m);
	init_rt();
	idcnt=0;
	depart();	
	int all=1<<idcnt;
	for(int i=0;i<all;i++){
		f[i][0]=1;
		for(int j=1;j<m;j++) f[i][j]=0;
	}
	dfsdiv(1,0,1);
	for(int i=0;i<all;i++) dft(m,f[i],0);
	for(int i=1;i<all;i<<=1)
		for(int j=0;j<all;j++)
			if(!(i&j))
				for(int k=0;k<m;k++)
					f[j+i][k]=1LL*f[j+i][k]*f[j][k]%MOD;
	for(int i=0;i<all-1;i++){
		int a=1;
		for(int j=0;j<idcnt;j++)
			if(!((i>>j)&1)) a=-a;
		for(int j=0;j<m;j++)
			(f[all-1][j]+=a*f[i][j])%=MOD;
	}
	dft(m,f[all-1],1);
	int ans=f[all-1][0];
	printf("%d\n",ans<0?ans+MOD:ans);
}
int main(){
	int T;
	scanf("%d",&T);
	while(T--) Main();
	return 0;
}
```

