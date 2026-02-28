---
created: 2026-02-27 11:36
updated: 2026-02-28T21:05
status: Completed
topics: definition;seperation of implement and interface;Why interface
---
>[!SUMMARY] Table of Contents
>    - [[Reading14 Interface#Interfaces|Interfaces]]
>    - [[Reading14 Interface#Subtypes|Subtypes]]
>    - [[Reading14 Interface#Example: `MyString`|Example: `MyString`]]
>        - [[Reading14 Interface#Constructor for Interface|Constructor for Interface]]
>    - [[Reading14 Interface#Generic Interface|Generic Interface]]
>    - [[Reading14 Interface#Why Interfaces?|Why Interfaces?]]
>    - [[Reading14 Interface#Summary|Summary]]
## Interfaces

interface是Java表达ADT的语言机制，其中只能包含方法签名，不能有方法体或者成员变量，接口不能作为实例

接口有以下优点：
1. 为客户端提供契约，并将实现和抽象数据类型完全隔离
2. 同一个程序中可以共存ADT的多种表达形式

## Subtypes

实现interface A的类B被称为A的子类，表现在B必须遵从A的所有规范，即B的规范强度应该大于等于A的。

编译器要求B必须实现A中的所有方法，但是不能检查B的实现是否符合A的规范，所以还需要人来检查

## Example: `MyString`

给出了一个接口类MyString以及两个子类的实现：
```java
/** MyString represents an immutable sequence of characters. */
public interface MyString { 

    // We'll skip this creator operation for now
    // /** @param b a boolean value
    //  *  @return string representation of b, either "true" or "false" */
    // public static MyString valueOf(boolean b) { ... }

    /** @return number of characters in this string */
    public int length();

    /** @param i character position (requires 0 <= i < string length)
     *  @return character at position i */
    public char charAt(int i);

    /** Get the substring between start (inclusive) and end (exclusive).
     *  @param start starting index
     *  @param end ending index.  Requires 0 <= start <= end <= string length.
     *  @return string consisting of charAt(start)...charAt(end-1) */
    public MyString substring(int start, int end);
}
```

子类FastMyString：
```java
public class FastMyString implements MyString {

    private char[] a;
    private int start;
    private int end;

    /* Create an uninitialized FastMyString. */
    private FastMyString() {}

    /** Create a string representation of b, either "true" or "false".
     *  @param b a boolean value */
    public FastMyString(boolean b) {
        a = b ? new char[] { 't', 'r', 'u', 'e' } 
              : new char[] { 'f', 'a', 'l', 's', 'e' };
        start = 0;
        end = a.length;
    }

    @Override public int length() { return end - start; }

    @Override public char charAt(int i) { return a[start + i]; }

    @Override public MyString substring(int start, int end) {
        FastMyString that = new FastMyString();
        that.a = this.a;
        that.start = this.start + start;
        that.end = this.start + end;
        return that;
    }
}
```

子类SimpleMyString:
```java
public class SimpleMyString implements MyString {

    private char[] a;

    /* Create an uninitialized SimpleMyString. */
    private SimpleMyString() {}

    /** Create a string representation of b, either "true" or "false".
     *  @param b a boolean value */
    public SimpleMyString(boolean b) {
        a = b ? new char[] { 't', 'r', 'u', 'e' } 
              : new char[] { 'f', 'a', 'l', 's', 'e' };
    }

    @Override public int length() { return a.length; }

    @Override public char charAt(int i) { return a[i]; }

    @Override public MyString substring(int start, int end) {
        SimpleMyString that = new SimpleMyString();
        that.a = new char[end - start];
        System.arraycopy(this.a, start, that.a, 0, end - start);
        return that;
    }
}
```

`@Override`告诉编译器该方法的签名应与接口完全一致，不过编译器会自动检查是否实现接口的所有方法，这里仅起到解释作用

### Constructor for Interface

以下是一个使用该抽象类的示例：
```java
MyString s = new FastMyString(true);
System.out.println("The first character is: " + s.charAt(0));
```

该示例存在一个问题：要求用户理解子类构造方法的spec，这违背了接口的初衷——隔离实现和接口。
由于接口中不能包含构造方法（不能创建接口实例），而子类的构造方法的spec又不应该在抽象类中给出，所以应该用静态工厂方法解决
以下是使用静态工厂方法的解决方案：
```java
/** @param b a boolean value
     *  @return string representation of b, either "true" or "false" */
    public static MyString valueOf(boolean b) {
        return new FastMyString(true);
    }
```

## Generic Interface

泛型接口中的E作为占位符，不做规定；在子类中做限定，限定为某种类型，或者某些类型的集合，或另一抽象类型，或者不做限定

```java
public interface Set<E> {};

public class CharSet1 implements Set<Character> {};
```

此处子类CharSet1将参数类型限定为Character。但是子类并不一定要限定泛型类型，可以沿用占位符E，由编译器自动识别，前提是其实现适配所有对象类型，如HashSet：
```java
public class HashSet<E> implements Set<E>
```

## Why Interfaces?

**六大理由**支持使用接口：
1. Documentation for both the compiler and for humans：对于人类而言，接口可以看作一份文档，而且该文档完全与实现相隔离，**阅读成本很低**；对于编译器而言，接口可以看作一种契约，可以提供**静态检查**，检查实现是否合法以及调用方法是否合法
2. Allowing performance trade-offs：使用接口让替换成本局限于很小的一个部分，使客户端代码的**修改成本降低**,更容易实现性能权衡。比如`list`接口下的两个子类，如`linkedList`和`ArrayList`，对不同操作的表现不同，如果不作为list的子类，在客户端想换一个类型时需要修改所有调用方法；如果创建list变量，则只需要在creator部分new另一个子类即可。
3. Optional methods：可选方法，指接口中一些mutator不强制要求子类实现，这使得实现代码的**复用性增强**。比如我希望`list`接口中有add方法和其他很多observer，并拥有可变列表和不可变列表两种子类，但是这时候显然会出现冲突，不可变列表无法实现add方法，导致其不能成为list的子类，使很多相似甚至完全相同的observer不得不重复出现；所以`list`将所有mutator都定义为可选方法，以实现对observer、creator等操作对不同列表类型子类的复用，对mutator操作抛出异常,`throw new UnsupportedOperationException();`
4. Methods with intentionally underdetermined specifications:通过在接口的规范中留白，来保留实现类的**灵活度**。
5. Multiple views of one class：一个类可以作为多个接口的实现，拥有**多重身份**，并成为多个功能组合的**复杂组件**。
6. More and less trustworthy implementations ：可以为同一接口写快而不稳的版本和慢而稳的实现，或者将一个实现作为另一个实现的上一个已确认正确的版本，以实现**渐进式编程**，进行**风险控制**。由于可以给多个类似实现写同一个测试套件，也可以提高**测试套件的复用率**。

## Summary

interface是一种JavaADT语法结构，隔离了定义和实现，实现了表示独立性和多态复用。接口的优点参考[[Reading14 Interface#Why Interfaces?]](本章其实不只是对前文ADT优点的描述，其中还涉及了一些Java接口设计逻辑，比如multiple implement和optional methods，可以作为拓展）,逻辑参考[[Reading13 Abstraction Functions & Rep Invariants]]