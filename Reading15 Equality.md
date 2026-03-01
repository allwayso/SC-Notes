---
created: 2026-03-01 10:31
updated: 2026-03-02T00:34
status: Completed
topics: equals() and hashCode();Object contract;mutable and immutable types
---
>[!SUMMARY] Table of Contents
>    - [[Reading15 Equality#Introduction|Introduction]]
>    - [[Reading15 Equality#Three Ways to Regard Equality|Three Ways to Regard Equality]]
>    - [[Reading15 Equality#The Object Contract|The Object Contract]]
>        - [[Reading15 Equality#Breaking Hash Tables|Breaking Hash Tables]]
>    - [[Reading15 Equality#Equality of Mutable Types|Equality of Mutable Types]]
>    - [[Reading15 Equality#Summary|Summary]]
## Introduction

本章的equality指的是两个变量的值相等，而非引用相等
## Three Ways to Regard Equality

分别从抽象函数设计、代码实现和观察者角度看待相等性：
1. 抽象函数：a=b当且仅当f(a)=f(b)，即通过表达值映射到的抽象对象相同，则两对象相同
2. 等价关系：Equals方法必须满足等价关系的三个性质，即自反性、对称性和传递性
3. 观察视角：如果两个对象的所有操作的结果都相等，则这两个对象相等

## The Object Contract

对象契约相当重要，其中的equals方法的contract更重要：
1. equals必须定义一个等价关系
2. equals必须保持一致性：若参数对象不变，多次调用equals的结果应该一致
3. 任何非空对象应与空引用不相等
4. hashCode对两个相等对象的返回值应该相等：相等的对象应该被哈希映射到同一个桶中

这里之所以有哈希值的要求，是因为hashmap和hashSet依赖于hashCode的实现，为了使该类型能被所有容器正确使用，在定义类型时必须满足要求4，来适配两种哈希容器
### Breaking Hash Tables

这里介绍了一下哈希表的工作原理，即通过哈希函数将键映射到某个值的方法，是为了进一步解释要求4。

为了使类型适配于哈希容器，我们规定了相等对象的哈希值应该相等，hashCode的默认方法与equals类似，都是返回由地址解算出来的一个值，与本章比较对象值的equals目标冲突，所以在重写equals方法的时候总是得重写hashCode。

## Equality of Mutable Types

以上的相等性探讨都是针对不可变对象而言，而对于可变对象来说，由于引入mutator操作，可以将相等性分为两种：
1. 观察相等性：即两个可变对象只通过非mutator方法无法区分
2. 行为相等性：对两个可变对象施加任何操作序列都无法区分，此时两个对象为同一值的不同引用

观察相等性显然弱于行为相等性，Java对大多数可变类型都使用观察相等性，但是这可能引入一些微小的bug：
```java
List<String> list = new ArrayList<>();
list.add("a");

Set<List<String>> set = new HashSet<List<String>>();
set.add(list);
```
此时set.contains(list) → true，但是如果修改了list：`list.add("goodbye")`,contains方法的返回值将会变为false

这是因为list<String>类型的hashCode与其中内容有关，导致以他在HashSet中的哈希值改变。list元素存在旧的哈希值索引处，而contains方法根据新的哈希值找list元素，自然找不到，于是便认为set中不存在这个列表，但是通过遍历却能遍历到这个元素，这体现了hashSet两种方法的矛盾性

由于哈希容器对可变对象的支持不稳定，所以`java.util.Set` 的spec中提醒，如果执意将可变元素作为Set元素，则必须小心谨慎

也是出于这个原因，本章认为可变类型应该实现行为相等性，即两相等的引用只有作为同一个对象的别名时才满足equals，所以应该继承Object类的equals和hashCode方法。而对于需要“相似性”的可变类型，则应该另设一个similar方法，与equals方法区分

## Summary

总结：不可变对象通过抽象值比较，可变对象通过默认equals和hashCode方法比较



