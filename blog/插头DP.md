# 插头DP

## 简介

​	插头DP（轮廓线DP）是用来解决网格图回路问题的一种算法。

​	插头DP解决的经典问题就是统计经过所有格子的哈密顿回路条数，某些格子有障碍。

​	如果问题稍微进阶一点的话，不一定要求路径是回路、路径带权等等情况都可能出现。

​	它的时间复杂度比较高，但是已经属于比较高效的算法了。



## 基本概念

​	首先看经典问题：统计一个带障碍的$n*m$网格中$(n,m\le 12)$，经过所有格子的哈密顿回路的条数。

​	插头DP的状态围绕**轮廓线**进行转移。

​	我们的状态是$f(i,j,state)$，表示$(i,j)$格子转移完成后、轮廓线状态为$state$时的情况数。

​	轮廓线的形状相对于每一个$(i,j)$是确定的。$(i,j)$转移之前到转移之后，轮廓线的形状、变化如图所示：

![](C:\Users\Administrator\Pictures\Blog\插头DP\ct2.bmp)

​	即转移的格子$(i,j)$原来是轮廓线上的一个凸出点，转移后把轮廓线“从左上往右下拉”，使得$(i,j)$变成一个凹格。

​	轮廓线的长度为$m+1$，我们需要记录轮廓线的每一条边的上方（对于轮廓线中打竖的那条边，则是左方），是否有边作为接口，即是否有**插头**。这个信息，我们存在$state$中。

​	轮廓线所体现的，一是轮廓线上的插头状态，告诉你每一个地方是否应该用一条路径“接上”；二是所有路径的连通性：显然，路径是两两相连，两两成对的，我们分组标号来记录。下面用几幅图来表达一下轮廓线的记录方式：

![](C:\Users\Administrator\Pictures\Blog\插头DP\ct3.bmp)

​	括号的每个数代表对应轮廓线边的状态，"0"表示没有路径连接，其他数表示一对路径，每对路径的标号一样。

​	考虑到标号的大小没有保证，会导致状态的储存十分困难。

​	我们另寻它法：注意到每对路径在轮廓线上的接口不会相交，即不会出现$(...,x,...,y,...x,...y,...)​$的这种诡异情况。所以我们可以用括号序列来表示路径信息。还是上述三个例子：

​	![](C:\Users\Administrator\Pictures\Blog\插头DP\ct4.bmp)

​	这样一来，状态$state$可以看做一个$m+1$位的3进制数：0表示无接口，1表示一对路径的左端，2表示一对路径的右端。

​	

## 转移

​	$f(i,j,state)$的转移来源，是$f(i,j-1,state')$，但这样不好考虑和枚举。我们用$f(i,j,state)$，转移到$f(i,j+1,state')$。

​	我们将$f(i,j,\sum state)$转移到$f(i,j+1,\sum state')$称作大转移。

​	显然，我们只需要关注轮廓线上唯一变动的两条边：我们把轮廓线从左上拉到右下，这两条边的状态会更改，更改后是什么呢？取决于新的一格内路径怎么走：

​	![](C:\Users\Administrator\Pictures\Blog\插头DP\ct5.bmp)

​	我们现在关注转移的位点：

​	![](C:\Users\Administrator\Pictures\Blog\插头DP\ct6.bmp)

​	大转移的整体步骤是：枚举每一个状态$state$，得到$p_a$和$p_b$，根据$p_a$和$p_b$的取值，枚举$(i,j)$的路径走法，对于选择的$(i,j)$的走法，由$state$修改$p_a$和$p_b$分别变成$p_a'$和$p_b'$而得到新状态$state'$，执行$f(i,j+1,state')+=f(i,j,state)$。

​	根据$p_a,p_b$枚举走法分三大类情况：

​	（1）$p_a=0\;,p_b=0$：

​		这表明$(i,j)$的左方和上方没有路径连接，则当前格只能选取1号走法。

​		令$p_a'=1,p_b'=2$得到新状态。（转移前提：$i<n且j<m$，否则将出现连向网格边界的路径）

​	

​	（2）$p_a$和$p_b$恰好有一个是0：

​		这表明$(i,j)$的左方或者上方有一条路径连接。	

​		由于这种走法只是将连进来的路径延长，所以这条路径在原轮廓线和新轮廓线上的性质是一样的，括号表示相同，其值也相同。

​		令$p_a'=p_a+p_b,\;p_b'=0$代表3号（$p_a\neq0$）或5号（$p_b\neq0$）走法。（转移前提：$i<n$）

​		令$p_a'=0,\;p_b'=p_a+p_b$代表2号（$p_a\neq0$）或4号（$p_b\neq 0$）走法。（转移前提：$j<m$）

​	

​	（3）$p_a$和$p_b$皆不为0：

​		这表明$(i,j)$的左方和上方都有路径连接，所以$(i,j)$能填的只有6号走法，令$p_a'=0,\;p_b'=0$。可是我们将两条路径连接起来后，其他位置的状态也需要改变，因为括号序列发生了变动。

​		$p_a=1,\;p_b=1$：两条左端路径此刻相连，对于$b$对应的右端路径位置$c$，应该令$p_c'=1$，此时$c$和$a$对应的右端路径$d$是一对路径。

​		$p_a=2,\;p_b=2$：两条右端路径此刻相连，同上，对于$a$的左端路径位置$c$，令$p_c'=2$。

​		以上寻找对应括号路径，用括号序列配对的方式实现，复杂度$\mathcal O(m)$	

​		$p_a=1\;,p_b=2$：一条回路此时形成。**注意**！这个转移只能在最后一个非障碍的位置发生，在其他任意位置连成回路都是不合法的方案。而且，**答案就是被这种方式转移的量之和**。这种情况我们不需要转移了，你可以姑且理解为这已经是最后一步大转移；但是对于进阶问题，如回路不一定要经过所有点，则是因为这种转移不应该被后续过程所利用（因为路径已经形成），并且答案统计可以在任意位置进行。

​		$p_a=2,\;p_b=1$：相当于从中间拼接两条路径，这种情况最舒服，因为其他位置都不会有任何变化。

​	

​	转移种类较多，但是写起来其实是很好实现的。



## 实现

​	你可以将$f(i,j,state)$设成一个数组，但是这样非常不便，且大转移时要枚举所有状态，非常的缓慢，而在实际中，许多状态是不合法的。

​	我们用两个哈希表$s_0,s_1$来分别模拟$f(i,j)$和$f(i,j+1)$：所有有效的$f(i,j,state)$存在$s_0$中，大转移开始前，我们可以从$s_0$里提取所有有效状态，逐一转移至$s_1$中，也就是有效的$f(i,j+1,state)$。下一步大转移开始前，交换两个哈希表，并清空$s_1$即可。	

​	对于一个轮廓线状态$x$，为了操作方便，需要实现两个函数：提取某一位的值、改变某一位的值。上述括号配对表达方式中状态仅有012三种，但我们可以用四进制来表示，因为位运算的效率相对来说会比较高。

​	



## 总结

​	插头DP看起来十分难写，但只要自己动手实现一遍，就能理清楚插头DP的基本架构，就会发现它的实现方式其实挺简明的。建议读者还是自我摸索比较好。不过这里还是贴上[模板题BZOJ1814](https://www.lydsy.com/JudgeOnline/problem.php?id=1814)的代码：

```c++
#include <cstdio>
#define Push(x,y) s[hv].insert((x),(y));
using namespace std;
typedef long long ll;
const int N=13,BAS[13]={1,4,16,64,256,1024,4096,16384,65536,262144,1048576,4194304,16777216};
const int HASH_MOD=4001,S=50000;
int n,m,mp[N][N],ln,lm;
int qcnt,q1[S],hu,hv;
ll q2[S];
ll ans;
char str[N];
inline void swap(int &x,int &y){x^=y^=x^=y;}
inline int get(int st,int x){
	st>>=(x-1)<<1;
	return st-((st>>2)<<2);
}
inline void mdf(int &st,int x,int y){
	st+=(y-get(st,x))*BAS[x-1];
}
int match(int st,int x,int d){
	int top=1,stdcol=get(st,x);
	for(x+=d;top;x+=d){
		int v=get(st,x);
		if(v==0) continue;
		(v==stdcol)?top++:top--;
	}
	return x-d;
}
struct Hash{/*{{{*/
	int head[S],tot,id[S],nex[S];
	ll val[S];
	void reset(){
		tot=0;
		for(int i=0;i<HASH_MOD;i++) head[i]=0;
	}
	inline int get_hash(int x){return x%HASH_MOD;}
	void insert(int x,ll value){
		int hs=get_hash(x),u;
		for(u=head[hs];u&&id[u]!=x;u=nex[u]);
		if(!u){
			id[++tot]=x; val[tot]=value;
			nex[tot]=head[hs]; head[hs]=tot;
		}
		else val[u]+=value;
	}
	void layout(int &n,int *lis1,ll *lis2){
		n=0;
		for(int i=0;i<HASH_MOD;i++)
			for(int u=head[i];u;u=nex[u])
				n++,lis1[n]=id[u],lis2[n]=val[u];
	}
}s[2];/*}}}*/
void draw(int i,int j){
	int a=j,b=j+1,pa,pb;
	s[hu].layout(qcnt,q1,q2);	
	int x; ll y;
	while(qcnt){
		x=q1[qcnt]; y=q2[qcnt--];
		if(!y) continue;
		if(j==1) x=(x-get(x,m+1)*BAS[m])<<2;
		pa=get(x,a); pb=get(x,b);
		if(mp[i][j]==1){
			if(!pa&&!pb)
				Push(x,y);
			continue;
		}
		if(!pa&&!pb){
			if(i<n&&j<m){
				mdf(x,a,1); mdf(x,b,2);
				Push(x,y);
			}
		}
		else if(pa&&pb){
			if(pa==1&&pb==1){
				int pos=match(x,b,1);
				mdf(x,pos,1);
				mdf(x,a,0); mdf(x,b,0);
				Push(x,y);
			}
			else if(pa==2&&pb==2){
				int pos=match(x,a,-1);
				mdf(x,pos,2);
				mdf(x,a,0); mdf(x,b,0);
				Push(x,y);
			}
			else if(pa==1&&pb==2){
				if(i==ln&&j==lm){
					bool flag=true;			
					for(int k=1;k<=m+1&&flag;k++) if(k!=a&&k!=b&&get(x,k)) flag=false;
					if(flag) 
						ans+=y;
				}
			}
			else{//pa==2&&pb==1
				mdf(x,a,0); mdf(x,b,0);
				Push(x,y);
			}
		}
		else{
			int u=pa,v=pb;
			if(i<n){
				mdf(x,a,u+v); mdf(x,b,0);
				Push(x,y);
			}
			if(j<m){
				mdf(x,a,0); mdf(x,b,u+v);
				Push(x,y);
			}
		}
	}
}
int main(){
	scanf("%d%d",&n,&m);
	for(int i=1;i<=n;i++){
		scanf("%s",str+1);
		for(int j=1;j<=m;j++){
			if(str[j]=='.') mp[i][j]=0;
			else mp[i][j]=1;
			if(mp[i][j]!=1) ln=i,lm=j;
		}
	}
	hu=0; hv=1;
	s[hu].insert(0,1);
	for(int i=1;i<=n;i++)
		for(int j=1;j<=m;j++){
			draw(i,j);
			swap(hu,hv);
			s[hv].reset();
		}
	printf("%lld\n",ans);
	return 0;
}
```

​	

​	

[陈丹琦原论文]: https://wenku.baidu.com/view/9cfbb16e011ca300a6c390d5.html



