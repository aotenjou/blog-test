---
title: JavaTips
date: 2024-09-22 10:51:47
tags:
- 'Javaweb'
- '课外'
---
 <!-- more -->
# Java中*对象*和*实例*没有区别

# Java中的*方法*与cpp中的*函数*

| 特性        | Java                                             | C++                                             |
|-------------|-------------------------------------------------|-------------------------------------------------|
| 所属关系     | 类或接口                                         | 全局或类                                         |
| 传参方式     | 值传递 (对象类型传递引用)                       | 值传递、指针传递、引用传递                       |
| 重载        | 支持                                             | 支持 (更灵活)                                     |
| 虚函数/抽象方法 | 抽象方法 (`abstract`)，接口 (`interface`) | 虚函数 (`virtual`)，纯虚函数 (`= 0`)，抽象类 |
| 访问控制     | `public`, `protected`, `private`, 默认     | `public`, `protected`, `private`             |
| 异常处理     | `try-catch-finally`，受检异常                 | `try-catch`                                     |
| 内联函数     | 编译器决定                                         | `inline` 关键字建议                             |
| 函数指针     | 不支持                                             | 支持                                             |

# 静态方法和实例方法

静态方法存在于类级别，并且不依赖于任何特定对象，因此它无法直接访问或使用需要对象实例才能访问的实例方法和实例变量。

# 方法重载

参数**个数**或者参数**类型**不同即可.

# new关键字

### Java

- **分配内存**： 当使用 new 创建对象时，Java 虚拟机 (JVM) 会在堆内存中为该对象分配一块足够大的内存空间。
- **调用构造函数**： new 关键字后面跟着要创建对象的类的构造函数。这会调用该类的构造函数，并使用提供的参数初始化新创建的对象。

- 在 Java 中，所有对象都必须使用 `new` 关键字创建。
- `new` 关键字返回一个指向新创建对象的引用。
- 如果没有为类定义构造函数，Java 编译器会提供一个默认的无参构造函数。

## Java 与 C++ 中 new 的比较

| 特性          | Java                                 | C++                                     |
|-----------------|--------------------------------------|------------------------------------------|
| 内存分配       | JVM 在堆内存中自动分配和管理内存。  | 程序员需要手动分配和释放内存。             |
| 垃圾回收       | JVM 自动执行垃圾回收。              | 程序员需要手动释放不再使用的内存，否则会导致内存泄漏。 |
| 对象创建       | 只能使用 `new` 关键字创建对象。      | 可以使用 `new` 关键字或直接声明变量创建对象。 |
| 内存位置       | 对象存储在堆内存中。               | 对象可以存储在堆内存或栈内存中。         |
| 返回值          | `new` 返回指向对象的引用。           | `new` 返回指向对象的指针。               |
| 删除对象       | 不需要手动删除对象，由垃圾回收器负责。 | 需要使用 `delete` 运算符手动释放使用 `new` 分配的内存。 |

# 数组初始化

作为**类成员变量**声明: 如果数组是作为类的成员变量声明的，那么它会默认初始化为 0 或对应的默认值 (对于基本数据类型)，或 null (对于对象类型)。 这是因为 Java 会保证类的成员变量在对象创建时拥有一个初始值。
在**方法内部或其他局部代码块**中声明: 如果数组是在方法内部或其他局部代码块中声明的，那么它不会被自动初始化。 你需要在使用数组元素之前手动为其赋值，否则会遇到 *NullPointerException* (对于对象类型数组) 或编译错误 (对于基本数据类型数组)。

# 局部变量与成员变量

| 特性 | 局部变量 | 成员变量 |
|---|---|---|
| **声明位置** | 方法内部、构造函数内部、代码块内部 | 类内部，但不在任何方法、构造函数或代码块内部 |
| **作用域** | 声明该变量的代码块内部 | 整个类 |
| **生命周期** | 从声明开始到代码块执行结束 | 从对象创建开始到对象被销毁 |
| **默认值** | 没有默认值，必须显式初始化才能使用 |  拥有默认值，具体取决于数据类型：<br> - 数值类型：0<br> - 布尔类型：false<br> - 对象类型：null |
| **访问修饰符** | 不能使用访问修饰符（public、private等） | 可以使用访问修饰符来控制访问权限 |
| **存储位置** | 存储在栈内存中 | 存储在堆内存中（作为对象的一部分） |

# 匿名类与匿名对象

- 意义:只使用一次的"工具",没有名字的类与对象
使用方法:

    class outerClass {

    // 定义一个匿名类
    object1 = new Type(parameterList) {
         // 匿名类代码
    };
    }
- 以上的代码创建了一个匿名类对象 object1，匿名类是表达式形式定义的，所以末尾以分号 ; 来结束。
- 匿名类通常继承一个父类或实现一个接口。
- 实现之后即被gc

### 接口, default实现

- 接口定义:Interface.其实和class很像,不过不在interface里实现,而是在下面开一个implement来实现.
成员变量只能是常量
成员方法只能是抽象方法
- default**默认方法**:在接口中使用 default 关键字：
直接继承并使用接口提供的默认方法实现。
*选择性*地**覆盖 (Override)** 默认方法，提供自己的实现。
- 类与接口是实现关系,可以多实现,或者在继承一个类的时候实现多个接口.
- 接口与接口可以多继承

# 引用类型

## *指向内存地址的数据类型*

- 类 (Class): 类是创建对象的模板，每个类都定义了一组属性和方法。
- 接口 (Interface): 接口定义了一组方法的签名，但不提供实现。
- 数组 (Array): 数组是存储相同数据类型元素的固定大小的容器。
- 枚举 (Enum): 枚举是一组命名的常量。
- 注解 (Annotation): 注解是用于为代码添加元数据的特殊标记。

### 引用类型的特点

- 传递: 当将引用类型变量作为参数传递给方法时，传递的是**对象的引用**，而不是对象的副本。
- 共享: 多个引用类型变量可以指向同一个对象，因此**对一个引用变量的修改会影响到其他指向相同对象的引用变量**。
- null值: 引用类型变量可以赋值为 null，表示该变量不指向任何对象.


