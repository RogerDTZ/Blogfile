## Description

​	IT = Inverse Transform

​	两个长度为 $2n$ 的序列 $a,b$.  ( 下标从 $0$ 到 $2n−1$ )

​	满足$ 0≤a_i≤10^9$ , 且  由 $a$ 变换而来, 变换如下:

​	$b_i=∑\limits_{0≤j<2^n}f((i∨j)⊕i) a_j$

​	其中 $V$ 表示按位或, $⊕$ 表示按位异或. f(x)=[x的二进制中1的个数为偶数]f(x)=[x的二进制中1的个数为偶数]

​	(其中 [expression] 当 $expression$ 为真时为 1, 否则为 0)

​	现在, $a$ 序列被玩丢了... 

​	请你通过 $b$ 序列还原出 $a$ 序列

## Input

​	第一行一个数 $T$ 表示测试数据组数
​	接下来对于每组数据 : 
​	第一行一个整数 $n$
​	第二行 $2n$ 个整数, 表示序列 $b$

## Output

​	输出包括 $T$ 行
​	每行 $n$ 个整数, 表示序列 $a$

## Sample Input


​	4
​	0
​	1
​	1
​	1 3
​	2
​	5 3 4 10
​	3
​	101 91 92 104 93 105 108 190


## Sample Output


​	1
​	1 2
​	1 2 3 4
​	31 24 26 23 27 23 24 12


## HINT

​	对于 10% 的数据 n≤5
​	对于另外 10% 的数据 n≤10
​	对于 100% 的数据 : 
​	T≤5
​	1≤n≤20
​	数据保证 b 序列 能 唯一地对应一个 a 序列
​	且保证对应的 a 序列满足 $0≤a_i≤10^9$

​	( **注意** : 但 **不保证** 读入的 $b$ 序列满足 $0≤bi≤10^9$ )





# Solution

​	不得不说是道很好的题啊.

​	题目中的$f((i∨j)⊕i)$，当且仅当$i=0$且$j=1$时为1，其余情况皆为0.

​	正解用的是分治的方法，再向上合并，有点类似各大快速变换的思想.

​	首先考虑若知$a$，如何$O(n log n)$求$b$.

​	由于总长度是$2^n$，所以给分治带来了极大的便利。每次对$[l,r]$分治为$[l,mid]$与$(mid,r]$来解决。自底向上层层计算，对于分治区间$[l,r]$,  当下$b_i$的值表示：
$$
b_i=\sum_{j=l}^r f((i∨j)⊕i)*a_j \quad\quad\quad i\in[l,r]
$$


​	那么分治完成后，$b_i$的值才是最终答案。

​	考虑最底层的情况$[l,l]$，此时$b_l=f((l∨l)⊕l)*a_l=a_l$.

​	假设现在正在在分治$[l,r]$，记分治长度$len=(r-l+1)$，已经计算出$[l,mid]$和$(mid,r]$ 的$b$值，现在考虑如何将$b$合并。

​	分治区间的长度是2的整数次幂，这给了一个很棒的性质，记$k=log_2len$，若只看下标的后$k$位，我们会发现，对于左右区间，左边的最高位全是0，右边的最高位全是1。如果忽略最高位，对于$\forall i\in[l,mid]$都有$i==i+(r-l+1)/2$.  而随着分治的逐渐上推，$k$是不断加一加一的。

​	这意味着如果我们要求新的$b'_i$，我们只需要考虑当前$i$的最高位（第$k$位）带来的影响。

​	记$g_k(i,j)​$表示只考虑$i​$和$j​$的后$k​$位时，$f((i∨j)⊕i)​$的值。

​	首先看$i\in(mid,r]$, 它们的第$k$位都是1. 由于$(1,0)$和$(1,1)$计算出来都是$0$，所以我们可以直接令$b'_i=b_{i-len/2}+b_i$

​	为什么？因为$b_{i-len/2}$对应着满足$g_{k-1}(i-len/2,j)=1$的$a_j$的总和，而这些$j$同样满足$g_k(i,j)$，所以这些$a_j$可以加进来。

$b_i$对应着满足$g_{k-1}(i,j)=1$的$a_j$的总和，而这些$j$同样满足$g_k(i,j)$，所以这些$a_j$也可以加进来。

​	接着再看$i\in[l,mid]$，它们的第$k$位都是$0$. 我们令$b'_i=b_i+(\sum_{j=mid+1}^ra_j)   -b_{i+len/2}$。$b_i$的部分同上很好理解，因为$(0,0)$计算出来是$0$，满足$g_{k-1}(i,j)=0$的$j$同样满足$g_k{i,j}$，这些$a_j$可以加进来；另一部分比较特殊，因为$(0,1)$计算出来是1，满足$g_{k-1}(i+len/2,j)=1$的$j$必定不满足$g_k(i,j)=1$，因为计算出来的1会使1的总数变为奇数，那么我们现在反而要变向使用“不合法的情况”，即满足$g_{k-1}(i+len/2,j)=0$的情况，容易表达为$(\sum_{j=mid+1}^ra_j)   -b_{i+len/2}$。

​	总结一下，我们有：
$$
\begin{aligned}
b'_i&=b_i+(\sum_{j=mid+1}^ra_j)-b_{i+len/2} &i\in[l,mid]\\
b'_i&=b_{i-len/2}+b_i  &i\in(mid,r]
\end{aligned}
$$
​	就可以自底向上$O(nlogn)​$上推得到$b​$了。

​	下面看如何知$a$反推$b$，把上述两个式子一加再化简就可以得到：
$$
\begin{aligned}
b_i&=\frac{b'_i+b'_{i+len/2}+(\sum_{j=mid+1}^ra_j)}2 &i\in[l,mid]\\
b_i&=b'_i-b_{i-len/2} &i\in(mid+1,r]
\end{aligned}
$$
​	观察得到$b'_r=\sum_{j=l}^r a_j$，且$b'_{mid}=\sum_{j=l}^{mid}a_j$，所以$\sum_{j=mid+1}^r a_j=b'_r-b'_{mid}$

​	然后就神奇的反推下去了。代码奇短无比又简单。

```c++
#include <cstdio>
using namespace std;
const int N=1100000;
typedef long long ll;
namespace IO{/*{{{*/
	const int SIZE=50000000;
	char buffer[SIZE];
	int pos;
	void init(){fread(buffer,1,SIZE,stdin);}
	char getch(){return buffer[pos++];}
	ll getInt(){
		char c=getch(); ll x=0,f=1;
		while(c<'0'||c>'9'){if(c=='-')f=-f;c=getch();}
		while('0'<=c&&c<='9'){x=x*10+c-'0';c=getch();}
		return x*f;
	}
}/*}}}*/
int bn,n;
ll a[N];
void solve(){
	for(int i=n;i>1;i>>=1){
		for(int j=0;j<n;j+=i){
			ll s=a[j+i-1]-a[j+i/2-1];
			for(int k=0;k<i/2;k++){
				a[j+k]=(a[j+k]+a[j+i/2+k]-s)/2;
				a[j+i/2+k]-=a[j+k];				
			}
		}
	}
}
void Main(){
	bn=IO::getInt();
	n=1<<bn;
	for(int i=0;i<n;i++) a[i]=IO::getInt();
	solve();	
	for(int i=0;i<n;i++) printf("%lld ",a[i]);
	puts("");
}
int main(){
	freopen("input.in","r",stdin);
	IO::init();
	int T=IO::getInt();
	while(T--) Main();
	return 0;
}
```



