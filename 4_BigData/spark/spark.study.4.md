#spark学习笔记4

本文主要介绍spark机器学习库`MLlib`中的`optimization`模块。该模块包含以下文件: 

* Gradient.scala: 定义`梯度类`，提供函数计算损失函数在某点的梯度，包含LogisticGradient、LeastSquaresGradient、HingeGradient
* Optimizer.scala: 凸优化问题`求解器`
* GradientDescent.scala: `梯度下降法`求解凸优化问题
* LBFGS.scala: `LBFGS`求解凸优化问题
* Updater.scala: 求解凸优化问题，各种状态(权重)更新器，包含SimpleUpdater、L1Updater、SquaredL2Updater
* NNLS.scala: nonnegative least squares problems求解器


Optimizer凸优化问题求解器，用于求解下下凸问题 

min  L(w) + regParam * R(w)

Gradient表示下降方向
Update实施具体的梯度更新、位置更新，以及惩罚项的计算。

