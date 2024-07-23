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


