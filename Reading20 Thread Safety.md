---
created: 2026-03-09 10:02
updated: 2026-03-09T21:55
status: Completed
topics: The definition of ThreadSafe;3 Strategy to achieve thread safety;Thread safety argument
---
>[!SUMMARY] Table of Contents
>    - [[Reading20 Thread Safety#What Threadsafe Means|What Threadsafe Means]]
>    - [[Reading20 Thread Safety#Strategy 1: Confinement|Strategy 1: Confinement]]
>    - [[Reading20 Thread Safety#Strategy 2: Immutability|Strategy 2: Immutability]]
>        - [[Reading20 Thread Safety#Stronger definition of immutability|Stronger definition of immutability]]
>    - [[Reading20 Thread Safety#Strategy 3: Using Threadsafe Data Types|Strategy 3: Using Threadsafe Data Types]]
>        - [[Reading20 Thread Safety#Threadsafe Collections|Threadsafe Collections]]
>        - [[Reading20 Thread Safety#Make a Safety Argument|Make a Safety Argument]]
>    - [[Reading20 Thread Safety#Summary|Summary]]
## What Threadsafe Means

当一个数据类型或者一个静态方法被多个线程使用时，不论线程如何执行，不需要额外的调度，始终表现正确，则称其为线程安全

表现正确：指始终遵循spec且保持不变性
无论线程如何执行：线程可以在多个处理器上并发或者在单个处理器上并发
不需要额外的调度： 数据类型不应就时间顺序提出前提条件，比如不能规定在set()的同时不能get()

## Strategy 1: Confinement

策略一是限制，即将数据限定在线程内部。

局部变量总是线程受限的，但是如果一个他是一个对象引用，则应检查对象是否具有线程受限性。

而对于全局变量，比如静态变量，则java不能保证其为线程受限的。由于限定其始终只被一个线程使用很困难，而且极易产生silent failure，所以最好不要使用全局变量。

## Strategy 2: Immutability

策略二是不可变性，即使用不可变引用和不可变数据类型。

不可变引用仍需保证其指向对象也不可变，如[[Reading09 Mutability & Immutability#Risks of mutation]]中所述，不可变引用仍然存在危险。

不可变类型通常是安全的，但是由于我们对于不可变类型的定义相对宽松——只要求整个周期中的抽象值保持不变，而没有严格要求其内部表示不变，导致不可变类型不一定线程安全。比如一个不可变列表，拥有一个可变长度值，在第一次size操作时初始化，虽然能在整个周期中保持输出值不变，但是内部表示发生了变化。这被称为良性变异，而良性变异不是线程安全的，必须通过锁进行限制。

### Stronger definition of immutability

先前保证抽象值不变的不可变性定义对于线程安全而言不够强，所以引入更强的不可变性定义：
1. 所有fields都为final和private的
2. 不存在rep exposure
3. 不存在mutator操作
4. 不存在可变类型的良性突变

## Strategy 3: Using Threadsafe Data Types

策略三是将共享的可变数据存储在线程安全数据类型中。Java库会在spec中说明数据类型的线程安全性。

比如StringBuffer是线程安全的：
```markdown
[StringBuffer is] **A thread-safe, mutable sequence of characters**. A string buffer is like a String, but can be modified. At any point in time it contains some particular sequence of characters, but the length and content of the sequence can be changed through certain method calls.
```

而StringBuilder不是线程安全的:
```markdown
[StringBuilder is] A mutable sequence of characters. This class provides an API compatible with StringBuffer, but **with no guarantee of synchronization**. This class is designed for use as a drop-in replacement for StringBuffer in places where the string buffer was being used by a single thread (as is generally the case). Where possible, it is recommended that this class be used in preference to StringBuffer as it will be faster under most implementations.
```

在Java API中经常出现两个功能相似、线程安全性不同的两个数据类型，通常不保证线程安全性的数据类型性能会更高一点。但是对于StringBuffer和StringBuilder而言，他们并没有一个公共接口，导致数据类型的切换非常困难，这一点在下文中的collection库做的更好一点。

### Threadsafe Collections

Java中的集合接口都不是线程安全的，比如set，map等，但是collection给他们提供了一个包装类，使其操作拥有原子性。

但是操作原子性仍然不能保证线程安全，这是因为操作序列不具备原子性，即调用顺序不确定，而且可能出现竞态条件。所以即使使用线程安全API，仍然要求对其安全性进行证明，以isPrime方法为例：
```java
private static Map<Integer,Boolean> cache =Collections.synchronizedMap(new HashMap<>());
...

if (cache.containsKey(x)) return cache.get(x);
boolean answer = BigInteger.valueOf(x).isProbablePrime(100);
cache.put(x, answer);
```

由于cache经过Collenctions包裹，containsKey(),get()和put()都为原子操作，但是如果有多个进程并发执行这一段代码，仍然对cache对象可能有竞争。

所以即使单个操作拥有原子性也不能保证操作序列的正确性，因此我们需要论证竞争操作是否会威胁不变式。对于本问题而言：
 1. containsKey和get操作的竞争：由于该缓存只增不减，所以不会出现某线程执行到get的时候key被删除的情况
 2. containsKey和put操作的竞争：由于isProbablePrime对于同一个x的返回值永远相等，所以即使put的同时另一进程在containsKey，也只会重复put一个同样的键值对，只会进行重复计算，不影响正确性

### Make a Safety Argument

就像safety from rep exposure一样，我们对于一个数据类型也应该写Thread Safety Argument，实际上就是证明对于其中的每个field都不会出现竞态条件，即即使多线程对该变量同时操作，仍然不会出现不可预知的行为。

可以通过以下决策树来进行线程安全论证：
```
Field 安全性自检流程：
|
|-- [1] 只有当前线程能看到吗？
|   |-- 是 -> (策略：限制) -> 安全:局部变量或限制在特定线程中
|   |-- 否 -> 进入 [2]
|
|-- [2] 是 final 引用且指向不可变对象吗？
|   |-- 是 -> (策略：不可变性) -> 安全：该对象不存在写操作，自然破坏竞态条件
|   |-- 否 -> 进入 [3]
|
|-- [3] 是否封装在线程安全容器中？
|   |-- 是 -> 检查操作逻辑：
|   |        |-- 只有单一原子方法调用 (如 map.get) -> 安全
|   |        |-- 存在复合逻辑 (如 if-then-put) -> 进入 [4]
|   |-- 否 -> 进入 [4]
|
|-- [4] 是否有明确的锁同步 (synchronized/Lock)？
    |-- 是 -> (策略：同步) -> 安全
    |-- 否 -> 警告：存在竞态条件 (Race Condition)！
```

## Summary

本章讲述了线程安全的定义，保证线程安全的三个策略（封装、不可变性、线程安全变量类型），以及线程安全论证方法。