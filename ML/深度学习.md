# 分类

> ReLU：修正线性单元，修正指的是取不小于零的值。

- 标准神经网络（Standard NN）
- 卷积神经网络（CNN）
- 循环神经网络（RNN）
- 混合神经网络

![[Pasted image 20230707232425.png]]

# 处理对象

- 结构化数据：数字数据库
- 非结构化数据：图像、文本、音频

# 二分分类

## Logistic 回归

- 回归函数： 
$$ \hat{y^i} = \sigma \left(  w^T x^i + b \right) $$
- Sigmoid 函数：
$$ \sigma \left(  z \right) = \frac{1}{1 + e^{-z}} $$
- 损失函数：（基于单个样本）
$$ \iota \left(  \hat{y}^i,y^i \right) = 
-\Big(  y^i \log{\hat{y}^i} + \left( 1-y^i \right) \log{\left( 1-\hat{y}^i \right)} \Big) $$
- 代价函数：（基于总样本）
$$ J\left( w,b \right) = 
\frac{1}{m} \sum_{i=1}^{m} \iota \left( \hat{y}^i , y^i \right) 
= - \frac{1}{m} \sum_{i=1}^{m} y^i \log{\hat{y}^i} + \left( 1-y^i \right) \log{\left(1-\hat{y}^i \right)} $$
**梯度下降法**寻找代价函数的最小值，即为最优解：
$$ w := w - \alpha \frac{d J \left( w \right)}{dw} $$
其中 $\alpha$ 为学习率

![[Pasted image 20230708003238.png]]
