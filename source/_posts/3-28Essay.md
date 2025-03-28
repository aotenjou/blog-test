---
title: 3.28Essay
date: 2025-03-28 14:28:06
tags:
- 随记
---
 <!-- more -->

## MCP： Model Context Protocol

#### bg

应用程序和 AI 模型之间交换上下文信息的方式。这使得开发者能够以**一致的方式将各种数据源、工具和功能连接到 AI 模型**（一个中间协议层）。
MCP 就是以更标准的方式让 LLM Chat 使用不同工具。

![](https://pic1.imgdb.cn/item/67e646f10ba3d5a1d7e58acd.png)

*更结构化的上下文信息*

基于**prompt engineering**：tool的结构化描述和example

最早：手工实现prompt协作

后来引入`function call`，允许模型在需要时**掉用于定义函数来获取数据、执行操作**，局限性在于**平台依赖性强**，`function call API`差异大

MCP最大优势：

- 生态：现成插件。
- 统一性：支持很多模型自由切换。
- 数据安全：自行设计接口，不必上传。

#### how 2 use?
[使用doc](https://modelcontextprotocol.io/quickstart/user)

#### 架构

三个核心组件：Host,client,server。

- host：接受用户请求，与模型交互的接口。

- client：模型决定需要访问文件系统时，激活host内置的MCP client.该client与适当MCP server建立连接。

- server：MCP server会被调用。执行实际系统层面操作。

提出问题后的执行步骤

1. 由 LLM 确定使用哪些 MCP Server（**通过prompt确定有哪些工具**）。
2. 执行对应的 MCP Server 并对执行结果进行重新处理。

*工具具体使用描述：文本形式传递。调用：json*


tool的描述与`input schema`基本上由用户自定义。


## Do We Need Zero Training Loss After Achieving Zero Training Error? （ICML，2020）
一行代码一篇a.

损失函数变为：$\tilde{L} (\theta)=|L(\theta)-b|+b$

效果：**二次下降**。损失函数达到b之后，训练流程大概就是在交替执行梯度下降和梯度上升。

相当于一开始就加入梯度惩罚。达到推动参数往更平稳的区域走。







