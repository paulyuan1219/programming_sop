### 协方差 Cov(X, Y)

In discrete cases, the formula for the covariance of two random variables \(X\) and \(Y\) is given by:

$$\text{Cov}(X, Y) = \frac{1}{n} \sum_{i=1}^{n} (x_i - \bar{x})(y_i - \bar{y})$$

$$\text{Cov}(X, Y) = \sum p_i (x_i - \mu_X)(y_i - \mu_Y)$$

The formula for the covariance of two random variables \(X\) and \(Y\) is given by:

$$\text{Cov}(X, Y) = E[(X - \mu_X)(Y - \mu_Y)] $$


where:
- $\( n \)$ is the number of observations.
- $\( x_i \)$ and \( y_i \) are the \(i\)-th observations of \(X\) and \(Y\) respectively.
- \( \bar{x} \) is the mean of \(X\), calculated as \( \bar{x} = \frac{1}{n} \sum_{i=1}^{n} x_i \).
- \( \bar{y} \) is the mean of \(Y\), calculated as \( \bar{y} = \frac{1}{n} \sum_{i=1}^{n} y_i \).

Covariance measures the linear relationship between two random variables. If the covariance is positive, it indicates that the two variables tend to increase or decrease together. If the covariance is negative, it indicates that one variable tends to increase when the other decreases. If the covariance is zero, it indicates no linear relationship between the variables.

#### 简单理解
- 当X增加，Y增加，则Cov(X,Y) > 0
- 当X增加，Y减少，则Cov(X,Y) < 0
- 其他情况，则Cov(X, Y) = 0


## [“损失函数”是如何设计出来的？直观理解“最小二乘法”和“极大似然估计法”](https://www.youtube.com/watch?v=pjfnMgAnnIk&list=PLxIHUhMHF8okwhq8poRuiHBChWjkVUHLL&index=4)

### 线性模型

$$
\mathbf{\hat{y}} = \mathbf{W} \mathbf{x} + b
$$


三个关键点
- 最小二乘法
- 极大似然估计
- 交叉熵

#### 最小二乘法
单个样本上的损失函数如下
$$l^{(i)}(\mathbf{w}, b) = \frac{1}{2} \left(\hat{y}^{(i)} - y^{(i)}\right)^2.$$

所有样本上的损失函数如下。

$$L(\mathbf{w}, b) =\frac{1}{n}\sum_{i=1}^n l^{(i)}(\mathbf{w}, b) =\frac{1}{n} \sum_{i=1}^n \frac{1}{2}\left(\mathbf{w}^\top \mathbf{x}^{(i)} + b - y^{(i)}\right)^2.$$

在训练模型时，我们希望寻找一组参数（$\mathbf{w}^*, b^*$），
这组参数能最小化在所有训练样本上的总损失。如下式：

$$\mathbf{w}^*, b^* = \operatorname*{argmin}_{\mathbf{w}, b}\  L(\mathbf{w}, b).$$

##### 最小二乘法的解析解

线性回归刚好是一个很简单的优化问题。首先，我们将偏置$b$合并到参数$\mathbf{w}$中，合并方法是在包含所有参数的矩阵中附加一列。
我们的预测问题是最小化$$\|\mathbf{y} - \mathbf{X}\mathbf{w}\|^2$$。
这在损失平面上只有一个临界点，这个临界点对应于整个区域的损失极小点。
将损失关于$\mathbf{w}$的导数设为0，得到解析解：

$$\mathbf{w}^* = (\mathbf X^\top \mathbf X)^{-1}\mathbf X^\top \mathbf{y}.$$


To find the solution to the minimization problem \( \|\mathbf{y} - \mathbf{X}\mathbf{w}\|^2 \), we are essentially trying to minimize the sum of squared errors between the observed values \( \mathbf{y} \) and the predicted values \( \mathbf{X}\mathbf{w} \). This is a common problem in linear regression, where the goal is to find the optimal weight vector \( \mathbf{w} \) that minimizes this error.

The problem can be written as:

$$
\min_{\mathbf{w}} \|\mathbf{y} - \mathbf{X}\mathbf{w}\|^2
$$

Expanding and differentiating with respect to \( \mathbf{w} \), we get:

1. **Objective Function:**

$$
\|\mathbf{y} - \mathbf{X}\mathbf{w}\|^2 = (\mathbf{y} - \mathbf{X}\mathbf{w})^T(\mathbf{y} - \mathbf{X}\mathbf{w})
$$

2. **Expanding:**

$$
(\mathbf{y} - \mathbf{X}\mathbf{w})^T(\mathbf{y} - \mathbf{X}\mathbf{w}) = \mathbf{y}^T\mathbf{y} - 2\mathbf{y}^T\mathbf{X}\mathbf{w} + \mathbf{w}^T\mathbf{X}^T\mathbf{X}\mathbf{w}
$$

3. **Taking the Gradient with respect to \( \mathbf{w} \):**

$$
\frac{\partial}{\partial \mathbf{w}} \left( \mathbf{y}^T\mathbf{y} - 2\mathbf{y}^T\mathbf{X}\mathbf{w} + \mathbf{w}^T\mathbf{X}^T\mathbf{X}\mathbf{w} \right) = -2\mathbf{X}^T\mathbf{y} + 2\mathbf{X}^T\mathbf{X}\mathbf{w}
$$

4. **Setting the Gradient to Zero:**

$$
-2\mathbf{X}^T\mathbf{y} + 2\mathbf{X}^T\mathbf{X}\mathbf{w} = 0
$$

5. **Solving for \( \mathbf{w} \):**

$$
\mathbf{X}^T\mathbf{X}\mathbf{w} = \mathbf{X}^T\mathbf{y}
$$

$$
\mathbf{w} = (\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T\mathbf{y}
$$

Therefore, the optimal solution \( \mathbf{w} \) is given by:

$$
\mathbf{w} = (\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T\mathbf{y}
$$

This solution assumes that \( \mathbf{X}^T\mathbf{X} \) is invertible. If it's not, then alternative methods like regularization or pseudoinverse are used.

### 极大似然估计
似然函数，就是给定观测值，把模型参数视为变量，来计算一个模型参数下出现观测值的概率。

神经网络，使用$NN(\mathbf W, \mathbf b)$去逼近人脑里面的概率模型。

注意，这里的$x_i$的取值要么是0，要么是1。所以是一个贝努力分布
$P(X = x) = p^x (1 - p)^{1 - x}$

$$
\begin{aligned}
L(\theta) &= P(x_1, x_2, \ldots, x_n \mid \theta) \\
&= \prod_{i=1}^{n} P(x_i \mid \theta) \\
&= \prod_{i=1}^{n} P(x_i \mid W, b) \\
&= \prod_{i=1}^{n} P(x_i \mid y_i) \\
&= \prod_{i=1}^{n} y_i^{x_i} (1-y_i)^{1-x_i}
\end{aligned}
$$

为了方便，可以取log
$$
\begin{aligned}
log(L(\theta)) &= log(\prod_{i=1}^{n} y_i^{x_i} (1-y_i)^{1-x_i}) \\
&= \sum_{i=1}^{n} log(y_i^{x_i} (1-y_i)^{1-x_i}) \\
&= \sum_{i=1}^{n} ({x_i}log(y_i) + (1-x_i)log(1-y_i))\\
\end{aligned}
$$

所以，最大化$log(L(\theta))$等价于最小化$-log(L(\theta))$

$$
\begin{aligned}
-log(L(\theta)) &= - ( \sum_{i=1}^{n} ({x_i}log(y_i) + (1-x_i)log(1-y_i)) )\\
\end{aligned}
$$

注意，上面这个式子是用MLE推导出来了，为什么说这个也是交叉熵呢？

### 交叉熵

记住，我们的最终目的，是需要比较理想模型和我们NN学习出来的模型。这两个都是概率模型。

这里，我们把他们都转化成为熵，然后进行比较。

例如，如果两个概率模型都是高斯分布，那么只需要比较两个参数就可以了。但是，一个是高斯分布，一个是泊松分布，就没法直接比较了。

对于概率模型来说，熵就是一个统一的工度
