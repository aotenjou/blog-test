---

title: Java学习之路

date: 2024-09-14 14:33:31

tags: 
- 'Javaweb'
- '课外'
- '收纳分布式项目'
---

 <!-- more -->

# maven依赖

### maven主要作用

**依赖管理(jar),统一项目结构和项目构建**
- 用maven工程管理jar包:修改pom.xml,自动下载dependencies.

- 不同开发工具Java项目的目录结构不同.在maven下则能创建统一的目录结构.
```
maven-project
	src
		main #实际项目资源
			java #java源代码目录
			resources #配置文件目录
		test #测试项目文件
			java
			resources
		pom.xml #项目配置文件
```

- maven在"清理->编译->测试->打包->发布"五个阶段均能提供帮助.

# maven概述

### pom:对象模型(project object model)

maven模型
![](https://pic.imgdb.cn/item/66e55bc6d9c307b7e9eff138.png)
##### maven坐标:唯一的标识定位项目
1. **groupId (组织ID)**：通常是你的组织的**域名反转**，例如 `com.example`。
2. **artifactId (项目 ID)**：项目的名称，例如 `my-project`。
3. **version (版本号)**：项目的版本，例如 `1.0.0`。

### 仓库
- 本地仓库:计算机上的一个目录
- 中央仓库:全球唯一[maven团队维护仓库](repo1.maven.org)
- 远程仓库(私服):一般有公司仓库搭建.
- **顺序**:本地仓库->远程仓库->中央仓库
