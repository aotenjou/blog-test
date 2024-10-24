---
title: 大创java实战
date: 2024-10-23 19:28:12
tags:
- 课外
- Javaweb
- 收纳分布式项目
---

 <!-- more -->

## 一、初段思路

- 基本的crud
- 实现房间-容器-物品的*一对多*映射
- 使用Jimmer ORM 实现数据库操作
- mysql作为用户数据库
- mvc设计模式
- !目前不考虑优化

### 实现

**技术路径**:

#### 0.配置Gradle中的配置
主要是spring框架,jimmer orm的配置

#### 1.MySQL语言生成表->

![1](https://pic.imgdb.cn/item/6719a364d29ded1a8c6a50a8.png)
这里bot_dev就是使用的db名

#### 2.JimmerGenerate生成实体Entity->

![2](https://pic.imgdb.cn/item/6719a3bed29ded1a8c6ad83a.png)
IDEA使用JimmerGenerate插件.
![3](https://pic.imgdb.cn/item/6719a418d29ded1a8c6b2de6.png)
调整路径\模块\源代码包.

#### 3.手写dto结构->

与model下的Entity匹配,可以包含外键写法
![4](https://pic.imgdb.cn/item/6719a47ed29ded1a8c6b97a6.png)
点击*build*,生成dto文件.
实体模型基本生成完成.

#### 4.实现service和serviceImpl(业务逻辑)->

![5](https://pic.imgdb.cn/item/6719a76cd29ded1a8c6dc54b.png)
service层:基本的crud接口
serviceImpl层:使用jSqlClient:jimmer的数据库客户端接口,直接用.xxx代表sql语言操作

#### 5.实现controller->

记得加注解@RequestMapping: 将 HTTP 请求映射到具体的控制器方法上。
@RestController:标识一个类为控制器，并默认将所有方法的返回值直接写入 HTTP 响应体中。

![6](https://pic.imgdb.cn/item/6718f0a3d29ded1a8cee7e1e.png)

### 后端基本生成完毕,接下来是连接数据库的配置

![7](https://pic.imgdb.cn/item/6719a9d8d29ded1a8c6f7b5a.png)

在`resource/application.yml`中配置.
本地测试时加`-dev`.

我是在docker中使用MySQL.在cmd中使用docker命令pull个8.0.36的img,在docker-compose中启动.
在IDEA右侧的*数据库*打开数据库配置,测试与MySQL的连接.

![8](https://pic.imgdb.cn/item/6719d381d29ded1a8c9d360b.png)

### 全部连接完成后,点击运行BackendApplication.main,使用*PostMan*测试接口.

![9](https://pic.imgdb.cn/item/6719d5b3d29ded1a8c9fd7a4.png)


## Tips
纠错:Alt+Enter 可以快速生成解决办法

jimmer语句自动生成sql操作语句.

#### ID:在 Jimmer 框架下，id 属性代表着一个实体类在数据库中的唯一标识符，它通常对应数据库表中的主键列在 Jimmer 框架下，id 属性代表着一个实体类在数据库中的唯一标识符，它通常对应数据库表中的主键列

jimmer框架下,id通常不可变,代表唯一一个数据实体.
同时,id 属性是实体之间建立关联关系的关键，通过 id 属性，Jimmid 属性是实体之间建立关联关系的关键，通过 id 属性，Jimmer 可以识别出不同实体之间的关联关系，并进行相应的操作er 可以识别出不同实体之间的关联关系，并进行相应的操作

关联: 一对一,一对多,多对多.

**生成**:

- 自动生成:jimmer自动生成,通常为Long类型

- 手动生成:UUID或在实体类使用注解@Id

在 Jimmer 框架中，每个实体类都会生成一个静态实例，这个实例包含了该实体类所有属性的静态访问器,如.$: 通常用于访问与实体类关联的静态实例或方法,通过id实现访问。
