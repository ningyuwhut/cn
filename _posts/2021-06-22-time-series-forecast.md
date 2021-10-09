---
  layout: post
  title: 时间序列预估
  categories: MachineLearning
  tags:
--- 

简单指数平滑

适用于没有趋势和周期性的数据

过去所有时刻的数据的加权平均。权重是随着时间推移指数衰减的，即离当前时刻越远，权重越小。

衰减的速度由平滑参数$$\alpha$$控制。该值越大，说明距离近的时刻权重越大，反之，越小。

$$
\begin{equation}
  \hat{y}_{T+1|T} = \alpha y_T + \alpha(1-\alpha) y_{T-1} + \alpha(1-\alpha)^2 y_{T-2}+ \cdots,   \tag{7.1}
\end{equation}
$$

另一种表达形式:

$$
\begin{align*}
  \text{Forecast equation}  && \hat{y}_{t+h|t} & = \ell_{t}\\
  \text{Smoothing equation} && \ell_{t}        & = \alpha y_{t} + (1 - \alpha)\ell_{t-1}，
\end{align*}
$$

带有趋势的时间序列预估，又叫二阶指数平滑

$$
\begin{align*}
  \text{Forecast equation}&& \hat{y}_{t+h|t} &= \ell_{t} + hb_{t} \\
  \text{Level equation}   && \ell_{t} &= \alpha y_{t} + (1 - \alpha)(\ell_{t-1} + b_{t-1})\\
  \text{Trend equation}   && b_{t}    &= \beta^*(\ell_{t} - \ell_{t-1}) + (1 -\beta^*)b_{t-1}，
\end{align*}
$$

$$\ell_t$$是在t时刻该时间序列的水平的估计值,$$b_t$$是t
  时刻的趋势（斜率）的估计,$$\alpha$$是水平方向的平滑参数，$$\beta$$是趋势的平滑参数。


holt-winter算法(三阶指数平滑)

$$
\begin{align*}
  \hat{y}_{t+h|t} &= \ell_{t} + hb_{t} + s_{t-m+h_{m}^{+}} \\
  \ell_{t} &= \alpha(y_{t} - s_{t-m}) + (1 - \alpha)(\ell_{t-1} + b_{t-1})\\
  b_{t} &= \beta^*(\ell_{t} - \ell_{t-1}) + (1 - \beta^*)b_{t-1}\\
  s_{t} &= \gamma (y_{t}-\ell_{t-1}-b_{t-1}) + (1-\gamma)s_{t-m}，
\end{align*}
$$

$$
\begin{align*}
  \hat{y}_{t+h|t} &= (\ell_{t} + hb_{t})s_{t-m+h_{m}^{+}} \\
  \ell_{t} &= \alpha \frac{y_{t}}{s_{t-m}} + (1 - \alpha)(\ell_{t-1} + b_{t-1})\\
  b_{t} &= \beta^*(\ell_{t}-\ell_{t-1}) + (1 - \beta^*)b_{t-1}                \\
  s_{t} &= \gamma \frac{y_{t}}{(\ell_{t-1} + b_{t-1})} + (1 - \gamma)s_{t-m}
\end{align*}
$$

ARIMA




http://www.dathinking.com/cn/2014/05/Exponential-Smoothing/

https://otexts.com/fppcn/holt.html