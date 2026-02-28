---
created: 2026-02-09 12:12
updated: 2026-02-28T21:33
status: Completed
topics: Static typing ; Three big properties of good software
---
>[!SUMMARY] Table of Contents
>    - [[Reading1 Static Check#Hailstone sequeue|Hailstone sequeue]]
>    - [[Reading1 Static Check#Types|Types]]
>        - [[Reading1 Static Check#Surprice：Primitive Types Are Not True Numbers|Surprice：Primitive Types Are Not True Numbers]]
>    - [[Reading1 Static Check#Static Check|Static Check]]
>        - [[Reading1 Static Check#Static Typing|Static Typing]]
>        - [[Reading1 Static Check#Check methods|Check methods]]
>    - [[Reading1 Static Check#Arrays and Collections|Arrays and Collections]]
>        - [[Reading1 Static Check#Iterating|Iterating]]
>    - [[Reading1 Static Check#Methods|Methods]]
>    - [[Reading1 Static Check#Good Programming Style|Good Programming Style]]
>        - [[Reading1 Static Check#Documenting Assumptions|Documenting Assumptions]]
>        - [[Reading1 Static Check#Hacking Vs Engineering|Hacking Vs Engineering]]
>    - [[Reading1 Static Check#Goal of Mit 6.005|Goal of Mit 6.005]]
>    - [[Reading1 Static Check#选择java的原因|选择java的原因]]
## Hailstone sequeue

一个悬而未决的命题：如果一个数为奇数，则对其乘三加一；若其为偶数，则除以二。那么是否对于任意正整数，其最终都要变为1？
这个本章只是作为Java语言的一个引入，提及了一些关于expression，semicolon，parentheses和curly braces有关的规范
冰雹问题将贯穿本文，来介绍java的特性和规范

## Types

Primitive types：int，long，double，char，boolean
Objective types：String，BigInteger
同时也提及了一些有关function和overload的基本知识

### Surprice：Primitive Types Are Not True Numbers

基本类型跟数学意义上的数字有区别，主要体现在：
1. 整数除法：向零取整
2. 整数溢出：超出边界的整数将回绕
3. 特殊值：`NaN` (which stands for “Not a Number”), `POSITIVE_INFINITY` , and `NEGATIVE_INFINITY` 这些值并不对应特定数字，不可用于计算

## Static Check

### Static Typing

Java是一种静态类型(Static typing)的语言，这要求所有变量在编译时就确定类型，因此类型错误会在编写或者编译过程中被发现
静态检查(Static check)是指在编译时发现错误，所以静态类型是静态检查中关于类型检查的部分
Eclipse for java会在编写过程中检查类型错误，如有错误不能进入编译

### Check methods

语言有三种检查方式：静态检查，动态检查（在运行时检查错误）和不检查（允许任何错误产生）

静态检查可以捕获：语法错误，参数类型错误，错误名称，参数数量错误和返回类型错误
动态检查可以捕获：
1. 非法参数：如x/y只有当y为0时产生除0错误，这是一种动态错误
2. 不可表述的返回值：即返回值与返回类型不匹配
3. 越界索引：数组越界等
4. 在空对象上（null）调用方程

静态检查主要关注类型，而动态类型关注值

## Arrays and Collections

提及了java中Array和List的基本操作和概念，主要区别是：数组定长，列表变长；数组是类型，列表是接口；数组接受基本类型，列表只接受对象类型
需要说明的是，java中每个基本类型都有对应的对象类型，如int和Integer，这些类型可以在除了尖括号包裹部分以外自动转换

### Iterating

简单介绍了一下for循环

## Methods

语句必须包含在方法内，方法必须写在类中
简单介绍了一下访问控制、静态类型和编程规范

## Good Programming Style
### Documenting Assumptions

应该写下假设，从而方便自己和他人进行阅读和修改
程序必须以两个目标为导向：
1. 与计算机良好沟通：语法、类型和逻辑正确
2. 与人良好沟通：提高程序的可读性

### Hacking Vs Engineering

Hacking：
1. 写很长的代码而不做测试
2. 把细节记在脑子里而不是写入代码
3. 假设缺陷不存在或者易于修复

Engineering：
1. 边写边测试
2. 记录假设
3. 避免错误

## Goal of Mit 6.005

如introduction中提及的目标,[[MIT6.005 Introduction & Getting Started#Course Description|safe from bugs, easy to understand, and ready for change]]

## 选择java的原因

安全性：java的安全性使其成为学习良好软件工程习惯的合适语言
普适性：或许java并不是最适用于教学的，但是由于其在应用上的广泛性，我们选择了java
多语言：语言只是工具，好的程序员至少要会两种语言
生态完备：内置库和可用库繁多，生态丰富
