---
created: 2026-03-03 11:39
updated: 2026-03-04T22:32
status: Updating
topics:
---
# Part1：Recursive Data Types

第一部分讨论了不可变列表类`ImList`的递归定义，不过我们应该先了解递归数据类型的定义，与递归算法类似，递归数据类型也由基础情况和递归步骤定义。

对于不可变列表，其基本情况`empty`即创建并返回空列表，递归步骤`cons`为在当前列表前加一个元素并返回这个新列表，这两个操作递归定义了不可变列表的创建过程。为了实现它递归遍历，我们引入另外两个基本操作，`first`返回第一个元素，`rest`返回除了第一个元素外的剩余列表。

为了实现ImList，我们创建了两个子类，Empty和Cons，分别实现列表为空和非空时的逻辑。这种接口设计与ArrayList和LinkedList本质不同，后者是通过不同子类实现了不同的底层数据结构，而Empty和Cons是相互配合实现ImList的逻辑。

Empty子类的实现如下：
```java
public class Empty<E> implements ImList<E> {
    public Empty() {
    }
    public ImList<E> cons(E e) {
        return new Cons<>(e, this);
    }
    public E first() {
        throw new UnsupportedOperationException();
    }
    public ImList<E> rest() {
        throw new UnsupportedOperationException();
    }
}
```

Cons子类实现如下：
```java
public class Cons<E> implements ImList<E> {
    private final E e;
    private final ImList<E> rest;

    public Cons(E e, ImList<E> rest) {
        this.e = e;
        this.rest = rest;
    }
    public ImList<E> cons(E e) {
        return new Cons<>(e, this);
    }
    public E first() {
        return e;
    }
    public ImList<E> rest() {
        return rest;
    }
}
```

此时客户端只需要使用ImList接口即可完成所有操作，而不用关心Cons和Empty实现类的情况

```java
ImList<Integer> myList = ImList.empty().cons(3) .cons(2) .cons(1);
```

这里实际上调用ImList.empty()是创造了一个Empty类型的实例，但是在调用cons方法后，`return new Cons<>(e, this);`，myList变为Cons类型，但是客户端不需要关心变量的类型，只需要调用接口函数即可

## Recursive datatype definitions

递归数据类型可以用一个等式表示，如ImList：`ImList<E> = Empty + Cons(first:E, rest:ImList)`

其格式要求为：
1. 左侧为抽象数据类型，由右边的rep定义
2. rep由用+号相连的变体（variant）表示
3. 每个变体是一个构造函数

这里的variant不是变量，而是抽象数据类型的不同形态，所以+的含义是”或“，即ImList要么是Empty，要么是Cons形态

## Functions over recursive datatypes

递归数据类型的函数看似和一般抽象数据类型的函数没什么区别，但是由于它的实现类实际上代表着抽象类的不同形态，所以这里的函数也具有递归性质

以size方法为例：Empty类中直接return 0，Cons方法中return (1+rest().size())，这里size的返回值也是依赖于Cons的上一个形态的返回值，所以也具有递归性质

contains方法同理，Empty类直接return false，Cons方法中return(first.contains(e)||rest.contains(e));

## Tuning the rep

但是对于size方法而言，o(n)的时间复杂度显然不达预期，所以这里通过缓存一个size变量来使其加快到o(1)

```java
private int size = 0;
public int size() { 
        if (size == 0) size = 1 + rest.size();
        return size;
    }
```

我们在不可变数据类型中内置了一个可变的rep，而没有改变其不可变性，是因为暴露给用户的size方法的返回值是恒定的。

所以不可变数据类型并不排斥所有可变rep，只要保持不可变性即可。

## Avoid Null

Empty和Cons其实可以放在一个类中，即直接将ImList作为实现类，通过迭代方式实现Cons的功能，用null值替代Empty

但是MIT6.005非常反感null值，是由于充斥在实现代码中的if判断，而且还不能对空对象调用任何操作。而如果用Empty这个哨兵对象的话，则避免了上面的两个缺点，而且意义更明确，可读性更强。

Anyway，keep `null` s out of your data structures, and your life will be happier.

## Another example: Boolean formulas

ImList的例子非常直观，为了更深入理解递归数据类型，这里引入了布尔公式，即命题

其递归数据类型定义如下：
```
Formula = Variable(name:String)
		  + Not(formula:Formula)
          + And(left:Formula, right:Formula)
          + Or(left:Formula, right:Formula)
```

即一个命题有以下几个形态：变量，非、和、或运算符构成

对于一个命题，我们想知道其是否能为真，则需要构建一个evaluate方法，计算每个指派对应的真值

## Summary for part1

- **核心定义**：由**基础情况**（`empty`）和**递归步骤**（`cons`）定义。对于 `ImList`，这组成了创建过程；而 `first` 和 `rest` 则定义了其递归遍历过程。
    
- **实现结构**：采用“接口 + 子类”设计。`Empty` 和 `Cons` 两个变体（Variants）相互配合实现逻辑，客户端只需通过 `ImList` 接口操作，无需关心具体形态。
    
- **递归函数**：函数结构与数据形态对称。例如 `size()` 和 `contains()` 在不同子类中各司其职（`Empty` 返回基础值，`Cons` 进行递归调用）。
    
- **Rep 优化**：支持**良性突变**（Beneficent Mutation）。通过在不可变对象内部缓存 `size` 变量提高性能，只要外部抽象值恒定，就不破坏不可变性。
    
- **设计原则**：用哨兵对象（`Empty`）替代 `null`，减少 `if` 判断并增强代码鲁棒性。

# Part 2: Writing a Program with Abstract Data Types

## Recipes for programming

本节回顾了测试优先方法完成一个静态方法的过程，并将其推广到抽象类和完整程序的实现上，这里直接介绍遵循test-first programming原则，使用抽象数据类型实现完整程序的过程：
1. 选择抽象数据类型：决定哪些为可变类型，哪些为不可变类型
2. 选择过程：编写顶层过程，并将其分解为小过程
3. 设计spec：为抽象数据类型和过程写spec，保持ADT操作简单
4. 测试：为ADT和procedure编写测试，并在这个过程中修改spec，使其更加规范
5. 简单实现：尽快整合体系，实现基本功能，暂时不考虑开销和复杂边界问题，保留to-do list
6. 迭代优化

## Problem: matrix multiplication

以矩阵运算为例，展示用抽象数据类型实现程序的过程

### Choose datatypes & desigh spec

这里选择了MatrixExpression（下文简称为表达式类型）作为抽象数据类型，并定义了一些基本操作：
1. make：输入double类型和double\[]\[]类型变量，返回相应表达式变量
2. times：输入两个表达式变量，返回表达式结果
3. isIdentity：输入表达式变量，判断是否为单位阵
4. optimize：输入表达式变量，返回更容易计算的表达式

### Test

以optimize为例：

- Number of scalars in expression: 0, 1, 2, >2
- Position of scalar in expression tree: immediate left, immediate right, left-of-left, left-of-right, right-of-left, right-of-right

### Choose a rep

将MatrixExpr看作递归数据类型是很自然的，它可以被拆分为以下形态：常数、矩阵、乘法、单位阵

其定义表达式为：MatExpr = Identity + Scalar(double) + Matrix(double\[]\[]) + Product(MatExpr, MatExpr)

### Choose an identity

这里将Identity类型变量作为null值

### Implementing `make` : use factory methods

我们想实现make方法，同时也不希望暴露Scalar、Matrix等内部实现，所以可以在接口类中通过静态方法实现

```java
/** @return a matrix expression consisting of just the scalar value */
public static MatrixExpression make(double value) {
    return new Scalar(value);
}

/** @return a matrix expression consisting of just the matrix given */
public static MatrixExpression make(double[][] array) {
    return new Matrix(array);
}
```

这种静态构造方法也被称为工厂方法。工厂方法介绍见[[Reading12 Abstract Data Types#Classifying Types and Operations]].

### Implementing `isIdentity` : don’t use `instanceof`

在实现isIdentity的时候，如果作为一个静态方法，则需要用instanceof方法不断判断形态类型，而本节强调抽象数据类型中应尽量避免使用该方法，而应该将其分解为适合于每个形态的方法

### Implementing `optimize` without `instanceof`

当然用各种instanceof实现静态方法optimize是不符合规范的，为了将其分解为适合于每个形态的方法，我们引入scaler和matrix操作，将表达式类型变量分解为标量和矩阵。

这样的话，对于其他形态，optimize只需要return this，而对于product类，返回times(scaler,matrix)即可

## summary for part2

- **开发配方**：遵循**测试优先**原则。流程为：选择 ADT $\rightarrow$ 选择过程（Procedures） $\rightarrow$ 设计 Spec $\rightarrow$ 编写测试 $\rightarrow$ 简单实现（先跑通全流程） $\rightarrow$ 迭代优化。
    
- **递归思维进阶**：以矩阵运算为例，将表达式建模为递归数据类型（常数、矩阵、乘法等变体），将复杂计算转化为树状结构的递归处理。
    
- **最佳实践**：
    
    - **工厂方法**：在接口中使用静态方法（`make`）创建实例，隐藏内部实现类。
        
    - **去 `instanceof` 化**：严禁使用类型判断，提倡利用**多态**将逻辑分散到各个形态的方法中实现。