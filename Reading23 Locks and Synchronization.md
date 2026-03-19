---
created: 2026-03-17 20:58
updated: 2026-03-19T11:39
status: Completed
topics: Lock;Synchronization;Deadlock
---
>[!SUMMARY] Table of Contents
>    - [[Reading23 Locks and Synchronization#Introduction|Introduction]]
>    - [[Reading23 Locks and Synchronization#Synchronization|Synchronization]]
>        - [[Reading23 Locks and Synchronization#Bank account example|Bank account example]]
>        - [[Reading23 Locks and Synchronization#Locking|Locking]]
>        - [[Reading23 Locks and Synchronization#Monitor pattern|Monitor pattern]]
>        - [[Reading23 Locks and Synchronization#Deadlock|Deadlock]]
>    - [[Reading23 Locks and Synchronization#Developing a threadsafe abstract data type|Developing a threadsafe abstract data type]]
>        - [[Reading23 Locks and Synchronization#Steps to develop the datatype|Steps to develop the datatype]]
>        - [[Reading23 Locks and Synchronization#Locking discipline|Locking discipline]]
>        - [[Reading23 Locks and Synchronization#Designing a datatype for concurrency|Designing a datatype for concurrency]]
>    - [[Reading23 Locks and Synchronization#Atomic operations|Atomic operations]]
>        - [[Reading23 Locks and Synchronization#Sprinkling synchronized everywhere？|Sprinkling synchronized everywhere？]]
>        - [[Reading23 Locks and Synchronization#Deadlock rears its ugly head|Deadlock rears its ugly head]]
>        - [[Reading23 Locks and Synchronization#Deadlock solution 1: lock ordering|Deadlock solution 1: lock ordering]]
>        - [[Reading23 Locks and Synchronization#Deadlock solution 2: coarse-grained locking|Deadlock solution 2: coarse-grained locking]]
>    - [[Reading23 Locks and Synchronization#Goals of concurrent program design|Goals of concurrent program design]]
>    - [[Reading23 Locks and Synchronization#Concurrency in practice|Concurrency in practice]]
## Introduction

线程安全的宗旨：并发程序的正确性不应依赖于特定时序

[[Reading20 Thread Safety]]中提到了四种确保线程安全的策略，而前面三种已经被提及过了，本章详细阐述第四种策略：**Synchronization**

同步实际上是妥协的策略，如果类型中不使用共享数据（策略一），或者共享不可变数据类型（策略二），或者使用线程安全的可变数据类型（策略三），就不需要同步策略了。
## Synchronization

如上文所说，同步针对的是、可变的、非线程安全的共享数据，为了避免竞态条件，我们使用锁来保证同时只有一个线程访问。

锁有两个基本操作：acquire和release，与操作系统中的锁一致，不再展开

### Bank account example

这一部分就是说通过lock保证对账户的互斥访问，只有思路和伪代码，不做展开

### Locking

java中lock必须依附于某个对象类型存在，也就是没有单独的lock类型，可以声明String lock，只要是对象都可以作为锁。

声明了一个锁之后，有两种使用方法：
1. synchronized方法：声明某个方法为synchronized方法，默认使用this作为锁对象，锁粒度为整个实例对象
2. synchronized块：声明某个对象为锁对象，试图acquire该对象，由该同步块包裹的部分保持互斥访问

使用同步块有一个基本原则，一个被保护filed只能对应一把锁，不过也可以用一把锁保护多个fields，比如说this锁

### Monitor pattern

用上述两种方法将所有读或写rep的方法都用synchronize包裹，此时整个对象的状态变化都是原子的，即同时只有一个线程能对改对象操作

不过构造方法没有必要通过synchronize包裹，因为对象尚未创建之前不可能出现竞态条件，除非在构造函数中暴露this给其他线程

### Deadlock

这里依旧拿银行账户为例举例说明死锁，此处引用OS笔记中的部分说明

[[死锁#死锁发生的条件（Coffman条件）]]说明了死锁发生的条件，即互斥、持有并等待、资源非抢占和循环等待

## Developing a threadsafe abstract data type

这里举了一个多用户编辑器的例子，引入了线程安全的抽象数据类型涉及。前面[[Reading20 Thread Safety#Make a Safety Argument]]提到了线程安全论证，而[[Reading12 Abstract Data Types]]和[[Reading13 Abstraction Functions & Rep Invariants]]论述了如何涉及ADT，这里将其拓展为设计一个线程安全的抽象数据类型。

### Steps to develop the datatype

前面设计ADT有以下步骤：
1. spec
2. test
3. rep jk

到实现了rep并通过test就完成了，但是这个数据类型实际上没有满足线程安全，即使spec中规定了exp（safety from exposure）也不行。为例实现线程安全，需要继续以下步骤：

4. Synchronization：论证当前数据类型是否为线程安全的 
5. Iterate：在第四步或者更前的步骤就发现当前的spec无法满足一个线程安全的接口，所以需要回到操作集合处重新设计，并重复迭代直至线程安全论证完成

### Locking discipline

锁规范是使用同步策略保证线程安全的要求，有以下两点：
1. 每个共享的可变变量都必须由某个锁保护
2. 如果rep invariant与多个共享可变变量有关，则所有必须用同一把锁保护这些变量（实际上是保护不变量），并且在修改完之后检查不变量是否为真

### Designing a datatype for concurrency

这个子目录看上去和他的父目录[[#Developing a threadsafe abstract data type]]一个意思，但是这里主要提出了一个思想：好的并发设计应该把可能出现的并发问题在设计时解决，即使用各种策略设计一个易于使用的、并发安全的数据类型，而不是由客户端考虑对象或者某个操作需不需要加锁

## Atomic operations

即使遵循锁规范、满足了线程安全的所有要求的对象类型，也只能保证单个操作的时候不会产生竞态条件，当多个线程对其进行组合操作的时候，仍然无法杜绝竞态条件。

此时客户端在使用该对象的时候，可以用synchronized块包裹该对象，以实现组合操作的原子性。

### Sprinkling synchronized everywhere？

虽然同步块很好用，但是在设计一个类的时候不应该随便在所有方法中都使用它：
1. 带来更大的性能开销
2. 锁被暴露：使用者知道了该对象就是锁本身，而线程安全的具体实现方式不应暴露给使用者
3. 锁的正确性：有些为了同时操作的静态方法不应被上锁

这里看似有些诺言诺语，之前[[#Monitor pattern]]还说可以给除了构造方法的所有方法来实现原子性类型，实际上，这里的意思是：监视器模式并不适用于所有方法，应该更谨慎、灵活的使用synchronized，要配合线程安全的其他策略一起使用，并兼顾性能开销

### Deadlock rears its ugly head

这里是强调锁的使用会增加死锁的风险，事实上引入阻塞就会引入死锁，不过锁比阻塞I/O更容易导致死锁，尤其是监视器模式。

这是容易理解的，当所有对象都为锁本身的时候，相互的操作很容易造成循环等待。

### Deadlock solution 1: lock ordering

解决死锁的策略之一：破坏循环等待，通过按序获取锁实现

```java
    public void friend(Wizard that) {
        Wizard first, second;
        if (this.name.compareTo(that.name) < 0) {
            first = this; second = that;
        } else {
            first = that; second = this;
        }
        synchronized (first) {
            synchronized (second) {
                if (friends.add(that)) {
                    that.friend(this);
                } 
            }
        }
    }
```
这里实现了一个基于用户名的锁排序，即使多个线程需要相同的两把锁，也会因为锁排序而形成单向等待，即字典序更大的等待字典序更小的用户

当然实际社交网络中不可以使用用户名字典序排序作为锁排序，这是因为用户名可边，会导致锁排序失效，即锁排序的依据必须是不可变且全局唯一的。

不过虽然锁排序看起来挺强的，它仍然有一些缺点：
1. 非模块化：要求了解子系统中的所有锁
2. 计算成本：必须要进行一些遍历才能确定锁的情况

### Deadlock solution 2: coarse-grained locking

粗粒度锁是更暴力的解决方案，即在高层级的对象中设计一个锁，保护多个对象实例。

```java
public class Wizard {
    private final Castle castle;
    private final String name;
    private final Set<Wizard> friends;
    ...
    public void friend(Wizard that) {
        synchronized (castle) {
            if (this.friends.add(that)) {
                that.friend(this);
            }
        }
    }
}
```
这里使用一个castle锁保护所有Wizard对象，不过代价更大，整个程序变为顺序执行，性能较差

## Goals of concurrent program design

所以并发程序设计的两个目的为：
1. 安全性：不能破坏rep invariant，不能产生竞态条件，即不发生设计、预料之外的事
2. 活性：程序始终按预设运行，不因为死锁卡死

## Concurrency in practice

在实践中，不同情况下会使用不同的同步策略：
1. 库数据结构：要么不采取同步措施，把并发问题交给客户端处理（如ArrayList的spec：Note that this implementation is not synchronized)，要么使用监视器模式
2. 复合对象（对象内部包含多个可变部分）：粗粒度锁或者线程封闭（约定只有某个线程可以访问）
3. 搜索：使用不可变数据类型
4. 操作系统：使用细粒度锁，并通过锁排序处理死锁问题（当然实际上有很多策略，见[[死锁]]）