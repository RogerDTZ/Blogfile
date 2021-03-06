## Description

​	如果某个串可以由两个一样的串前后连接得到，我们就称之为“偶串”。比如说“xyzxyz”和“aaaaaa”是偶串，而“ababab”和“xyzxy”则不是偶串。 

​	对于一个非空串$S​$，我们定义$f(S)​$是在$S​$后面添加一些字符得到的最短偶串。比如$f(​$'abaaba'$)=​$'abaababaab'。容易证明，对于一个非空串$S​$，$f(S)​$是唯一的 。

​	现在给定一个由小写英文字母构成的偶串$S$，你需要求出$f^{10^{100}}(S)$，并统计计算结果的第$l$个字符到第$r$个字符中，每个字母出现了多少次 。

​	其中，$f^{10^{100}}(S)$是指$f(f(f(...f(S)...)))$，式子中共有$10^{100}$个$f$ 。



## Solution

​	打表找规律。从整个串来寻找规律看不出什么，反而是分别考虑两半的变化比较有效。

​	我们称$T$为$S$的"Border"，当且仅当$S$作为$T$的循环串的前缀出现，且$T$长度最小。

​	现有输入串$SS$，令$S$的"Border"为$T$，则会发现：
$$
SS\rightarrow (ST)(ST)\rightarrow (STS)(STS)\rightarrow(STSST)(STSST)...
$$
​	由于操作次数过多，只考虑前半部分的变化，就能回答所有询问：
$$
S\rightarrow ST\rightarrow STS\rightarrow STSST\rightarrow STSSTSTS
$$
​	我们发现，从第三个串开始，每一个串$i$，都是由串$i-1$+串$i-2$得来。姑且称它们为$\text{fib}$串。

​	特别地：若$|S|-|T|$ 整除$|S|$，则变化为
$$
S\rightarrow ST\rightarrow STT\rightarrow STTT...
$$
​	特判掉这种情况。

​	记$len_i$为第$i$个串的长度，$g_{i,j}$为第$i$个串中$j$字符出现了多少次。由于$\text{fib}$在第85项左右就已经超出了$1e18$，因此我们可以暴力计算出这两个数组。

​	对答案差分，假设要求$1...r$中，每个字符的出现次数。

​	有结论是，只要有$len_{x_1}+len_{x_2}+...+len_{x_k}=r$，且$len_{x_1}>len_{x_2}>...>len_{x_k}$，则$1...r$这个字符串就可以用$x_1,x_2,...,x_k$这$k$个$\text{fib}$串首尾相接得到。

​	所以对于询问我们从大到小枚举$\text{fib}$串累加每个字符的答案。最后可能会剩余一定长度无法消去，但剩余的长度一定不超过$|S|$（在$\text{fib}$串中甚至没有组成一个元素$S$），且是$S$的一个前缀，额外计算上这些部分即可。

​	

## Code

```c++
#include <cstdio>
#include <cstring>
using namespace std;
typedef long long ll;
const int N=200005;
int n,up,nex[N],type;
char str[N];
int pre[N][26];
ll l,r,cut,len[100],ans[26];
ll f[100][26];
void readData(){
	scanf("%s%lld%lld",str+1,&l,&r);
	n=strlen(str+1);
	n>>=1;
	for(int i=1;i<=n;i++){
		for(int j=0;j<26;j++) pre[i][j]=pre[i-1][j];
		pre[i][str[i]-'a']++;
	}
}
void getNex(){
	nex[1]=0;
	for(int i=2,j;i<=n;i++){
		j=nex[i-1];
		while(j&&str[j+1]!=str[i]) j=nex[j];
		nex[i]=(str[j+1]==str[i])?j+1:0;
	}
}
void preWork(){
	len[1]=n;
	for(int i=1;i<=n;i++){
		f[1][str[i]-'a']++;
		f[2][str[i]-'a']++;
	}
	len[2]=n+cut;
	for(int i=0;i<26;i++) 
		f[2][i]+=pre[cut][i];
	for(int i=3;;i++){
		for(int j=0;j<26;j++) f[i][j]=f[i-2][j]+f[i-1][j];
		len[i]=len[i-2]+len[i-1];
		if(len[i]>1e18){
			up=i;
			break;
		}
	}
}
void calc1(ll m,ll a){
	if(m<=0) return;
	ll t=m/cut;
	for(int i=0;i<26;i++) ans[i]+=a*t*pre[cut][i];
	t=m%cut;		
	for(int i=0;i<26;i++) ans[i]+=a*pre[t][i];
}
void calc2(ll m,ll a){
	for(int i=up;i>=1;i--)
		if(len[i]<=m){
			m-=len[i];
			for(int j=0;j<26;j++) ans[j]+=a*f[i][j];
		}
	for(int j=0;j<26;j++) ans[j]+=a*pre[m][j];
}
int main(){
	readData();
	getNex();
	cut=n-nex[n];
	if(n%cut==0){
		calc1(r,1);
		calc1(l-1,-1);
	}
	else{
		preWork();
		calc2(r,1);
		calc2(l-1,-1);
	}
	for(int i=0;i<26;i++) printf("%lld ",ans[i]);
	return 0;
}
```

