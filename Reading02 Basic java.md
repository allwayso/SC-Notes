---
created: 2026-02-10 09:24
updated: 2026-02-28T21:33
status: Completed
topics: Learn basic Java syntax and semantics;Transition from writing Python to writing Java(not even know python yet :P)
---
>[!SUMMARY] Table of Contents
>    - [[Reading2 Basic java#Getting started with the Java Tutorials|Getting started with the Java Tutorials]]
>        - [[Reading2 Basic java#Language basics|Language basics]]
>        - [[Reading2 Basic java#Numbers and strings|Numbers and strings]]
>        - [[Reading2 Basic java#Classes and objects|Classes and objects]]
>        - [[Reading2 Basic java#Hello, world!|Hello, world!]]
>    - [[Reading2 Basic java#Snapshot diagrams|Snapshot diagrams]]
>        - [[Reading2 Basic java#Mutating values vs. reassigning variables|Mutating values vs. reassigning variables]]
>    - [[Reading2 Basic java#Java Collections|Java Collections]]
>    - [[Reading2 Basic java#Java API documentation|Java API documentation]]
## Getting started with the Java Tutorials

The next few sections link to the **[Java Tutorials](https://docs.oracle.com/javase/tutorial/index.html)** to get you up to speed with the basics.
You can also look at [Getting Started: Learning Java](https://ocw.mit.edu/ans7870/6/6.005/s16/getting-started/java.html) as an alternative resource.
This reading and other resources will frequently refer you to the [Java API documentation](https://docs.oracle.com/javase/8/docs/api/) which describes all the classes built in to Java.

### Language basics

Read **[Language Basics](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/index.html)** .
You should be able to answer the questions on the _Questions and Exercises_ pages for all four of the langage basics topics.
- [Questions: Variables](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/QandE/questions_variables.html)
- [Questions: Operators](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/QandE/questions_operators.html)
- [Questions: Expressions, Statements, Blocks](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/QandE/questions_expressions.html)
- [Questions: Control Flow](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/QandE/questions_flow.html)
Note that each _Questions and Exercises_ page has a link at the bottom to solutions.

### Numbers and strings

Read **[Numbers and Strings](https://docs.oracle.com/javase/tutorial/java/data/index.html)** .
Don’t worry if you find the `Number` wrapper classes confusing. They are.
You should be able to answer the questions on both _Questions and Exercises_ pages.

- [Questions: Numbers](https://docs.oracle.com/javase/tutorial/java/data/QandE/numbers-questions.html)
- [Questions: Characters, Strings](https://docs.oracle.com/javase/tutorial/java/data/QandE/characters-questions.html)

### Classes and objects

Read **[Classes and Objects](https://docs.oracle.com/javase/tutorial/java/javaOO/index.html)** .
You should be able to answer the questions on the first two _Questions and Exercises_ pages.
- [Questions: Classes](https://docs.oracle.com/javase/tutorial/java/javaOO/QandE/creating-questions.html)
- [Questions: Objects](https://docs.oracle.com/javase/tutorial/java/javaOO/QandE/objects-questions.html)
Don’t worry if you don’t understand everything in _Nested Classes_ and _Enum Types_ right now. You can go back to those constructs later in the semester when we see them in class.

### Hello, world!

Read **[Hello World!](https://docs.oracle.com/javase/tutorial/getStarted/application/index.html)**
You should be able to create a new `HelloWorldApp.java` file, enter the code from that tutorial page, and compile and run the program to see `Hello World!` on the console.

## Snapshot diagrams

快照图表示程序运行时的内部状态：包括堆和栈的状态
以下是对快照图的介绍：
1. 原始值用裸常数表示
2. 对象用圆形表示，里面用type:variable的格式表示成员变量，用箭头指向其值

### Mutating values vs. reassigning variables

改变值和改变引用是不一样的，这里用了两个例子来说明

```java
String s = "a";
s = s + "b";
```
String是不可变对象的示例，原本s指向String对象a，这里并不是将对象a的值变为ab，而是将s指向新对象ab

```java
final StringBuilder sb = new StringBuilder("a");
sb.append("b");
```
而StringBuilder是一个可变对象，这里append("b")就是将sb指向的对象值变为了sb

所以我们应该区分可变引用+不可变对象和不可变引用+可变对象，以上述两段语句为例：
1. `String s`前面没有final，即s的引用可以改变，但是由于String为不可变类型，所指向的值不可变，要改变他需要将其指向另一个String对象
2. `final StringBuilder sb`前面有final限定，所以sb的引用不可改变，但是StringBuilder是可变类型，要改变他只需要改变其值

## Java Collections

讲了一下List、set和map这三种接口的声明、常用方法和遍历方法

## Java API documentation

介绍了一些常用的java库，以BufferedReader为例，讲解了class的构成和使用API文档了解类的方法

