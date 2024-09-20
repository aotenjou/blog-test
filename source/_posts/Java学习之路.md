---
title: Java学习之路
date: 2024-09-19 15:52:06
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
	\src
		\main #实际项目资源
			\java #java源代码目录
			\resources #配置文件目录
		\test #测试项目文件
			\java
			\resources
		\pom.xml #项目配置文件
```

- maven在"清理->编译->测试->打包->发布"五个阶段均能提供帮助.
- 打包文件储存在/target目录下.

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

# 从SpringBootWeb开始
### Spring:[开发生态圈](spring.io)
### spring boot快速入门:
##### 需求:浏览器发出请求/hello,返回字符串"Hello World~"
##### 步骤:
- 1.springboot工程(spring initializer),勾选**web开发**相关依赖.
- 2.定义 HelloController 类,添加方法hello,添加**注解**(@RestController,@RequestMapping)
- 3.运行启动类,测试.

# HTTP协议
- 基于TCP
- 一次请求一次响应
- 无状态协议

### 请求数据格式
**请求行**:
- 请求数据第一行
- 请求方式(GET POST)请求路径,协议.
**请求头**:
- 第二行开始
- key: value

    HOST 请求主机名
    User-Agent 浏览器版本
    Accept 浏览器能接受的资源类型 */*所有
    Accept-Language
    Accept-Encoding
    Content-Type
    Content-Length

**请求体**
- POST请求(GET无请求体)
- POST大小无限制

### 响应数据格式
**响应行**:
- 响应数据第一行
- 协议
- 状态
  
    1xx 响应中
    2xx 成功
    3xx redirect
    4xx client error
    5xx server error

- 描述
**响应头**
- key:value

    content type  html/text/json
    content length 长度
    content encoding 压缩算法
    cache control 如何缓存
    set cookie 设置cookie
**响应体**
- 响应正文:json数据

### 协议解析

# Tomcat
web应用服务器

- javaSE:标准版
- javaME:小型版
- javaEE:企业版, 13项技术规范.

实操了一下,懒得记录了.
常见的启动失败:
- 端口冲突
- 缺少jdk
![](https://pic.imgdb.cn/item/66ebe205f21886ccc0c31137.png)

# 入门程序解析

browser$\leftrightarrow$DispatcherServlet(前端控制器),把所有请求解析并封装到HTTPservletRequest,响应Response$\leftrightarrow$Controller

### 请求
##### postman(用apifox代替了)
接口测试

#### 常见参数
##### 简单参数
###### 1.原始方式获得请求参数
- Controller方法中形参声明HttpServletRequest变量
- 调用对象的getParameter
###### 2.接收简单参数
- **请求参数名**和**方法形参变量名**相同
- 自动进行类型转换
###### 3.@RequestParam注解
- 方法**形参**和**请求参数**名称不匹配,用该注解完成映射.
- 注解**required**默认为**true**,代表参数必须传递.


##### 实体参数
- 实体对象参数:**请求参数名**和**对象属性名**,相同,即可直接通过POJO接收.

##### 数组\集合参数
- 数组:**请求参数名**和**形参中数组变量名**相同,可以直接使用数组封装.
- 集合:...,通过@RequestParam绑定参数关系.

##### 日期参数
- @DateTimeFormat完成日期参数格式转换.(pattern="")指定格式.
##### JSON参数
- Body->raw
- 使用 @RequestBody 注解将请求体中的 JSON 数据映射到 Java 对象.

##### 路径参数
- 使用 @PathVariable 注解获取路径参数,在@RequestMapping中也要标识id{多个用"/"分割}

##### P.S.
- @Valid :指示 Spring 框架对方法参数或对象进行验证，确保传入的数据符合预期的格式和约束。
  
  @Valid 注解会触发 Spring 的数据验证机制，对被注解的类或方法参数进行验证。验证规则由你定义的校验注解（如 @NotNull、@NotBlank、@Size 等）来指定。
  如果验证失败，Spring 会抛出 MethodArgumentNotValidException 异常。你可以使用 @ExceptionHandler 注解处理该异常，并返回相应的错误信息给客户端。
  验证逻辑和业务逻辑分离.

- @RequestMapping :用于将 HTTP 请求映射到特定的控制器方法。简单来说，它告诉 Spring 框架，当接收到某个特定类型的 HTTP 请求时，应该执行哪个控制器方法。

### 响应
@ResponseBody
- Controller方法
- 将**方法返回值**直接响应.如果是实体对象/集合,则返回JSON.
- @RestController=@Controller+@ResponseBody

#### 统一响应结果

    public class Result
    {
        private bool code;
        //响应,1成功,0失败

        private string msg;
        //提示信息

        private object data;
        //返回数据
    }

# 分层解耦
### 三层架构
