---
title: AI漫游
date: 2025-03-16 16:39:45
tags:
- 论文
---
 <!-- more -->

：今天本想写一天代码，上午实验课后睡醒午觉之后很虚，看点论文打发时间。

# Attention is all you need！

#### LSTM：（长短期记忆网络）

**一种特殊的循环神经网络（RNN），**用于处理序列数据。解决了传统 RNN 的梯度消失和长期依赖问题。
整体框架是RNN.

- LSTM 的核心结构

LSTM 通过门控机制控制信息的流动，包含以下关键组件：

**Ct(conveyor Belt)**:传输带记为**向量Ct**，过去的信息通过传送带送到下个时刻，不发生太大变化。以此避免梯度消失问题。

![](https://pic1.imgdb.cn/item/67d6c2bc88c538a9b5bf42d8.png)

1. ​遗忘门（Forget Gate）​：
    - 决定哪些信息从细胞状态中丢弃。
    - 公式：$f_t​=σ(W_​f\cdot [h_{t−1}​,x_t​]+b_f​)$

![](https://pic1.imgdb.cn/item/67d6bc0888c538a9b5bf4166.png)

参数：
- $f_t$ 遗忘门输出，取值范围为 [0,1]，表示对前一时刻细胞状态 $C_{t−1}$​ 的遗忘程度。
- sigmoid函数：$\sigma (x)=\frac{1}{1+e^{-x}}$。输入实数映射到(0,1),生成S型曲线。主要用在遗忘门中，决定信息**遗忘/保留**程度，0完全遗忘，1完全保留。
- $W_f:$ 遗忘门的权重矩阵，用于对输入进行线性变换。
- $[h_{t-1},x_t]$将前一时刻的隐藏状态$h_{t−1}​$和当前时刻的输入$x_t​$拼接成一个向量。
- $b_f$:遗忘门偏置项。
两部分组成：1.sigmoid函数输入。2.两矩阵乘积。
输出$f_t$，有选择的让传输带C的值通过，元素值即为通过比例。

2. 记忆/输入门（Input Gate）​：
    - 决定哪些新信息存储到细胞状态中。
    - 公式：$i_t​=σ(W_i​\cdot [h_{t−1},x_t​]+b_i​)$,
    - and $\tilde{C}_t​=tanh(W_C​\cdot [h_{t−1}​,x_t​]+b_C​)$

![](https://pic1.imgdb.cn/item/67d6c1e888c538a9b5bf42ab.png)

新参数$W_i$：参数矩阵,需要通过反向传播从训练数据中学习。


3. ​更新（new value）：
    - 更新细胞状态 $C_t$​
    - 公式：$ C_t​=f_t​\cdotC_{t−1}​+i_t​\cdot \tilde{C}_t​$
![](https://pic1.imgdb.cn/item/67d6c29f88c538a9b5bf42d1.png)


4. 输出门（Output Gate）​：
    - 决定输出哪些信息到隐藏状态 $h_t$​。
    - 公式：$o_t​=σ(W_o​\cdot[h_{t−1}​,x_t​]+b_o​)$,$h_t​=o_t​\cdot tanh(C_t​)$

![](https://pic1.imgdb.cn/item/67d6c45688c538a9b5bf4314.png)

**变体：**
- BiLSTM：双向 LSTM，同时考虑过去和未来的信息。
​- GRU（Gated Recurrent Unit）​：简化版 LSTM，只有两个门（更新门和重置门）。














