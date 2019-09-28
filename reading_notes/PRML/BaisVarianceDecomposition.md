> yangliang @ 20171007 @ sigma



[TOC]

todo

* 以分类为例推导误差分解，参见西瓜书
* 方差、偏差大小图示；多项式回归实验
* 整体梳理





# 相关背景

* bias- variance trade-off：a frequentist viewpoint of the model complexity issue 
* 模型评估指标
* 以回归 and 平方误差为例



|    符号    |              含义               |
| :------: | :---------------------------: |
|   $x$    |             单个样本              |
|   $D$    |              样本集              |
| $y(x;D)$ | 由训练集 $D$ 学得的模型 $y$ 对 $x$ 的预测值 |
|  $h(x)$  |      optimal prediction       |
|          |                               |
|  $E(L)$  |             泛化误差              |





# 泛化误差分解

期望误差
$$
E(L) = \int\int L(t,y(x))p(x,t)dxdt
$$
通常选取 $L(t,y(x)) =\{y(x)-t\}^2​$，展开即得 $L(t,y(x)) =\{y(x)-E[t|x]+E[t|x]-t\}^2=\{y(x)-E[t|x]\}^2+2\{y(x)-E[t|x]\}\{E[t|x]-t\}+\{E[t|x]-t\}^2​$，代入 $E(L)​$ 中得到
$$
E(L) = \int \{y(x)-E[t|x]\}^2p(x)dx+ \int \{E[t|x]-t\}^2p(x)dxdt
$$
上面第1项表示预测函数 $y(x)$ 和真实值得误差，第2项表示数据的噪音，和 $y(x)$ 的选取无关。



optimal prediction 由以下条件期望 $h(x)$ 给出
$$
h(x) = E[t|x] = \int {tp(t|x)dt}
$$
于是 $E(L)$ 可以写作
$$
E(L) = \int \{y(x)-h(x)\}^2p(x)dx + \int \{h(x)-t\}p(x,t)dxdt
$$
我们的目标是寻找最佳的 $y(x)$ 使得 $E(L)$ 最小。容易知道上式中第二项和 $y(x)$ 的选取无关，因此希望选取 $y(x)  = h(x)$，当然这是建立在我们有无穷多的样本基础上。事实上，我们的数据集 $D$ 总是有限的，因此 $ h(x)$ 对于我们总是未知的。

有N 个独立同分布 p(t,x) 训练数据组成数据集 D，假设我们用含参函数 y(x,w) 来对 h(x) 建模。



考察单个样本 x 的损失
$$
\{y(x;D)-h(x)\}^2
$$
按照如下展开
$$
\{y(x;D) - h(x)\}^2 = \{y(x;D) - E_D[y(x;D)] + E_D[y(x;D)] - h(x)\}^2   \\ = \{y(x;D) - E_D[y(x;D)]\}^2 + \{E_D[y(x;D)] - h(x)\}^2    \\   + 2\{y(x;D) - E_D[y(x;D)]\}\{E_D[y(x;D)] - h(x)\}
$$
等式2边针对数据集 D 取期望
$$
E_D[\{y(x;D)-h(x)\}^2] = \{E_D[y(x;D)] - h(x)\}^2 + E_D[\{y(x;D)-E_D[y(x;D)]\}^2]
$$
综合(4)式得到
$$
expected \ loss = bias^2 + variance + noise
$$
其中
$$
bias^2 = \int \{E_D[y(x;D)]-h(x)\}^2p(x)dx
$$

$$
variance = \int E_D[\{y(x;D)-E_D[y(x;D)]\}^2] p(x)dx
$$

$$
noise = \int \{h(x)-t\}^2p(x,t)dxdt
$$



# 一个例子



# 深入理解

泛化误差 = 偏差 + 方差 + 噪声

`偏差` 度量了学习算法的期望预测和真实结果的偏离程度，即刻画了 **学习算法本身的拟合能力**。

`方差` 度量了同样大小的训练集的变动所导致的学习性能的变化，即刻画了 **数据扰动所造成的影响** 。

`噪声` 则表达了在当前任务上任何学习算法所能达到的期望泛华误差的下界，即刻画了 **学习问题本身的难度**。

泛化误差的分解说明，泛化性能是由学习算法的能力、数据的充分性、学习任务本身的难度所共同确定的。给定学习任务，为了取得好的泛化性能，则需要使偏差较小即能够充分拟合数据；并且使方差较小，即使得数据扰动产生的影响小。



# 参考资料

* http://liuchengxu.org/blog-cn/posts/bias-variance/
* PRML Chapter3.2节 The Bias-Variance Decomposition
* https://www.zhihu.com/question/27068705
* 周志华《机器学习》