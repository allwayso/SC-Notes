---
created: 2026-03-12 09:26
updated: 2026-03-12T09:55
status: Completed
topics: Abstract Data Types;Recursive;Immutable;Paser Generator
---

>[!SUMMARY] Table of Contents
>    - [[ProblemSet3 Expressivo#Description|Description]]
>        - [[ProblemSet3 Expressivo#Problem 1: Representing Expressions|Problem 1: Representing Expressions]]
>        - [[ProblemSet3 Expressivo#Problem 2: Parsing Expressions|Problem 2: Parsing Expressions]]
>        - [[ProblemSet3 Expressivo#Problem 3: Differentiation & Problem 4: Simplification|Problem 3: Differentiation & Problem 4: Simplification]]
>    - [[ProblemSet3 Expressivo#Summary|Summary]]
## Description

问题集3和2一样都是抽象数据类型，2的两个实现体现的是可变数据类型和不可变数据类型表达的区别，并且针对的是泛型；3的实现是抽象类的不同变体，即递归数据类型，并且通过Antlr自动生成解析器，变体中只需要实现基本构造函数并重写三个判等方法，接口中只需要简单调用解析器即可

### Problem 1: Representing Expressions

定义一个不可变、递归的抽象数据类型Expression，与ps2相比，这里对各个variant没有规定顶层spec，把更多自由度交给实现者。比较需要注意的是，这里要求一个不可变数据类型，所以各个变体的rep都应当使用private final约束；应该写出抽象数据类型定义。

本问题中只要求实现接口规定的几个基本方法即可。

### Problem 2: Parsing Expressions

解析表达式的第一步是修改.g4文件，符合问题1中的抽象数据类型定义，使其生成lexer，parser和listener；第二步是参考之前[[Reading18 Parser Generator]]中的main函数，调用Antlr生成的解析器，实现parser静态方法；此时可以通过console和main函数交互，所以第三步就是通过控制台调试parser方法

### Problem 3: Differentiation & Problem 4: Simplification

此前接口类都是预定义好的，本问题由实现者自由定义接口方法，即微分和化简。实现没有什么难的，递归即可。

## Summary

本章主要练习利用Antlr工具实现一个递归抽象数据类型，并把更多自由度交给实现者，应当是为最后完全自定义数据类型实现功能做准备。本次练习中我只负责审查，test suit和spec都交由ai实现，尝试vibe coding。
