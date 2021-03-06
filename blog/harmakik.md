对于原树一个节点$x$：

$f_x(h)$表示，$x$作为一个深度为$h$的点时，$x$及其子树的安排方案有多少（不考虑$x$具体在深度为$h$的哪个点）

$F_x(h)$表示，对于一个固定的深度为$h$的节点$y$，$x$在$y$或其子树中，$x$及其子树的安排方案有多少。

则有关系：
$$
F_x(h)=\sum_{i\ge h}f_x(i)*2^{i-h}
$$

对于叶子：  
$$
F_x(h)=[h\le h_x]2^{h_x-h}
$$

已知二者都可以表示成这些形式

$$
f_x(h)=\sum_{i\geq 0}c_i*2^{-ih}\\
F_x(h)=\sum_{i\geq 0}c_i*2^{-ih}
$$

对于叶子$x​$，赋值后直接回溯：
$$
c_1=2^{h_x}
$$


依照60分DP，可以推出由儿子到自己的转移（两个$c$分别是两个$F$的$c$，$c'$是转移后的$f_x$的$c$）：
$$
\begin{aligned}
f_x(h)&=F_l(h+1)F_r(h+1)\\
&=(\sum_{i\geq 0}c_i2^{-i(h+1)})(\sum_{j\geq 0}c_j2^{-j(h+1)})\\
&=(\sum_{i\geq 0}\frac{c_i}{2^i}2^{-ih})(\sum_{j\geq 0}\frac{c_j}{2^j}2^{-jh})\\
&=\sum_{i\ge0}{c'}_i2^{-ih}\\
\end{aligned}
$$

当然，也可以在卷积完之后每个$c_i$除去$2^i$

观察到这个卷积，再考虑边界，$c$的下标为$0...siz[x]$，$siz[x]$为$x$子树中叶子数。暴力卷积，用树上背包思路分析，这一步的复杂度是全局$\mathcal O(n^2)$的

得到自己的$f$后，由于父亲要使用自己的$F$，所以根据定义式由$f$推出$F$：
$$
\begin{aligned}
F_x(h)&=\sum_{h \le i<maxh}f_x(i)*2^{i-h}\\
&=\sum_{h \le i<maxh}\sum_{j\ge 0}c_j*2^{i(1-j)-h}\\
&=\sum_{j\ge0}\frac{c_j}{2^h}\sum_{h \le i<maxh}(2^{(1-j)})^i\\
&=\sum_{j\ge0}\frac{c_j}{2^h}\frac{(2^{1-j})^{maxh}-(2^{1-j})^{h}}{(2^{1-j})-1}\\
&=\sum_{j\ge 0}c_j(\frac{2^{(1-j)maxh}}{2^{1-j}-1}2^{-h}-\frac{1}{2^{1-j}-1}2^{-jh})
\end{aligned}
$$



答案即$F_1(0)$，$\sum c_i$







































