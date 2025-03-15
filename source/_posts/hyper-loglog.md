---
title: hyper_loglog
date: 2025-03-15 21:44:47
tags:
- 论文
---
 <!-- more -->

## hyperloglog：快速估算不重复元素数量

基数统计：
对数据流$D=\{x_1,x_2...x_n\}$-->不重复元素个数$S$,称为基数。

一个显而易见的方法：

$$x_i在map/set ? \begin{cases}
  & \text{ 在 } :基数不变 \\
  & \text{ 不在 }: x->map/set,S++
\end{cases}$$
空间复杂度很高。

hyperloglog：估计，误差范围内——>$S$。

#### 算法流程

$D=\{i|x_i\}$
$x_i--hash-->(...w_2,w_1)_2,w_i=0/1$

寻找一个$b$位把二进制结果分为两半。

$...w_{b+2},w_{b+1}|w_b,...w_1 -->第？桶$

桶的个数$m=2^b$

对每个桶进行计数：

$M_1=max_w\rho(w)$
$M_1=max_w\rho(w)$
...
$M_m$

$\rho()的参数是二进制序列，值是从左向右第一次出现1的位置$

$\therefore 元素个数_1=2^{M_1},元素个数_2=2^{M_2}...$

调和平均数：$$\frac{m}{\frac{1}{2^{M_1}}+\frac{1}{2^{M_2}}+...\frac{1}{2^{M_m}}}=Z$$
整体元素个数$$m\times Z\times \alpha_m$$


