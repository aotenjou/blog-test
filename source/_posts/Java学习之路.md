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

    maven-project
        \src
            \main #实际项目资源
                \java #java源代码目录
                \resources #配置文件目录
            \test #测试项目文件
                \java
                \resources
            \pom.xml #项目配置文件


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

- Controller: 控制层,接收/处理请求,响应数据.
- Service: 业务逻辑层.
- DAO: 数据访问层(持久层),数据增删改查.面向接口编程.

### 解耦

- 内聚:软件各个功能费内部的功能联系.
- 耦合:衡量软件中各个层/模块之间的依赖关联程度.
- 好标准:**高内聚,低耦合**.
- Service层和DAO层的实现类,交给IOC容器管理.
- 为Controller及Service注入运行时依赖的对象.

### IOC:控制反转

- 对象创建控制权由自身转移到外部容器.

### DI:依赖注入

- 容器为应用程序提供运行时所依赖的资源.
- @Autowired按照类型自动装配

    如果同类型bean存在多个
        @Primary
        @Autowired+@Qualifier("bean的名称")
        @Resource(name="bean的名称")
- @Resource和@Autowired的区别
    @A是spring框架提供的注解,而@R是JDK提供的注解.
    @A默认按类型注入,@R默认按名称注入.

### Bean对象

- IOC容器中创建管理的对象.
- 要把对象交给IOC管理,需要在对应的类上加上注解.
- 声明bean的时候,可以通过value指定bean名,如果没有指定,默认首字母小写.
- spring boot集成web开发中,声明bean只能用@Controller

##### Bean组件扫描

- 声明Bean的四大注解,想要生效,还需要被注解扫描@ComponentScan扫描
- @ComponentScan虽然没有显示配置,但实际上已经包含了在启动类声明注解@SpringBootApplication中,默认扫描范围是启动类所在包及其子包.

***

# 基础规范(以cpp学习为基础的不同)

## 继承

- 只支持单继承.不能有多个父类.
- 弊端: 高耦合,打破封装
- **同名**成员变量查找:子类局部范围->子类成员范围->父类成员范围(不考虑父亲的父亲).

### super

与**this**很像,理解为指向最近的父类的指针.

- this:本类引用.
- super:父类引用.

**用法**:

#### 访问成员变量

super.变量名

#### 访问构造/成员方法

- 访问成员方法:super.方法名(参数列表)
- 访问构造方法: super(参数列表)

### 抽象类

不能与private,final,static共存

- 与接口的关系
抽象类:is a,共性
接口:like a,扩展

#### 向上转型

将一个*子类类型的对象*赋值给一个*父类类型的变量*

    Animal animal = new Dog(); 

多态性的基础.

- 隐式转换： 向上转型是 Java 中的一种隐式转换，不需要显式地进行类型转换。
- 安全性： 向上转型是安全的，因为子类对象总是包含父类定义的所有属性和方法。
- 信息丢失： 向上转型会导致信息丢失，因为父类类型的变量只能访问子类对象中继承自父类的成员，而无法访问子类特有的成员。

它可以引用任何 Animal 类或其子类的对象

#### 向下转型

对应的,是*父类*转*子类*.

- 显式转换： 向下转型需要进行显式类型转换，使用 (子类名) 父类对象 的语法。
- 风险性： 向下转型是存在风险的，因为编译器无法保证父类类型的变量一定引用了子类类型的对象。如果父类变量实际引用的是其他子类对象或父类对象，进行向下转型会导致 ClassCastException 异常。
- 访问子类成员： 向下转型是为了访问子类特有的成员。

使用 instanceof 运算符来检查父类变量实际引用的对象类型.

## 多态成员访问特点.

### 形式参数如果是引用类型：

- 具体类：该类的对象。
- 抽象类：该类的子类对象。
- 接口：该接口的实现类对象。
- 数组：数组的地址值。其实就是一个数组对象。


### 成员变量

- 编译左,运行左

### 成员方法

- 编译左,运行右
在编译阶段，编译器会根据*引用变量的类型（左边）(类及其子类)*来检查调用的方法是否存在。
在运行阶段，虚拟机会根据实际对象的类型（右边）来确定调用哪个方法。

### 静态方法

- 编译左,运行左.

### 链式编程

    class Person {
        private String name;
        private int age;
        private String address;

        public Person setName(String name) {
            this.name = name;
            return this; // 返回当前对象
        }

        public Person setAge(int age) {
            this.age = age;
            return this; // 返回当前对象
        }

        public Person setAddress(String address) {
            this.address = address;
            return this; // 返回当前对象
        }

        @Override
        public String toString() {
            return "Person [name=" + name + ", age=" + age + ", address=" + address + "]";
        }
    }

实例化

    Person person = new Person()
                    .setName("Alice")
                    .setAge(30)
                    .setAddress("123 Main St");

注意*降低耦合度*.

## 包

本质是文件夹.
package语句必须是程序的第一条可执行代码,在Java文件中只能有一个.

### 带包的类的编译和运行

#### 手动

- a.javac编译当前类
- b.手动建立文件夹
- c.把a步骤class文件放到b步骤的最终文件夹下.
- d.通过Java命令执行,需要带包名称

#### 自动

- javac编译带-d即可.

### 不同包下*类之间*的访问

加上包名前缀

#### 导包

import.
颗粒度到类的名称.

##### package->import->class的顺序关系

package --> import --> class
唯一         多个       多个

你说的没错！表格的变量应该是 **作用范围** 而不是 **变量**。 

以下是修正后的表格：

## Java 权限修饰符作用范围

| 修饰符 | 同类 | 同包子类&其他类 | 不同包子类 | 不同包其他类 |
|---|---|---|---|---|
| `public` | √ | √ | √ | √ |
| `protected` | √ | √ | √ |  |
| `default` (无修饰符) | √ | √ |  |   |
| `private` | √ |  |  |  |

## 内部类

可以直接访问外部类成员,包括private
外部类要访问内部类,则必须创建对象

### 位置

- 成员内部类
- 局部内部类

#### 成员内部类

实际开发不用.

