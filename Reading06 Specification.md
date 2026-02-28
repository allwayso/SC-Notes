---
created: 2026-02-13 21:06
updated: 2026-02-28T21:33
status: Completed
topics: Specification；Test against spec ；Exception
---
>[!SUMMARY] Table of Contents
>- [[Reading6 Specification#Specification |Specification ]]
>    - [[Reading6 Specification#Why Specification|Why Specification]]
>        - [[Reading6 Specification#Behavioral equivalence|Behavioral equivalence]]
>    - [[Reading6 Specification#Specification content|Specification content]]
>        - [[Reading6 Specification#Specification in Java|Specification in Java]]
>        - [[Reading6 Specification#What a specification may talk about|What a specification may talk about]]
>    - [[Reading6 Specification#Testing and specifications|Testing and specifications]]
>        - [[Reading6 Specification#Testing Units|Testing Units]]
>    - [[Reading6 Specification#Specifications for mutating methods|Specifications for mutating methods]]
>    - [[Reading6 Specification#Summary for Specification|Summary for Specification]]
>- [[Reading6 Specification#Exceptions|Exceptions]]
>    - [[Reading6 Specification#Exception Usage|Exception Usage]]
>        - [[Reading6 Specification#Exceptions for signaling bugs|Exceptions for signaling bugs]]
>        - [[Reading6 Specification#Exceptions for special results|Exceptions for special results]]
>    - [[Reading6 Specification#Exception Categories|Exception Categories]]
>        - [[Reading6 Specification#Checked Exceptions|Checked Exceptions]]
>        - [[Reading6 Specification#Unchecked Exceptions|Unchecked Exceptions]]
>        - [[Reading6 Specification#Throwable hierarchy|Throwable hierarchy]]
>    - [[Reading6 Specification#Exception design consideration|Exception design consideration]]
>- [[Reading6 Specification#Summary|Summary]]
# Specification 

## Why Specification

1. 使用者友好：使用者不需要知道内部实现，只需要关注规范中规定的输入和输出情况即可，降低理解成本
2. 实现者友好：实现者不需要关注使用者的使用方式，只要遵守规范，可以相对自由的改变实现

也就是说规范的作用是将使用者和实现者解耦

### Behavioral equivalence

上文中提到，“*只要遵守规范，可以相对自由的改变实现* ”,实际上就是说两个行为等价的实现可以相互替代
这里的行为等价，指的是对用户来说等价，而不是实现方式相同；而用户使用方法是依赖规范的，所以行为等价就是两个实现都符合规范

## Specification content

简而言之，规范由前置条件和后置条件构成，前置条件（Precondition）是用户需要遵循的条件，后置条件（Postcondition）是实现者承诺的行为；前置条件通过关键词`requires:`约束，后置条件通过关键词`effects:`约束

### Specification in Java

见[[Reading4 Code Review#Comments where needs]]，这里的意思是要把前置条件尽可能放在`@param`里，把后置条件放在`@return`和`@throw`里

Eclipse根据规范自动生成Javadoc文档，见[generate Javadoc](https://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2Freference%2Fref-export-javadoc.htm)

### What a specification may talk about

规范不应该谈及任何实现的局部变量或者代码段，这些部分应该被视为对使用者不可见

## Testing and specifications

这里谈论的主要是基于黑盒测试，但是即使是对于白盒测试，测试也必须基于规范

对于白盒测试，已知实现可能强于规范，或者存在一些规范中未定义的行为，但是测试不应依赖于这些行为，这句话的意思是：
1. 设计测试用例参考实现：比如对于代码段`if(x>100)`，想到用输入值101作为测试用例
2. 断言必须依赖规范：当测试用例的代码逻辑于规范中的设定冲突时，以规范为标准

>这是因为：1.避免同义重复（Tautology）：如果实现逻辑错误，而测试按照实现逻辑判断，则结果永远正确；
>2.测试的复用性：当代码段替换或者优化时，白盒测试仍然应该忠实反映代码是否符合规范，而不是反映代码逻辑是否与先前一致

### Testing Units

[[Reading3 Test#Part and Whole：Unit Testing and Stubs]]中给出了一个示例，并提到单元测试应该相互独立，即一个方法的测试不应该由于另一方法的实现错误而失败。对于本章而言，这就意味着：一个好的单元测试只关注单一的规范

对于集成测试，Reading3中提到它用于测试单元之间的正确配合；对于本章而言，这就意味着：一个好的集成测试确保不同方法的规范之间相互兼容，即不同方法按照对方期望的方式传递和返回值

但是集成测试不能代替单元测试，因为a方法的依赖方法b的返回值不一定覆盖了a的输入空间，即对单一方法的测试不完备；所以集成测试要在单元测试的基础上进行

## Specifications for mutating methods

除非规范的后置条件中有说明，否则默认不得改变传入的可变参数

## Summary for Specification

这里的规范与前面的[[Reading4 Code Review#Comments where needs]]和[[Reading3 Test#Specification]]部分多有重复，不过着重强调了两个关系，即实现和规范的关系，规范和测试的关系。具体而言，就是实现要在规范下实现，使用和实现解耦；测试要以规范为判据，预期结果应独立于实现。总的而言，强调了规范的至高地位。

# Exceptions

异常是规范的一部分，由于其十分重要，将其单开一节

## Exception Usage

### Exceptions for signaling bugs

异常的一个作用是抛出一些信息帮助解决bug，比如先前提到的`ArrayIndex­OutOfBounds` and `Null­Pointer­Exception`分别表示超出容器有效范围和调用空引用方法。

### Exceptions for special results

异常的另一个作用是处理特殊结果。

比如对于一个int类型方法，只能处理正整数，在输入为负数时返回-1，接收该方法输出的函数会对-1做特殊处理。但是对于字符串，或者其他返回值而言，所用的-1等特殊值可能不够特殊，或者存在某种疏漏，比如[千年虫问题](https://en.wikipedia.org/wiki/Year_2000_problem)就有一部分是因为设计者认为世纪末该代码不会再被使用，使用99/9/9作为特殊值而导致。

所以对于特殊结果，采用异常更合理

## Exception Categories

### Checked Exceptions

所谓的检查异常，指的是该异常会被编译器检查：
1. 如果一个方法可能抛出某个检查异常，则这个异常必须在声明中写出
2. 如果一个方法调用了另一个可能抛出异常的方法，要么处理这个异常，要么声明这个异常，此时该异常将被向上传播到上级方法

这确保了抛出的异常都会被解决，所以检查异常一般用于处理特殊结果。

### Unchecked Exceptions

所谓的非检查异常，指的是该异常不会被编译器检查是否声明或者是否解决，一旦出现直接崩溃，这是因为：

1. 对于`NullPointerException`等常见异常，如果都在方法签名中声明，签名部分将会非常臃肿
2. 这些异常是原则上可以避免的

所以非检查异常用于处理bug

### Throwable hierarchy

为了理解Java如何划分检查异常和非检查异常，我们展开Throwable异常类结构：

![[Pasted image 20260215224730.png]]
如[Throwable 官方文档](https://docs.oracle.com/javase/8/docs/api/?java/lang/Throwable.html)表述，`Throwable` and any subclass of `Throwable` that is not also a subclass of either [`RuntimeException`](https://docs.oracle.com/javase/8/docs/api/java/lang/RuntimeException.html "class in java.lang") or [`Error`](https://docs.oracle.com/javase/8/docs/api/java/lang/Error.html "class in java.lang") are regarded as checked exceptions，即Error和RuntimeException的子类为非检查异常，其余都为检查异常

## Exception design consideration

上文中说检查异常用于处理特殊结果，非检查异常用于处理bug，这个规则对工程而言不够准确。

以下给出了一个使用不适合的异常类型导致的问题：
对于队列queue的pop方法，设计者认为队空异常应该被视为检查异常，但是使用者在某个方法中，在调用pop方法前，已经用isEmpty方法确认过队列非空，却由于pop方法中throw了一个队空异常，而不得不写try-catch结构满足编译器的要求，即使这一段代码事实上不可能被运行到

所以以下是更精细的规则：
如果你预料用户能用简单的方式确保异常不会发生，且异常是意外失败，那么就应该使用非检查异常；否则则应使用检查异常

给出了几个例子：
1. `Url.getWebPage()` 在无法获取网页时抛出一个检查的 `IOException` ，因为调用者不容易阻止这种情况。
2. `int integerSquareRoot(int x)` 在 `x` 没有整数的平方根时抛出一个检查的 `Not­Perfect­Square­Exception` ，因为测试 `x` 是否为完全平方数与找到实际的平方根一样困难，所以预期调用者无法阻止它

所以由于异常设计较为复杂，有时以null值作为特殊情况返回值是可以接受的，但是应该满足严格规范和谨慎使用

# Summary

规范充当过程实现者和其客户端之间至关重要的防火墙。它使分离开发成为可能：客户端可以自由地编写使用过程的代码，而无需查看其源代码；实现者可以自由地编写实现过程的代码，而无需知道它将如何被使用。





