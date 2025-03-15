---
title: Auto_Test
date: 2025-03-14 22:38:51
tags:
- 课外
---
 <!-- more -->

# 自动化测试-JUnit

- 黑盒：提供input,output。查看代码结果是否达到要求。不需要写代码。
- 白盒：提供input,output。关注程序执行具体过程。需要写代码。（JUnit）

### 白盒测试：
1. 定义测试类（测试用例）
    1. 测试类名：被测试类名Test
    2. 包名：.test

2. 定义测试方法：可以独立运行
    1. 方法名：test测试方法名
    2. 返回值：void
    3. 参数列表：空参

3. 给方法加@Test
4. 导入junit依赖

### 初始化方法
资源申请，测试方法执行前都会先执行该方法。
`@Before`

### 释放资源方法
释放资源方法，所有测试方法执行完后，都会自动执行该方法。
`@After`


判定结果：断言

    Assert.assertEquals(expection,result);

### Fixture

类似RAII，提供编写测试前准备、测试后清理的固定代码。
`@BeforeEach``@AfterEach`环绕在每个`@Test`前后。
`@BeforeAll``@AfterALl`在所有`@Test`方法前后仅运行一次，智能初始化静态变量。

`assertThrows()`来期望捕获一个指定的异常。new Executable()封装了我们要执行的会产生异常的代码。

### 条件测试

`@Disable`阻止一个`@Test`运行。
`@EnabledOnOs()，@EnableOnJre()`等条件测试判断。

### 参数化测试

`@ParameterizedTest`替换掉`@Test`
`@MethodSource`编写一个同名静态方法，传入测试参数。
另一种使用`@CsvSource`每个字符串代表一行，用“，”分隔。测试量大的时候使用独立csv文件。标注`@CsvFileSource`








