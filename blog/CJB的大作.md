## Description

给你一个长度不超过100的字符串。一共进行$N$次操作，第$i$次操作是将当前字符串复制一份接到后面，并将新的一份循环移位$k_i$（$1 \le k_i \le 100$）次。给出$M$（$1 \le M \le 100000$）个询问，每次询问所有操作完成之后，$[l,r]$中，某个字母出现了多少次。其中$1 \le l \le r \le 10^{18}$。



## Solution

	我开场就写了2个小时的暴力分治，并惊喜地发现单个询问可以秒出。结果一组询问需要0.01s，拿到了40分的好成绩。

	首先$N$只有约前60项是有意义的，因此总操作数大约是60次。

	这种字符串变换题目要重点观察变换前和变换后是否有一大块一大块是相等的。比如某次操作前，字符串是$S$，$A$和$B$是$S$的两块（即$S=AB$）其中B的长度恰好是这次操作的偏移值。那么操作完之后，字符串的变化如下：
$$
AB\rightarrow AB\;BA
$$
	既然只有60次操作，而每次操作的$B$的长度不超过100，我们完全可以算出每次操作的$B$串。即枚举每个字符，反向模拟这个字符是怎么从原串变换来的，即可在一个$\log$的时间确定每个字符是什么。

	将询问查分，则我们要求$[1,n]$中有多少个$c$字符。

	考虑从最后一次操作开始递归向前计算（其实应该是：最早的一次操作，满足操作后串长不小于$n$)。

	如果$n$落在左边的$AB$内：那么直接递归前一次操作，返回其答案。

	如果$n$落在右边的$B$内：左半边$AB$的答案显然，就是原串中这个字符的数量乘上左半边有多少个原串。对于右边部分，我们预处理出每次操作中$B$的字母前缀和，$O(1)$可询问$B$前缀字母数量。两者加起来就是答案。

	如果$n$落在右边的$A$内：$ABB$的答案可以用上一个情况的思路直接算，而最右边的$A$的答案，其实就是最左边的$A$的对应位置的答案。和第一种情况一样，我们递归前一次操作即可知道这部分的答案。

	我们发现总分治递归层数不会超过有效操作次数。因此这题可以在$\mathcal O(M\log 10^{18})$内解决。

		

## Code

```c++
#include <cstdio>
#include <cstring>
using namespace std;
typedef long long ll;
namespace IO{
	const int S=20000005;
	char buffer[S];
	int pos;
	void Load(){
		pos=0;
		fread(buffer,1,S,stdin);
	}
	char getChar(){
		char res=buffer[pos++];
		if(pos>=S)
			Load();
		return res;
	}
	ll getLong(){
		ll x=0,f=1;
		char c=getChar();
		while(c<'0'||c>'9'){if(c=='-')f=-1;c=getChar();}
		while('0'<=c&&c<='9'){x=x*10+c-'0';c=getChar();}
		return x*f;
	}
	int getStr(char *str){
		int len=0;
		for(char c=getChar();'a'<=c&&c<='z';c=getChar())
			str[len++]=c;
		return len;
	}
}
using IO::getLong;
using IO::getStr;
const int N=110,B=70,C=26;
const ll UP=2e18;
char str[N];
int slen;
int n,m,off[100005];
int pre[N][C],f[B][N][C];
void readData(){
	slen=IO::getStr(str+1);
	n=getLong(); m=getLong();
	for(int i=1;i<=n;i++)
		off[i]=getLong();
}
inline ll get_len(int bit){
	return 1ll*slen*(1ll<<bit);
}
char get_char(int bit,ll x){
	if(bit==0)
		return str[x];
	ll mid=get_len(bit-1);
	if(x<=mid)
		return get_char(bit-1,x);
	ll cut=mid+off[bit];
	if(x<=cut)
		return get_char(bit-1,mid-(cut-x));
	else
		return get_char(bit-1,x-cut);
}
void precount(){
	for(int i=1;i<=slen;i++){
		for(int j=0;j<26;j++)
			pre[i][j]=pre[i-1][j];
		pre[i][str[i]-'a']++;
	}
	for(int i=1;get_len(i)<UP;i++){
		ll mid=get_len(i-1);
		off[i]%=mid;
		for(int j=1;j<=off[i];j++){
			for(int k=0;k<26;k++)
				f[i][j][k]+=f[i][j-1][k];
			char c=get_char(i-1,mid-off[i]+j);
			f[i][j][c-'a']++;
		}
	}
}
ll solve(int bit,ll n,int c){
	if(bit==0)
		return pre[n][c];
	ll mid=get_len(bit-1),cut=mid+off[bit];
	if(n<=mid) 
		return solve(bit-1,n,c);
	if(n<=cut)
		return 1ll*pre[slen][c]*(1ll<<(bit-1))+f[bit][n-mid][c];
	else
		return 1ll*pre[slen][c]*(1ll<<(bit-1))+f[bit][cut-mid][c]+solve(bit-1,n-cut,c);
}
ll calc(ll n,char c){
	if(!n) 
		return 0;
	int bit;
	for(bit=0;get_len(bit)<n;bit++);
	return solve(bit,n,c-'a');
}
void answerQuery(){
	ll l,r;
	char c[2];
	for(int i=1;i<=m;i++){
		l=getLong(); r=getLong(); getStr(c);
		printf("%lld\n",calc(r,c[0])-calc(l-1,c[0]));
	}
}
int main(){
	IO::Load();
	readData();
	precount();
	answerQuery();
	return 0;
}
```

