## Description

​	[题目链接](https://agc006.contest.atcoder.jp/tasks/agc006_e)



## Solution

​	显然每一列只能一起动，乱动则无解。

​	对原网格按列黑白染色，显然每一列数只能在相同颜色之间交换，乱动则无解。

​	之后考虑构造方案。

​	我们需要发(shou)现(wan)出一些好用的变换：

​		（1）使一种颜色的相邻两列同时上下翻转。

​		（2）使一种颜色的相邻两列交换，不翻转它们，而翻转另一个颜色中，不在这两列中间的，一个列。由于$n \ge 5$，我们总能实现这个操作。

​	统计出黑色列总共需要使用（2）交换多少次达到目标、以及需要翻转多少次（由于使用（2）不影响上下状态，直接用初始状态作比较即可），记为$flip_0$与$inv_0$，同理对白色列计算$flip_1$与$inv_1$。

​	我们首先使用（2）逐一实现每一个$flip_0$，每做一次$flip_0$的同时，$inv_1$也会减少一次。同理每做一次$flip_1$，$inv_0$也会减少一次。

​	优先做完$flip_0$与$flip_1$，此时$inv$不论是正负都没关系（负也是有意义的），只要是偶数，就可以使用（1）逐个翻转回来。重要的是此时$inv$必须是偶数，这意味着没操作前，当且仅当$flip_0$与$inv_1$的奇偶性相同，且$flip_1$与$inv_0$的奇偶性相同时，才有解，否则无解。

​	对于$flip$的计算，使用冒泡排序类模拟的套路，通过统计原位置右边有多少已操作的数来计算出当前实际位置，从而确定每一次冒泡的步数。

​	总时间复杂度$\mathcal O (n \log_2 n)$



## Code

```c++
#include <cstdio>
using namespace std;
const int N=100005;
int n;
int a[N][3],where[N];
int inv[2],flip[2];
inline int abs(int x){
	return x<0?-x:x;
}
void readData(){
	scanf("%d",&n);
	for(int j=0;j<3;j++)
		for(int i=1;i<=n;i++) scanf("%d",&a[i][j]);
}
bool preDraw(){
	for(int i=1;i<=n;i++){
		int id=(a[i][0]-1)/3+1;
		if((i-id)&1) return false;
		where[id]=i;
		int x=(id-1)*3+1,y=x+1,z=y+1;
		if(a[i][0]==x&&a[i][1]==y&&a[i][2]==z);
		else if(a[i][0]==z&&a[i][1]==y&&a[i][2]==x)
			inv[i&1]^=1;
		else return false;
	}
	return true;
}
namespace BIT{
	//suffix sum
	int a[N];
	inline void add(int u,int x){
		for(;u;u-=u&-u)
			a[u]+=x;
	}
	inline int que(int u){
		int res=0;
		for(;u&&u<=n;u+=u&-u)
			res+=a[u];
		return res;
	}
	inline void reset(){
		for(int i=1;i<=n;i++)
			a[i]=0;
	}
}
void calcFlip(){
	for(int i=1;i<=n;i+=2){
		int nowpos=where[i]+BIT::que(where[i])*2;
		flip[1]^=(abs(i-nowpos)/2)&1;
		BIT::add(where[i],1);
	}
	BIT::reset();
	for(int i=2;i<=n;i+=2){
		int nowpos=where[i]+BIT::que(where[i])*2;
		flip[0]^=(abs(i-nowpos)/2)&1;
		BIT::add(where[i],1);
	}
}
int main(){
	readData();
	if(!preDraw()){
		puts("No");
		return 0;
	}
	calcFlip();
	if(inv[0]!=flip[1]||inv[1]!=flip[0])
		puts("No");
	else 
		puts("Yes");
	return 0;
}
```

