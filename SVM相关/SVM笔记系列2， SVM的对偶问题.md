<div align=center>
<font size="6"><b>《SVM笔记系列之二》SVM的对偶问题</b></font>
</div>

# 前言
**支持向量机的对偶问题比原问题容易解决，在符合KKT条件的情况下，其对偶问题和原问题的解相同，这里我们结合李航博士的《统计学习方法》一书和林轩田老师的《机器学习技法》中的内容，介绍下SVM的对偶问题。本人无专业的数学学习背景，只能直观上理解一些问题，请数学专业的朋友不吝赐教。**
**如有谬误，请联系指正。转载请注明出处。**

*联系方式：*
**e-mail**: `FesianXu@163.com`
**QQ**: `973926198`
**github**: `https://github.com/FesianXu`
**有关代码开源**: [click][click]

*****

# SVM的原问题的拉格朗日乘数表示
　　我们在上一篇博文《SVM笔记系列1，SVM起源与目的》中，谈到了SVM的原问题，这里摘抄如下：
$$
\min_{W,b} \frac{1}{2}||W||^2 \\
s.t. 1-y_i(W^Tx_i+b) \leq 0, \ i=1,\cdots,N
\tag{1.1}
$$
其满足形式:
$$
\min_{W,b} f(x) \\
s.t. c_i(x) \leq0, i=1,\cdots,k \\
h_j(x) = 0, j=1,\cdots,l
\tag{1.2}
$$

假设原问题为 $\theta_P(x)$ ，并且其最优解为 $p^*=\theta_P(x^*)$ 。
这是一个有约束的最优化问题，我们利用广义拉格朗日乘子法(具体移步[《拉格朗日乘数法和KKT条件的直观解释》][ref_1])，将其转换为无约束的形式：
$$
L(W,b,\alpha) = \frac{1}{2}||W||^2 + \sum_{i=1}^N \alpha_i (1-y_i(W^Tx_i+b)), \ \alpha_i \geq 0
\tag{1.3}
$$
变形为：
$$
L(W,b,\alpha) = \frac{1}{2}||W||^2 + \sum_{i=1}^N {\alpha_i}-\sum_{i=1}^N{\alpha_iy_i(W^Tx_i+b)} , \ \alpha_i \geq 0
\tag{1.4}
$$
我们将会得到原问题的另一个表述为：
$$
f(x) = \max_{\alpha} L(W, b, \alpha)=\max_{\alpha} \frac{1}{2}||W||^2 + \sum_{i=1}^N {\alpha_i}-\sum_{i=1}^N{\alpha_iy_i(W^Tx_i+b)},, \ \alpha_i \geq 0
\tag{1.5}
$$
$$
\theta_P(x) = \min_{W,b}f(x) = \min_{W,b} \max_{\alpha} L(W, b, \alpha)=\min_{W,b} \max_{\alpha} \frac{1}{2}||W||^2 + \sum_{i=1}^N {\alpha_i}-\sum_{i=1}^N{\alpha_iy_i(W^Tx_i+b)},, \ \alpha_i \geq 0
\tag{1.6}
$$
这里我觉得有必要解释下为什么 $f(x)$ 可以表述为 $\max_{\alpha} L(W, b, \alpha)$ 这种形式。
假设我们有一个样本点$x_i$是不满足原问题的约束条件 $1-y_i(W^Tx_i+b) \leq 0$ 的，也就是说 $1-y_i(W^Tx_i+b) \gt 0$ ，那么在 $\max_{\alpha}$ 这个环节就会使得 $\alpha_i \rightarrow +\infty$ 从而使得 $L(W,b,\alpha) \rightarrow +\infty$ 。如果$x_i$是满足约束条件的，那么为了求得最大值，因为$1-y_i(W^Tx_i+b) \leq 0$而且 $\alpha_i \geq 0$，所以就会使得 $\alpha_i = 0$。由此我们得知：
$$
\max_{\alpha}L(W,b,\alpha) = \begin{cases}
\frac{1}{2}||W||^2 & 1-y_i(W^Tx_i+b) \leq 0 满足约束条件\\
+\infty & 1-y_i(W^Tx_i+b) \gt 0 不满足约束条件
\end{cases}
\tag{1.7}
$$
因此在满足约束的情况下，
$$
\max_{\alpha}L(W,b,\alpha)=\frac{1}{2}||W||^2
$$
不满足约束条件的样本点则因为无法对正无穷求最小值而自然抛弃。
这个时候，我们试图去解 $\max_{\alpha}L(W,b,\alpha)$ 中的 $\max_{\alpha}$ 我们会发现因为 $L(W,b,\alpha)=\frac{1}{2}||W||^2 + \sum_{i=1}^N {\alpha_i}-\sum_{i=1}^N{\alpha_iy_i(W^Tx_i+b)}$ 对于 $\alpha$ 是线性的，非凸的[^1]，因此无法通过梯度的方法求得其最大值点，其最大值点应该处于可行域边界上，因此我们需要得到SVM的对偶问题进行求解。
**至此，我们得到了原问题的最小最大表述：**
$$
\theta_P(x) = \min_{W,b} \max_{\alpha} L(W, b, \alpha)=\min_{W,b} \max_{\alpha} \frac{1}{2}||W||^2 + \sum_{i=1}^N {\alpha_i}-\sum_{i=1}^N{\alpha_iy_i(W^Tx_i+b)}, \alpha_i \geq0,i=1,\cdots,N
\tag{1.8}
$$

****
# SVM的对偶问题
　　从上面的讨论中，我们得知了SVM的原问题的最小最大表达形式为：
$$
\theta_P(x) = \min_{W,b} \max_{\alpha} L(W, b, \alpha)=\min_{W,b} \max_{\alpha} \frac{1}{2}||W||^2 + \sum_{i=1}^N {\alpha_i}-\sum_{i=1}^N{\alpha_iy_i(W^Tx_i+b)}, \alpha_i \geq0,i=1,\cdots,N
\tag{2.1}
$$
设SVM的对偶问题为 $\theta_D(\alpha)$ ，其最优解为 $d^*=\theta_D(\alpha^*)$ ，可知道其为：
$$
g(x) = \min_{W,b} L(W,b,\alpha)=\min_{W,b} \frac{1}{2}||W||^2 + \sum_{i=1}^N {\alpha_i}-\sum_{i=1}^N{\alpha_iy_i(W^Tx_i+b)}
\tag{2.2}
$$
$$
\theta_D(\alpha) = \max_{\alpha}g(x) = \max_{\alpha} \min_{W,b} L(W,b,\alpha)=\max_{\alpha} \min_{W,b} \frac{1}{2}||W||^2 + \sum_{i=1}^N {\alpha_i}-\sum_{i=1}^N{\alpha_iy_i(W^Tx_i+b)}
\tag{2.3}
$$
此时，我们得到了对偶问题的最大最小表述，同样的，我们试图去求解 $\theta_D(\alpha)$ 中的 $\min_{W,b}$ ，我们会发现由于 $L(W,b,\alpha)=\frac{1}{2}||W||^2 + \sum_{i=1}^N {\alpha_i}-\sum_{i=1}^N{\alpha_iy_i(W^Tx_i+b)}$ 对于$W$来说是凸函数，因此可以通过梯度的方法求得其最小值点（即是其极小值点）。



求解 $\min_{W,b} L(W,b,\alpha)$ ，因为 $L(W,b,\alpha)$ 是凸函数，我们对采用求梯度的方法求解其最小值（也是KKT条件中的， $\nabla_WL(W,b,\alpha)=0$ 和 $\nabla_b L(W,b,\alpha)=0$ ）：
$$
\frac{\partial{L}}{\partial{W}}=W-\sum_{i=1}^N\alpha_iy_ix_i=0, i=1,\cdots,N
\tag{2.4}
$$
$$
\frac{\partial{L}}{\partial{b}}=\sum_{i=1}^N\alpha_iy_i=0,i=1,\cdots,N
\tag{2.5}
$$
得出：
$$
W=\sum_{i=1}^N\alpha_iy_ix_i,　\sum_{i=1}^N\alpha_iy_i=0,　\alpha_i \geq0,i=1,\cdots,N
\tag{2.6}
$$
将其代入 $g(x)$ ，注意到 $\sum_{i=1}^N\alpha_iy_i=0$ ,得：
$$
g(x) =
\frac{1}{2} \sum_{i=1}^N \alpha_iy_ix_i \sum_{j=1}^N a_jy_jx_j+\sum_{i=1}^N\alpha_i
-\sum_{i=1}^N\alpha_iy_i(\sum_{j=1}^N \alpha_jy_jx_j \cdot x_i+b)=
-\frac{1}{2}\sum_{i=1}^N \sum_{j=1}^N \alpha_i \alpha_jy_iy_j(x_i \cdot x_j)+ \sum_{i=1}^N\alpha_i
$$
整理为:
$$
\max_{\alpha}g(x) = \max_{\alpha}
-\frac{1}{2}\sum_{i=1}^N \sum_{j=1}^N \alpha_i \alpha_jy_iy_j(x_i \cdot x_j)+ \sum_{i=1}^N\alpha_i \\
s.t. \ \sum_{i=1}^N\alpha_iy_i=0 \\
\alpha_i \geq0,i=1,\cdots,N
\tag{2.7}
$$

等价为求最小问题:
$$
\min_{\alpha}g(x) = \min_{\alpha}
\frac{1}{2}\sum_{i=1}^N \sum_{j=1}^N \alpha_i \alpha_jy_iy_j(x_i \cdot x_j)- \sum_{i=1}^N\alpha_i \\
s.t. \ \sum_{i=1}^N\alpha_iy_i=0 \\
\alpha_i \geq0,i=1,\cdots,N
\tag{2.8}
$$


根据**Karush–Kuhn–Tucker(KKT)条件**[^2],我们有：
$$
\nabla_WL(W^*,b^*,\alpha^*)=W^*-\sum_{i=1}^N\alpha_i^*y_ix_i=0 \Longrightarrow W^* = \sum_{i=1}^N\alpha_i^*y_ix_i
\tag{2.9}
$$
$$
\nabla_bL(W^*,b^*,\alpha^*) =
-\sum_{i=1}^N \alpha^*_i y_i=0
\tag{2.10}
$$
$$
\alpha^*_i(1-y_i(W^*x_i+b^*))=0
\tag{2.11}
$$
$$
1-y_i(W^*x_i+b^*) \leq 0
\tag{2.12}
$$
$$
\alpha^*_i \geq 0
\tag{2.13}
$$
前两个式子我们已经在求极值的时候利用了，得知:
$$
W^* = \sum_{i=1}^N\alpha_i^*y_ix_i
\tag{2.14}
$$
并且其中至少有一个 $\alpha_j^* \gt 0$ ，对此 $j$ 有，$y_j(W^*x_j+b^*)-1=0$
代入刚才的$W^*$，我们有
$$
b^*=y_j-\sum_{i=1}^N\alpha^*_iy_i(x_i \cdot x_j)
\tag{2.15}
$$
所以决策超平面为：
$$
\sum_{i=1}^N \alpha^*_iy_i(x_i \cdot x)+b^*=0
\tag{2.16}
$$
分类超平面为：
$$
\theta(x)=sign(\sum_{i=1}^N \alpha^*_iy_i(x_i \cdot x)+b^*)
\tag{2.17}
$$
其中，我们可以观察到超平面只是依赖于 $\alpha_i^*>0$的样本点$x_i$，而其他样本点对其没有影响，所以这些样本是对决策超平面起着决定性作用的，因此我们将 $\alpha_i^*>0$对应的样本点集合 $x_i$称为**支持向量**。同时，我们可以这样理解当 $\alpha^*_i >0$时，我们有 $1-y_i(W^*x_i+b)=0$，这个恰恰是表明了**支持向量**的函数间隔都是1，恰好和我们之前的设定一致。
![svm_margin][svm_margin]


至此，我们得到了**硬间隔线性支持向量机**的数学表述形式，所谓硬间隔线性支持向量机，就是满足我之前的假设
> 两类样本是线性可分的，总是存在一个超平面$W^Tx+b$可以将其完美分割开。

但是，在现实生活中的数据往往是或本身就是非线性可分但是近似线性可分的，或是线性可分但是具有噪声的，以上两种情况都会导致在现实应用中，**硬间隔线性支持向量机**变得不再实用，因此我们将会在后续讨论用以解决近似线性可分的**软间隔线性支持向量机**和**基于kernel的支持向量机**，后者可以解决非线性可分的问题。下图表示了**硬间隔线性支持向量机**和**软间隔支持向量机**之间的区别。
![hard_margin][hard_margin]
![soft_margin][soft_margin]

在下一篇中，我们紧接着现在的内容，介绍**序列最小最优化算法（Sequential Minimal Optimization,SMO）**，用于求解 $\theta_D(x)$，得到 $\alpha^*_i$以便于得到超平面的$W^*$和$b$。我们将在其他文章中介绍**软间隔线性支持向量机**，**广义拉格朗日乘数法**，**KKT条件**和**基于kernel的支持向量机**。

**这里我们要记住我们需要最优化的目的式子，我们以后将会反复提到这个式子。**
$$
\min_{\alpha}
\frac{1}{2}\sum_{i=1}^N \sum_{j=1}^N \alpha_i \alpha_jy_iy_j(x_i \cdot x_j)- \sum_{i=1}^N\alpha_i
$$
$$
s.t. \ \sum_{i=1}^N\alpha_iy_i=0
$$
$$
\alpha_i \geq0,i=1,\cdots,N
$$






[^1]: 易证明。参考wikipedia的凸函数定义。
[^2]: 事实上，如果 $\theta_D(x)$ 的 $L(W,b,\alpha)$ 满足KKT条件，那么在SVM这个问题中，$W^*$ 和 $b^*$ 和 $\alpha^*_i$ 同时是原问题和对偶问题的解的充分必要条件是满足KKT条件，具体见《统计学习方法》附录和[《拉格朗日乘数法和KKT条件的直观解释》](http://blog.csdn.net/loseinvain/article/details/78624888)。


[click]: https://github.com/FesianXu/AI_Blog/tree/master/SVM%E7%9B%B8%E5%85%B3
[svm_margin]: ./imgs/svm_margin_2.png
[hard_margin]: ./imgs/hard_margin_svm.png
[soft_margin]: ./imgs/soft_margin_svm.png


[ref_1]: http://blog.csdn.net/loseinvain/article/details/78624888
