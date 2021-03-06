## Description

​	数学老师走啦，英语老师来上课啦
​	他的性格与众不同，又因为大家都是理科班的学生
​	他希望大家在数字母的过程中领悟英语的快乐
​	他用m种字母进行排列组合，
​	得到了所有不同的，长度为n的字符串
​	（不需要所有字母都出现在字符串中）
​	对于每个字符串s
​	定义C(s)为s中出现次数最多的字母的出现次数
​	那么问题来了
​	所有的这些字符集大小为m，长度为n的字符串中
​	C(s)=k的有多少个呢

## Input

​	一行三个整数n,m,k，分别表示长度，字符集和要求的C(s)

## Output

​	输出一行表示结果
​	答案对998244353取模

## Sample Input

​	3 2 2

## Sample Output

​	6

## HINT

​	数据保证k≤n
​	对于10%的数据，1≤n,m≤8
​	对于30%的数据，1≤n,m≤200
​	对于50%的数据，1≤n,m≤1000
​	对于100%的数据，1≤n,m≤50000

​	样例解释：

​	假设样例中的两个字母为a,b
​	则满足条件的有aab,aba,abb,baa,bab,bba六个



# Solution

​	首先把最直观的DP方程列出来。

​	记$f[i][j][k]$为当前考虑到第$i$个字母，已经使用了串中的$j$个位置，出现最多的字母次数不超过$k$的方案数。答案就是$f[m][n][k]-f[m][n][k-1]$。

​	转移方程显然是枚举当前字母使用多少次：
$$
f[i][j][k]=\sum_{x=0}^k {j\choose x}f[i-1][j-x][k]
$$
​	然后可以发现$k$十分的冗余，并没有参与转移。也就是说$k$仅仅作用于循环范围控制上。

​	我们尝试把最后一维省掉：$f[i][j]$。$k$仍然发挥作用，也就是现在的$f[i][j]$对应着原来的$f[i][j][k]$。

​	现在看看方程：
$$
\begin{aligned}
f[i][j]&=\sum_{x=0}^k{j\choose x}f[i-1][j-x]\\
&=\sum_{x=0}^k\frac{j!}{x!(j-x)!}f[i-1][j-x]\\
\frac{f[i][j]}{j!}&=\sum_{x=0}^k\;x!\;\frac{f[i-1][j-x]}{(j-x)!}
\end{aligned}
$$
​	后面显然是一个卷积的形式，并且**等号左边的形式和卷积右半边的形式一样**。所以可以把每个$f[i]$看做一个多项式
$$
f[i]=\frac{f[i][0]}{0!}+\frac{f[i][1]}{1!}x+\frac{f[i][2]}{2!}x^2+...+\frac{f[i][n]}{n!}x^n
$$


​	转移就是这个多项式和
$$
T(x)=\frac1{0!}+\frac1{1!}x+\frac1{2!}x^2...+\frac1{k!}x^k
$$
​	的卷积。即$f[n]=f[0]*T^{n}(x)$

​	而$T(x)$是独立的存在不受其他东西影响，所以将$T(x)$用快速幂自卷积一下，再用$f[0]$卷积一下就好了。根据定义，$f[0]=1$，所以相当于直接求$T(x)$的$n$次方。答案别忘了乘上$n$的阶乘。

```c++
#include <cstdio>
#include <cstring>
using namespace std;
const int N=50005,MOD=998244353,G=3,B17=131100;
int fact[N],iact[N];
inline void swap(int &x,int &y){x^=y^=x^=y;}
inline int pow(int x,int y){
	int res=1;
	for(;y;x=1LL*x*x%MOD,y>>=1)
		if(y&1) res=1LL*res*x%MOD;
	return res;
}
namespace NTT{/*{{{*/
	int n,invn,bit,rev[B17],W[B17][2];
	void build(){
		int b=pow(G,MOD-2);
		for(int i=0;i<=17;i++){
			W[1<<i][0]=pow(G,(MOD-1)/(1<<i));
			W[1<<i][1]=pow(b,(MOD-1)/(1<<i));
		}
	}
	void init(int _n){
		for(n=1,bit=0;n<_n;n<<=1,bit++);
		invn=pow(n,MOD-2);
		for(int i=0;i<n;i++) rev[i]=(rev[i>>1]>>1)|((i&1)<<(bit-1));
	}
	void clear(int *a){for(int i=0;i<n;i++)a[i]=0;}
	void ntt(int *a,int f){
		for(int i=0;i<n;i++) if(i<rev[i]) swap(a[i],a[rev[i]]);
		int u,v,w_n,w;
		for(int i=2;i<=n;i<<=1){
			w_n=W[i][f==-1];
			for(int j=0;j<n;j+=i){
				w=1;
				for(int k=0;k<i/2;k++){
					u=a[j+k]; v=1LL*w*a[j+i/2+k]%MOD;
					a[j+k]=(u+v)%MOD; a[j+i/2+k]=(u-v)%MOD;
					w=1LL*w*w_n%MOD;
				}
			}
		}
		if(f==-1)
			for(int i=0;i<n;i++) a[i]=1LL*a[i]*invn%MOD;
	}
}/*}}}*/
void ksm(int *x,int y,int n,int *res){
	NTT::init((n+1)*2);
	NTT::clear(res);
	res[0]=1;
	for(;y;y>>=1){
		NTT::ntt(x,1);
		if(y&1){
			NTT::ntt(res,1);
			for(int i=0;i<NTT::n;i++) res[i]=1LL*res[i]*x[i]%MOD;
			NTT::ntt(res,-1);
			for(int i=n+1;i<NTT::n;i++) res[i]=0;
		}
		for(int i=0;i<NTT::n;i++) x[i]=1LL*x[i]*x[i]%MOD;
		NTT::ntt(x,-1);
		for(int i=n+1;i<NTT::n;i++) x[i]=0;
	}
}
int solve(int n,int m,int k){
	static int a[B17],b[B17];
	memset(a,0,sizeof a);
	for(int i=0;i<=k;i++) a[i]=iact[i];
	ksm(a,m,n,b);
	return 1LL*fact[n]*b[n]%MOD;
}
int main(){
	freopen("input.in","r",stdin);
	NTT::build();
	int n,m,k;
	scanf("%d%d%d",&n,&m,&k);
	fact[0]=1;
	for(int i=1;i<=n;i++) fact[i]=1LL*fact[i-1]*i%MOD;
	iact[n]=pow(fact[n],MOD-2);
	for(int i=n-1;i>=0;i--) iact[i]=1LL*iact[i+1]*(i+1)%MOD;
	int ans=(solve(n,m,k)-solve(n,m,k-1))%MOD;
	printf("%d\n",ans<0?ans+MOD:ans);
	return 0;
}
```



​	



​	