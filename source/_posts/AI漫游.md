---
title: AI漫游
date: 2025-03-16 16:39:45
tags:
- 论文
---
 <!-- more -->

：今天本想写一天代码，上午实验课后睡醒午觉之后很虚，看点论文打发时间。

# Attention is all you need！

### 从神经网络开始

- ​神经元：模拟生物神经元，接收输入，计算加权和，通过激活函数输出。**本质：无限拟合函数。**
- ​层：
    - ​输入层：接收数据。
    - ​隐藏层：提取特征。
    - ​输出层：输出结果。
- 激活函数：某种映射或过滤。引入非线性（ReLU（负数->0,正数不变）、Sigmoid（见LSTM）、Tanh（双曲正切，压缩到[-1,1]））。
- ​损失函数：衡量预测值与真实值的差距（MSE（方差）、Cross-Entropy（交叉熵）、MAE（平均极差）、​Hinge Loss（铰链损失）​、Kullback-Leibler 散度）。
- ​优化器：更新参数以最小化损失（如 SGD、Adam）。

#### CNN 卷积
- 卷积层：
    - 使用卷积核（filter）提取局部特征。
    - 每个卷积核学习不同的特征（如边缘、纹理）。
- ​池化层：
    - 下采样，降低数据维度（如最大池化、平均池化）。
- ​全连接层：
    - 将特征图展平，用于分类或回归。

#### RNN 循环
- 循环结构：
    - 每个时间步接收当前输入和前一时刻的隐藏状态。
    - 公式：$h_t​=f(W_h​\cdot h_{t−1}​+W_x​\cdot x_t​+b)$
- 隐藏状态：
    - 捕捉序列中的时间依赖关系。
- ​输出层：
    - 根据任务需求，输出分类、回归或序列。





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
    - 以及：$\tilde{C}_t​=tanh(W_C​\cdot [h_{t−1}​,x_t​]+b_C​)$

![](https://pic1.imgdb.cn/item/67d6c1e888c538a9b5bf42ab.png)

新参数$W_i$：参数矩阵,需要通过反向传播从训练数据中学习。


3. ​更新（new value）：
    - 更新细胞状态 $C_t$​
    - 公式：$C_t​=f_t​\cdotC_{t−1}​+i_t​\cdot \tilde{C}_t​$
![](https://pic1.imgdb.cn/item/67d6c29f88c538a9b5bf42d1.png)


4. 输出门（Output Gate）​：
    - 决定输出哪些信息到隐藏状态 $h_t$​。
    - 公式：$o_t​=σ(W_o​\cdot[h_{t−1}​,x_t​]+b_o​)$,$h_t​=o_t​\cdot tanh(C_t​)$

![](https://pic1.imgdb.cn/item/67d6c45688c538a9b5bf4314.png)

**变体：**
- BiLSTM：双向 LSTM，同时考虑过去和未来的信息。
​
- GRU（Gated Recurrent Unit）​：简化版 LSTM，只有两个门（更新门和重置门）。

### 注意力机制

![](https://pic1.imgdb.cn/item/67d7d16688c538a9b5bfe337.png)

主要两个过程：

1. 根据Query和Key计算权重系数。

2. 根据权重系数对Value进行加权求和。

![](https://pic1.imgdb.cn/item/67d7d2a488c538a9b5bfe369.png)

说烂了，不赘述。

#### Self-attention自注意力机制

减少了对外部信息的依赖，更擅长捕捉数据或特征的内部相关性。自注意力机制在文本中的应用，主要是通过计算单词间的互相影响，来解决长距离依赖问题。

自注意力机制的计算过程：

1. 将输入单词转化成嵌入向量；

2. 根据嵌入向量得到q，k，v三个向量；

3. 为每个向量计算一个score：$score =q\cdot k$ ；

4. 为了梯度的稳定，Transformer使用了score归一化，即除以$\sqrt{dk}$

5. 对score施以softmax激活函数；

6. softmax点乘Value值v，得到加权的每个输入向量的评分v；

7. 相加之后得到最终的输出结果z ：$z= \sum v$。






