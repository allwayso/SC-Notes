---
created: 2026-03-01 10:42
updated: 2026-03-06T21:52
status: Updating
topics: interface；generics；
---
## Description

本问题集为抽象数据类型实现程序的完整流程，initial commit中其实已经包含了抽象类的spec和数据类型选择，需要自行完成test-implement-debug流程。由于涉及抽象数据类型，所以测试类的实现也应该有抽象类和实现子类，以实现DRY目的。

## Problem 1: Test `Graph​<String>`

程序起点：Graph完成spec，子类和测试类只给出最基本结构
接口情况：Graph是一个有向图泛型抽象类，有两个子类，分别通过以点为索引的邻接表和传统<V,E>形式存储
题目要求：写Graph的类型为\<String>时的测试，根据抽象类中的spec写各个操作的spec，以复用在两个接口的测试类中

## Problem 2: Implement Graph​

实现步骤：
1. 设计ConcreteEdgesGraph及其辅助类Edge的spec
2. 设计和实现对Edge和toString方法的测试套件
3. 实现ConcreteEdgesGraph及其辅助类
4. 调试debug
5. 同一步骤实现另一rep

## Problem 3: Implement Graph​\<L>

本章将两个实现从\<String>类型扩展到泛型\<L>，通过以下步骤实现：
1. 将底层类vertex和edge的类型改为\<L>
2. 将实现类的类型改为Graphxxx\<L>
3. 将内容中的Edge和Vertex变量类型改为\<L>类型
4. 将测试类中的变量类型改为\<String>类型、

## Problem 4:Poetic Walks

在实现了Graph之后，本节希望用这个数据类型实现一点艺术性的功能：根据文本库，对input语句进行润色

为了实现这个功能，我们需要先实现GraphPoet：
1. 完成GraphPoet的spec：RI，AF，RE等，这里增加了一个接受String类型文本的构造函数，通过重载实现
2. 设计测试套件
3. 实现GraphPoet并测试

之后就可以通过构造函数给GraphPoet注入灵魂，我选择的是Shakespeare的十首十四行诗，以期通过简单意向补全一些陌生化的诗句，但是发现了很多问题：
1. 注入灵魂样本量太少：诗文本身的文本内容太少，很难连成连续、复杂的文本图
2. spec规定的分隔词方法漏洞：通过space和换行划分，没有清洗标点符号，导致文本图中出现`xxxx,`顶点，对润色文本没有任何作用
3. 本质缺陷：除非是正好根据样本来刻意挖去一个bridge单词，否则英语中很难找到同时做a的后缀和b的前导的词
4. 逻辑问题：如果可以允许bridge为一个词组，则灵活度将大大提高，也更符合实际逻辑，当然实现和内存负担会造成一些困难

但是时间有限，只能稍微增加一点文本库，做一下标点符号清洗，在多次尝试下终于找到能够连续迭代的用例：
```
original vertion: to thou art
number 1 version: to time thou art
number 2 version: to the time thou art
number 3 version: to hear the time thou art
```

## Summary

PS1初步引入spec、test、debug的概念，并开始test-first programming的实践。
在本问题集中，不仅加深了对测试优先编程的理解，也引入了新的概念：接抽象类型，及其下的抽象函数、表达不变性、接口等知识
本章实现了抽象类型Graph，以及两个分别基于edge和vertex的子类，并实现了泛型，最终使用了该数据类型完成客户端的使用，总的而言，涵盖了12-15节的基本内容，是一个很好的实践。