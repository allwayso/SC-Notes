---
created: 2026-02-26 21:32
updated: 2026-02-27T00:34
status: Completed
topics: Invariants;Abstraction funcion;representation invariant;Safty from Rep Exposure;Replace Preconditions
---
>[!SUMMARY] Table of Contents
>    - [[Reading13 Abstraction Functions & Rep Invariants#Invariants|Invariants]]
>        - [[Reading13 Abstraction Functions & Rep Invariants#Immutability|Immutability]]
>        - [[Reading13 Abstraction Functions & Rep Invariants#Immutable Wrappers Around Mutable Data Types|Immutable Wrappers Around Mutable Data Types]]
>    - [[Reading13 Abstraction Functions & Rep Invariants#Rep Invariant and Abstraction Function|Rep Invariant and Abstraction Function]]
>        - [[Reading13 Abstraction Functions & Rep Invariants#Example: Rational Numbers|Example: Rational Numbers]]
>            - [[Reading13 Abstraction Functions & Rep Invariants#Checking the Rep Invariant|Checking the Rep Invariant]]
>            - [[Reading13 Abstraction Functions & Rep Invariants#No Null Values in the Rep|No Null Values in the Rep]]
>    - [[Reading13 Abstraction Functions & Rep Invariants#Documenting the AF, RI, and Safety from Rep Exposure|Documenting the AF, RI, and Safety from Rep Exposure]]
>        - [[Reading13 Abstraction Functions & Rep Invariants#How to Establish Invariants|How to Establish Invariants]]
>    - [[Reading13 Abstraction Functions & Rep Invariants#ADT invariants replace preconditions|ADT invariants replace preconditions]]
>    - [[Reading13 Abstraction Functions & Rep Invariants#Summary|Summary]]
## Invariants

不变性指一个在程序运行过程中某个谓词始终为真，通过方法内部结构保证，不依赖于客户端的实现

### Immutability

需要说明的是，不可变性是不变性的一种简单情况，而不等同于不变性。因为不变性可以允许值改变，只要谓词始终为真即可，当值不变的情况下满足谓词为真相对简单。

接下来引入Tweet作为例子，为了将其变为具有不可变性的类型，需要对实现进行改造：
```java
/**
 * This immutable data type represents a tweet from Twitter.
 */
public class Tweet {

    public String author;
    public String text;
    public Date timestamp;

    /**
     * Make a Tweet.
     * @param author    Twitter user who wrote the tweet
     * @param text      text of the tweet
     * @param timestamp date/time when the tweet was sent
     */
    public Tweet(String author, String text, Date timestamp) {
        this.author = author;
        this.text = text;
        this.timestamp = timestamp;
    }
}
```

当前实现中存在一个很明显的问题：
表示暴露（Representation exposure）：使用者能够从外部轻易改变author等变量值

这一步的修改非常简单，将访问控制改为private即可，当然这些状态值应该提供get方法返回：
```java
	private final String author;
	private final String text;
	private final Date timestamp;

/** @return Twitter user who wrote the tweet */
    public String getAuthor() {
        return author;
    }

    /** @return text of the tweet */
    public String getText() {
        return text;
    }

    /** @return date/time when the tweet was sent */
    public Date getTimestamp() {
        return timestamp;
    }
```


不过这并没有完全解决表示暴露的问题，只是转变为了相对隐蔽的形式：
Data是可变类型，当使用者接受`getTimestamp`方法返回的Data类型返回值，并将其改变时，`timestamp`会同步被改变

这类场景在[[Reading9 Mutability & Immutability]]中已经阐述过这类问题的成因和解决方法：防御性复制或者替换为不可变类型。
先采用防御性复制：
```java
public Date getTimestamp() {
    return new Date(timestamp.getTime());
}
```

返回`timestamp`的拷贝，这样就避免了通过返回值改变来修改fields，但是使用可变类型的荼毒尚未清除干净：
```java
/** @return a list of 24 inspiring tweets, one per hour today */
public static List<Tweet> tweetEveryHourToday () {
    List<Tweet> list = new ArrayList<Tweet>(); 
    Date date = new Date();
    for (int i = 0; i < 24; i++) {
        date.setHours(i);
        list.add(new Tweet("rbmllr", "keep it up! you can do it", date));
    } 
    return list;
}
```

此处Tweet列表传入了共同的Date类型参数date，每个Tweet共享同一个date副本，导致最终日期值完全一致。
为了解决这个问题，同样也可以通过防御性复制解决：
```java
public Tweet(String author, String text, Date timestamp) {
    this.author = author;
    this.text = text;
    this.timestamp = new Date(timestamp.getTime());
}
```

优先使用不可变类型，一劳永逸；无法使用不可变类型的情况下，使用可变类型+防御性复制；只有当防御性复制的成本过高，而用不可变类型又无法实现的情况下，才考虑在spec中对某个变量的后续行为进行约束，比如timestamp在全局代码中都不应该改变。

### Immutable Wrappers Around Mutable Data Types

[[Reading9 Mutability & Immutability#Useful immutable types]]中列举过这种折中方案：给可变类型加一个不可变包装器，只向用户提供不可变包装器，此时将会在运行时获得不可变性

## Rep Invariant and Abstraction Function

这一部分是对抽象类型的底层逻辑的理论推导，相对比较抽象，且与离散数学的函数比较相近

先介绍函数的定义域和值域，此处的定义域是表示值空间（Rep space），即各种变量的值；陪域是抽象值空间（Abstract space），即客户端得到的抽象对象
定义域有约束条件，即有效定义域，由表示不变性（Rep invariant）约束，即限定条件要求始终为真
从Rs到As的映射方法即抽象函数（Abstract function），这个映射应该满足以下条件：
1. 在有效定义域上为满射，即所有抽象空间的元素都应当被覆盖
2. 作为一个函数，满足右唯一性，即每个r只能对应一个a

所以抽象类型的实现由RS，AS，RI和AF共同决定，缺少任何一个都不能唯一确定一个抽象类型，抽象类型的设计应该统筹考虑这些因素


### Example: Rational Numbers

这里给出了一个有理数的抽象类型设计：
```java
public class RatNum {

    private final int numer;
    private final int denom;

    // Rep invariant:
    //   denom > 0
    //   numer/denom is in reduced form

    // Abstraction Function:
    //   represents the rational number numer / denom

    /** Make a new Ratnum == n.
     *  @param n value */
    public RatNum(int n) {
        numer = n;
        denom = 1;
        checkRep();
    }

    /** Make a new RatNum == (n / d).
     *  @param n numerator
     *  @param d denominator
     *  @throws ArithmeticException if d == 0 */
    public RatNum(int n, int d) throws ArithmeticException {
        // reduce ratio to lowest terms
        int g = gcd(n, d);
        n = n / g;
        d = d / g;

        // make denominator positive
        if (d < 0) {
            numer = -n;
            denom = -d;
        } else {
            numer = n;
            denom = d;
        }
        checkRep();
    }
}
```

这里的抽象值空间为有理数，表示值空间为两个整数，表达不变式为denom>0且为约数形式，抽象函数为\[n,m]->n/m

#### Checking the Rep Invariant

由于表达不变式贯穿抽象类型的整个生命周期，所以ADT的所有操作都应该检查RI是否为真，即`RatNum`类中的`checkRep()`方法。这种方法应该为私有的，正如其定义中要求的——表达不变式的检查应该由实现者/设计者负责，而不依赖于客户端的行为

#### No Null Values in the Rep

6.005中，前置条件和后置条件都隐式要求对象和数组不为空值，但是ADT的操作中没有方法能隐式的辅助判断空值，比如`s.length()`会自动对空值抛出异常，则应当对空值情况正确退出

## Documenting the AF, RI, and Safety from Rep Exposure

抽象函数和表达不变式都应当通过注释形式记录，除此之外，还应当记录**表达暴露安全性**，即该设计是如何保证不出现表达暴露的

以下为包含所有注释的Tweet抽象类实现：
```java
// Immutable type representing a tweet.
public class Tweet {

    private final String author;
    private final String text;
    private final Date timestamp;

    // Rep invariant:
    //   author is a Twitter username (a nonempty string of letters, digits, underscores)
    //   text.length <= 140
    // Abstraction Function:
    //   represents a tweet posted by author, with content text, at time timestamp 
    // Safety from rep exposure:
    //   All fields are private;
    //   author and text are Strings, so are guaranteed immutable;
    //   timestamp is a mutable Date, so Tweet() constructor and getTimestamp() 
    //        make defensive copies to avoid sharing the rep's Date object with clients.

    // Operations (specs and method bodies omitted to save space)
    public Tweet(String author, String text, Date timestamp) { ... }
    public String getAuthor() { ... }
    public String getText() { ... }
    public Date getTimestamp() { ... }
}
```

还有RatNum的示例：
```java
// Immutable type representing a rational number.
public class RatNum {
    private final int numer;
    private final int denom;

    // Rep invariant:
    //   denom > 0
    //   numer/denom is in reduced form, i.e. gcd(|numer|,denom) = 1
    // Abstraction Function:
    //   represents the rational number numer / denom
    // Safety from rep exposure:
    //   All fields are private, and all types in the rep are immutable.

    // Operations (specs and method bodies omitted to save space)
    public RatNum(int n) { ... }
    public RatNum(int n, int d) throws ArithmeticException { ... }
    ...
}
```
由于其表示皆为不可变类型，所以表达暴露安全的证明非常简单

### How to Establish Invariants

为了保证不变式为真，需要满足两个条件：
1. 初始状态下不变式为真：即创建和生产操作不改变不变式的值
2. 所有变更应该保持不变式为真：即观察和变更操作不改变不变式的值，且没有发生表达暴露，即不发生客户端变更
## ADT invariants replace preconditions

ADT表达不变式还可以用来将人治变为法治，即将前置条件变为不变式约束，由编译器检查

ADT设计方法：对于方法的前置条件，提取表达值空间，实现构造、观察操作以及不变式检查方法，并写下注释
替换方法：使用ADT作为参数类型，将前置条件约束从spec转移到ADT约束

使用ADT而不是前置条件的好处：
1. 不依赖客户端的行为
2. 自动检查

## Summary

在上一章[[Reading12 Abstract Data Types]]中规定了ADT是由操作定义的，本章主要讨论了ADT表示的设计方法和底层逻辑，包括什么是表达不变性和如何维护表达不变性、什么是表达值空间、抽象函数、抽象值空间和表达不变式、他们是如何确定一个ADT的，在结尾补充了ADT的一个使用案例：替代前置条件