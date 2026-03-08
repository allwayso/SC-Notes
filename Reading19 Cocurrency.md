---
created: 2026-03-08 20:22
updated: 2026-03-08T22:42
status: Completed
topics: Process,Threads and time-slicing;Anonymous class;Cocurrent bugs
---
>[!SUMMARY] Table of Contents
>    - [[Reading19 Cocurrency#Two Models for Concurrent Programming|Two Models for Concurrent Programming]]
>    - [[Reading19 Cocurrency#Processes, Threads, Time-slicing|Processes, Threads, Time-slicing]]
>    - [[Reading19 Cocurrency#Anonymous classes|Anonymous classes]]
>        - [[Reading19 Cocurrency#具名类 vs. 匿名类 (Concurrency Context)|具名类 vs. 匿名类 (Concurrency Context)]]
>    - [[Reading19 Cocurrency#The else|The else]]
## Two Models for Concurrent Programming

共享内存模型：多个并发进程处理相同物理内存
消息传递模型：多个并发进程通过信息通道相互通信

## Processes, Threads, Time-slicing

这些概念在操作系统中已经基本了解，只列出几点与java相关的点：
1. 进程通过System.in和System.out流相互通信
2. 进程可以看作一个虚拟计算机，线程可以看作虚拟处理器，同一进程下的线程共享内存

## Anonymous classes

创建一个接口的实现有两种方式，即使用具名类和匿名类，其区别整理在以下表格中

### Declared vs Anonymous 

|            | **具名实现类 (Declared Class)**                | **匿名内部类 (Anonymous Class)**       |
| ---------- | ----------------------------------------- | --------------------------------- |
| **复用性 **   | **高**。定义一次，可以在项目的任何地方多次实例化。               | **极低**。一次性使用，无法在别处再次调用。           |
| **作用域 **   | **广**。类名在整个包（或 public 范围内）可见，增加了外部依赖的可能性。 | **窄**。只在定义的代码块内有效，完美实现“局部化”。      |
| **代码位置**   | 逻辑分散。需要跳转到独立的文件或类定义处查看。                   | **所见即所得**。任务逻辑与启动代码紧挨在一起，便于阅读。    |
| **样板代码**   | **繁琐**。需要写完整的类声明、构造函数等。                   | **精简**。直接写核心逻辑，省略了给类起名字的过程。       |
| **适用逻辑长度** | 适合**复杂、长篇**的逻辑。                           | 适合**短小、精悍**（通常 10 行以内）的逻辑。        |
| **变量访问**   | 只能访问传入构造函数的参数或 public 成员。                 | 可以直接“捕获”外部作用域的 `final` 局部变量。      |
| **符合原则**   | **DRY **：避免重复逻辑。                          | **Encapsulation** ：将无关细节隐藏在最小范围内。 |

所以对于简单任务而言，使用匿名类创建线程是更划算的方法，也是本章提及匿名类的原因：
```java
public static void main(String[] args) {
    new Thread(new Runnable() {
        public void run() {
            System.out.println("Hello from a thread!");
        }
    }).start();
}
```

## The else

本章其他内容讲述了并发问题的机理，和一些引发并发问题的情境，这些概念在操作系统中已经进行了学习，具体参考[[进程同步与互斥机制]]笔记。不再展开。以下只对Heisenbug的概念进行补充。

海森bug是一种一旦调试就消失的bug，具有非确定性且难以复现，通常是由于执行调试的IO操作速度较慢，打破了产生bug的线程交叉执行顺序，导致bug消失。

虽然听着很诡异，但是这告诉我们：由于并发bug的产生具有不确定性，极难通过调试和测试来进行消除和溯源，所以对于并发程序而言，设计>测试