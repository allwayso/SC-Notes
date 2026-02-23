---
created: 2026-02-23 20:16
updated: 2026-02-23T21:22
status: Updating
topics: Debug;Avoid Debug
---
## First Defense: Make Bugs Impossible

在[[Reading1 Static Check#Static Check]]中提到了静态检查和动态检查，为了使在设计时让代码免于错误，还可以使用不变量

immutable并不是新概念，在[[Reading2 Basic java#Mutating values vs. reassigning variables]]中提到过不变引用和不变对象，并对引用和对象的不变性进行了区分，简而言之：
1. 不变对象：指的是该类型的值不可改变，但是这个变量可以指向另一个引用
2. 不变引用：指该对象被final约束，不能指向另一个引用，但是其內部值可以改变

## Second Defense: Localize Bugs

为了定位bug，我们沿用快速失败（fail fast）思想，当程序可能不合预期直接中止或者抛出异常，比如对方法输入的检查

### Assertion

快速失败等防御性编程手段经常通过断言实现，断言语法为`assert (Expressions)`，如果表达式为假则抛出`AssertError`异常，并提供错误处的堆栈信息

需要注意的是，Java中的断言是默认关闭的，需要通过在`VM Argument`处写入-ea指令开启

另外，Assert与JUnit中的`assertTrue`或`assertEquals`方法应用场景不同，前者用于在方法内部进行逻辑校验，后者用于在构建测试套件时进行判断，而且后者提供的信息更完整，还不受虚拟机参数约束

什么时候使用Assert：
可以在参数传入，返回值校验和覆盖所有情况三种场景下使用断言，根本上是校验方法内部逻辑是否正确

什么时候不应该使用Assert：
浅显的逻辑（当逻辑正确的时候不要怀疑编译器），外部失败（这不是断言的领域，应该交给异常）

### Incremental Development

渐进式开发，即通过测试保证当前部分正确，并在进行开发时重新测试，由此可以基本确定问题出在新加入的功能中，前面提到了实现渐进式开发的两种方式：[[Reading3 Test#Part and Whole：Unit Testing and Stubs|Unit Test]]和[[Reading3 Test#Development of Test Suite：Automated Testing and Regression Testing|Regression Test]]

### Modularity & Encapsulation

模块化和封装也是定位错误的重要方式

模块化指每个部分独立于其他部分，可以单独进行实现、调试，通过模块化可以定位模块的错误，而不是在庞大的系统中寻找错误

封装指将某个模块与其他部分进行隔离，包括访问控制和变量作用域，其中变量作用域越小越容易定位错误

#### Minimizing the scope of variables

关于最小化变量作用域，有以下惯例：
1. 在for循环中初始化循环变量
2. 在第一次需要时声明变量：没必要在方法开头声明所有需要的变量
3. 尽量避免使用全局变量

## Summary

本章包含两部分内容：如何避免调试和如何定位错误。第一部分主要通过静态检查、动态检查和使用不变量避免bug发生，第二部分则通过断言、渐进式开发、模块化和封装来定位错误。本章内容并没有什么新知识，只是把前面章节中有关check、test等内容通过**调试**这个主线串起来讲了，其中最小化作用域和断言的使用场景是新内容

