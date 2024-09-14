---
title: Java学习之路2
date: 2024-09-14 22:44:53
tags:
- 'Javaweb'
- '课外'
- '收纳分布式项目'
---
# 依赖管理
### 依赖:当前项目运行所需jar包
 - 使用<dependencies>标签,其中用<dependency>引入坐标
 - 如果不知道坐标信息,在[仓库](https://mvnrepository.com/)搜索

# 依赖传递
 - 若A(project)依赖B,B依赖C,则A依赖C
 - 可视化查看:diagram -> show dependencies
### 排除依赖:主动断开依赖的资源
 - 标签<exclusions>,其中<exclusion>.(与dependency类似)

# 依赖范围
### 依赖的jar包,默认可以在任何地方使用.可以用<scope>标签限制其使用范围.
##### 作用范围
  - 主程序 main
  - 测试程序 test
  - 打包 package
##### scope取值范围(验证类或接口)
- compile(默认):在main,test,package均有效
- test:仅在test有效
- provided:在package无效
- runtime:在主程序无效 

# 生命周期
### 三套相互独立的生命周期:
- clean:清理工作.
- default:核心工作:编译,测试,打包,安装,部署.
- site:生成报告,发布站点.

### 每套生命周期包含一些阶段(phase)
clean -> compile -> test -> package -> install
