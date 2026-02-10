---
title: 图形学：Monte Carlo
published: 2026-02-03
description: ''
tags: [计算机图形学, 路径追踪, 蒙特卡洛, Monte Carlo]
category: 'Graphics'
draft: false 
lang: ''
---

## Monte Carlo 是什么？

Monte Carlo 是一种随机采样的算法，它的思想在于通过大量的随机采样来逼近一个确定的值。

比如说目前我们知道三维高斯分布的函数表达式：

$$
f(x, y) = \frac{e^{-\frac{x^2+y^2}{2}}}{2\pi}
$$

，但是我们不知道它的曲面积分是多少，也不想算。那么这个时候我们就可以在该函数表达式的积分域内进行随机采样。采样较多的次数，最后将多次采样值加起来除以采样总次数就得到了三维高斯分布在积分域内的平均值。通过这个平均值我们再乘以积分域面积就可以得到这个三维高斯分布在积分域内的曲面积分。因此我们可以得出 Monte Carlo 积分的表达式：

$$
\int_A f(x, y) dA \approx \frac{A}{N} \sum_{i=1}^{N} f(x_i, y_i)
$$

，其中 $A$ 是积分域面积，$N$ 是采样总次数，$(x_i, y_i)$ 是在积分域内均匀随机采样得到的点。

:::note[另一种数学表示形式]

在计算机图形学中，Monte Carlo 积分的表达式通常会被写成：

$$
\int_A f(x, y) dA \approx \frac{1}{N} \sum_{i=1}^{N} \frac{f(x_i, y_i)}{PDF(x_i, y_i)}
$$

这是因为通常我们的积分域内采样的权重是不一样的，所以说我们需要通过一个函数来描述积分域内不同区域的采样权重，那么就需要引入 [PDF](https://yang-zhihang.github.io/posts/computer-graphics/pdf/)。有了 PDF 之后，我们就可以通过重要性采样使蒙特卡洛采样更快地收敛到我们想要的结果：降低方差，从而让画面在更少的采样数量下更快收敛且画面更干净。

:::

通过上面我们对 Monte Carlo 积分的一些基本了解，我们知道了 Monte Carlo 其实是一种「使用暴力手段来得到我们想要的解」的一种算法。因为手段暴力，所以蒙特卡洛积分带来的时间代价是非常大的。

## Monte Carlo 有什么用？

[Ray Tracing in One Weekend](https://raytracing.github.io/) 系列书籍的第三部中着重介绍了 Monte Carlo 积分在路径追踪当中的应用：当光打到 Lambert 材质的物体时，光遵从一个假设「在半球内散射的密度与极角余弦 $cos\theta$ 成正比」。这个时候如果说我们想要得到这个像素点反映出来的最真实的像素色彩，那么我们就需要通过 Monte Carlo 积分方法进行多次采样，最后将结果通过 PDF 加权修正再取平均得到我们想要的那个像素值。