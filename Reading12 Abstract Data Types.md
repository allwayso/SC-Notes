---
created: 2026-02-25 15:50
updated: 2026-02-26T00:16
status: Completed
topics: Abstract Data Types;Representation independence
---
>[!SUMMARY] Table of Contents
>    - [[Reading12 Abstract Data Types#What Abstraction Means|What Abstraction Means]]
>        - [[Reading12 Abstract Data Types#User-Defined Types|User-Defined Types]]
>    - [[Reading12 Abstract Data Types#Classifying Types and Operations|Classifying Types and Operations]]
>    - [[Reading12 Abstract Data Types#Designing an Abstract Type|Designing an Abstract Type]]
>    - [[Reading12 Abstract Data Types#Representation Independence|Representation Independence]]
>    - [[Reading12 Abstract Data Types#Summary|Summary]]
## What Abstraction Means

抽象数据类型——或者抽象，是一个通用的原则，有许多表述这个原则的名词，含义稍有分别：
- **Abstraction.** 隐藏低层级细节的高阶概念
- **Modularity.** 将系统分为几个互相独立的模块，每个模块都是独立设计、实现、测试的
- **Encapsulation.** 在模块周围建立屏障，私有数据和方法不对外开放，从而使外部bug不会侵害模块
- **Information hiding.** 隐藏模块内部的实现细节
- **Separation of concerns.** 把功能放在单一模块中

### User-Defined Types

在计算机发展的早期，编程语言内置类型，用户可以将内置类型组合成自定义的数据类型，比如cpp中的struct，但是它与抽象数据类型有本质上的区别：
1. ADT由操作定义，而UDT仍然围绕数据
2. ADT可以任意改变数据类型，而UDT一般只能接受一种参数，除非进行重载
3. ADT的可见性控制更为绝对，任何实现都不会出现中，UDT虽然可能进行访问控制，但是用户还是可以看到实现逻辑

## Classifying Types and Operations

不管是内置类型、用户定义类型还是抽象数据类型，都可以分为可变数据类型和不可变数据类型

ADT操作可以分为以下几种：
1. 创建：构造函数或者使用工厂函数
2. 生产：接受旧对象并创造新对象
3. 观察：接受对象并返回不同类型的返回值
4. 修改：改变对象

这里阐述一下构造函数和工厂函数的区别：
1. 重载：如果希望根据不同类型或结构的输入进行不同类型的构造过程，构造函数只能参数重载，工厂函数可以自由命名
2. 返回值：构造函数返回一个对象实例，工厂函数返回抽象类
3. 可见性：构造函数的实现无法隐藏，而工厂函数内部的构造方法可以与接口分离

当然这个操作分类并不完备，有的操作可能兼顾观察和修改，只是一个大略的分类

## Designing an Abstract Type

设计一个抽象类的时候有一些规则：
1. 简单：单个操作应当简单，使其更加灵活
2. 完整：操作应该足够使使用者实现各种复杂操作
3. 普适：操作应该适用于所有可能实现这个抽象类的类型，比如List类型不应该拥有sum方法，因为非整数列表无法兼容
4. 明确：操作应该拥有明确的目的
5. 纯粹：通用领域的抽象类不应该包含特定领域的操作，反之亦然，因为这是逻辑混乱且不安全的

## Representation Independence

实现独立性的意思是：抽象类型应该与其表示（具体实现）无关，即改变表示不应该使原本使用该操作的用户遇到bug

## Summary

这里介绍了抽象数据类型这个概念，基本上讨论的是广义的ADT，不仅仅局限于java。对于java的ADT，可以参考[[Slide6 Design,Debugging,Interface#Interfaces]]，即interface。广义的ADT是一个由操作定义的数据类型，核心特点是实现独立性，即其表现独立于抽象类型。

