---
created: 2026-02-24 11:07
updated: 2026-02-28T21:33
status: Completed
topics: Mutability；Risk ；Use Immutability
---
>[!SUMMARY] Table of Contents
>    - [[Reading9 Mutability & Immutability#Mutability|Mutability]]
>        - [[Reading9 Mutability & Immutability#Risks of mutation|Risks of mutation]]
>        - [[Reading9 Mutability & Immutability#Specifications for mutating methods|Specifications for mutating methods]]
>        - [[Reading9 Mutability & Immutability#Iterator ：example of risk|Iterator ：example of risk]]
>    - [[Reading9 Mutability & Immutability#Mutation and contracts|Mutation and contracts]]
>        - [[Reading9 Mutability & Immutability#Mutable objects can make simple contracts very complex|Mutable objects can make simple contracts very complex]]
>        - [[Reading9 Mutability & Immutability#Mutable objects reduce changeability|Mutable objects reduce changeability]]
>    - [[Reading9 Mutability & Immutability#Useful immutable types|Useful immutable types]]
>    - [[Reading9 Mutability & Immutability#Summary|Summary]]
## Mutability

这里重述了[[Reading2 Basic java#Mutating values vs. reassigning variables]]中的String和StringBuilder的例子，并稍微拓展：
由于StringBuilder为可变类型，进行更改时不需要进行拷贝，效率更高

这是使用可变类型的一个理由，即更高的效率；除此之外，使用可变类型也可以使程序中的两个部分通信更顺畅

### Risks of mutation

虽然可变类型有一些好处，但是以下的风险使我们不得不尽量原理它：
1. 传递可变值：当可变值作为参数传入时，方法可能对其进行操作，但是使用者并不明知，导致定位bug困难和理解困难
2. 返回可变值：当一个变量接受可变返回值，并对该变量进行改变时，可能改变方法内部缓存导致行为异常，这同样难以debug

返回可变值的例子较为抽象，下面详细展开：

```java
/** @return the first day of spring this year */
public static Date startOfSpring() {
    if (groundhogAnswer == null) groundhogAnswer = askGroundhog();
    return groundhogAnswer;
}
private static Date groundhogAnswer = null;
```
这个`startOfSpring`方法的作用是问土拨鼠春天开始的日期作为返回值，但是由于请求太多土拨鼠会被问烦，所以实现中将询问结果作为全局静态变量存储，为了避免泄露使用private约束

```java
// somewhere else in the code...
public static void partyPlanning() {
    // let's have a party one month after spring starts!
    Date partyDate = startOfSpring();
    partyDate.setMonth(partyDate.getMonth() + 1);
    // ... uh-oh. what just happened?
}
```
这时候有一堆人在计划开party，但是觉得春初太冷了，所以决定延期一个月，为此将`partyDate`变量的月份+1

但是由于`partyDate`和`groundhogAnswer`实际上指向的是同一个引用，所以在修改`partyDate`的同时，无意中把私有变量`groundhogAnswer`也修改了，这就导致后序调用`startOfSpring`的时候会出现bug

要解决这个问题，有两种方式：使用不可变类型`LocalDateTime`存储，或者返回一个副本，即防御性复制。但是使用防御性复制等于放弃了可变类型的灵活性，因为每次调用都会创造一个副本，所以使用不可变类型是更一劳永逸的方法，毕竟不可变类型永远不用考虑防御性复制的问题

这两个例子反映了可变变量行为不可控的原因：它可能存在多个引用，指向同一个对象，但是各个操作者对彼此的行为相互不知晓，导致变量值不可控

### Specifications for mutating methods

当输入参数可能被改变时，必须在spec中明确说明，如未说明则默认不改变

### Iterator ：example of risk

迭代器是遍历各种容器的底层实现，但是它的可变性会导致一些风险

这是自制迭代器的next方法实现：
```java
public String next() {
        final String element = list.get(index);
        ++index;
        return element;
    }
```

但是当我们需要在列表中使用迭代器，删除某些元素的时候，会产生bug：
```java
public static void dropCourse6(ArrayList<String> subjects) {
    MyIterator iter = new MyIterator(subjects);
    while (iter.hasNext()) {
        String subject = iter.next();
        if (subject.startsWith("6.")) {
            subjects.remove(subject);
        }
    }
}
```

对于如下输入，输出并不符合预期：
```
// dropCourse6(["6.045", "6.005", "6.813"])
//   expected [], actual ["6.005"]
```

这是因为`MyIterator`内置了`private int index`,当remove对象`6.045`的时候，index已经递增至1，此时由于`6.045`被删除，subjects\[0]变为了`6.005`，而这个对象永远不会再被遍历到了，因为index只能向后遍历

这个bug在ArrayList内置的迭代器中依然存在，会抛出著名的`ConcurrentModificationException`异常：
```java
jshell> var subjects = new ArrayList<>(List.of("6.005", "6.031", "8.01", "6.042"));
subjects ==> [6.005, 6.031, 8.01, 6.042]
jshell> for (String s : subjects) {
   ...>             if (s.startsWith("6.")) subjects.remove(s);
   ...>     }
|  异常错误 java.util.ConcurrentModificationException
|        at ArrayList$Itr.checkForComodification (ArrayList.java:1096)
|        at ArrayList$Itr.next (ArrayList.java:1050)
|        at (#12:1)
```

即使使用itera内置的remove方法规避这个bug，如果存在多个迭代器对列表操作，仍然会由于没有相互通信而产生错误。这牺牲了安全性来换取效率，尽管如此依旧值得，因为使用不变类型遍历容器的开销是不可接受的。

## Mutation and contracts

### Mutable objects can make simple contracts very complex

可变对象的基本问题：对同一对象的多重引用，导致涉及该对象的契约(constract)依赖于每个持有者的良好行为，而不是仅依赖于客户和实现者之间，导致简单的契约复杂化

### Mutable objects reduce changeability

可变类型反而会降低灵活度和可变性，比如由于担心多重引用，在spec中规定可变返回值永远不可以改变，这实际上加强了规范，使其变成一个终身合同，反而降低了灵活性。从安全性和便捷性上都不如使用不可变类型，除非性能开销差距。

## Useful immutable types

既然不可变类型那么好用，下面列出了几种常用的不可变类型：
1. 基本类型和他们的包装器（大写版）
2. 时间相关的使用java.time中的包装类
3. 基本集合的只读包装器：`Collections.unmodifiableList`当这被包装对象的持有者试图改变时，会抛出`UnsupportedOperationException`异常
4. 不可变空集合：`Collections.emptyList`

## Summary

本章主要强调可变类型会导致的一系列问题，本质问题是可变对象的多重引用可能导致的不可控性，为了消解这个问题可以使用防御性复制或者返回可变对象的不可变视图，但是反而会带来额外的性能开销并削减灵活性，所以尽可能使用不可变类型吧。