---
created: 2026-02-18 19:52
updated: 2026-02-23T20:16
status: Completed
topics: identify and assess nondeterminism；declarative vs. operational ；compare spec strength
---
>[!SUMMARY] Table of Contents
>    - [[Reading7 Designing Specifications#Introduction|Introduction]]
>    - [[Reading7 Designing Specifications#Deterministic vs. underdetermined specs|Deterministic vs. underdetermined specs]]
>        - [[Reading7 Designing Specifications#Nondeterministic vs Undetermined|Nondeterministic vs Undetermined]]
>    - [[Reading7 Designing Specifications#Declarative vs. operational specs|Declarative vs. operational specs]]
>    - [[Reading7 Designing Specifications#Stronger vs. weaker specs|Stronger vs. weaker specs]]
>        - [[Reading7 Designing Specifications#Diagramming specifications|Diagramming specifications]]
>    - [[Reading7 Designing Specifications#Designing good specifications|Designing good specifications]]
>        - [[Reading7 Designing Specifications#The specification should be coherent|The specification should be coherent]]
>        - [[Reading7 Designing Specifications#The results of a call should be informative|The results of a call should be informative]]
>        - [[Reading7 Designing Specifications#The specification should be strong enough|The specification should be strong enough]]
>        - [[Reading7 Designing Specifications#The specification should also be weak enough|The specification should also be weak enough]]
>        - [[Reading7 Designing Specifications#The specification should use _abstract types_ where possible|The specification should use _abstract types_ where possible]]
>        - [[Reading7 Designing Specifications#Precondition or postcondition?|Precondition or postcondition?]]
>            - [[Reading7 Designing Specifications#About access control|About access control]]
>            - [[Reading7 Designing Specifications#About static vs. instance methods|About static vs. instance methods]]
## Introduction

对于相似的行为，我们可以写出不同的规范，这些规范的差异主要是以下三个维度：
1. 确定性：是否定义了唯一合法的输出
2. 声明性：是否明确声明输出的计算方法
3. 强度：是否包含大量合法实现

## Deterministic vs. underdetermined specs

如intro所言，确定规范规定了满足requires的前提条件下，结果唯一可确定；
相应的，待定规范即使满足了前提条件，结果也不一定确定

```java
static int findExactlyOne(int[] arr, int val)
  requires: val occurs exactly once in arr
  effects:  returns index i such that arr[i] = val
```
对于这个例子，结果唯一可确定，因为前置条件确保了有且只有一个对应元素
以下是它的两个合法实现：
```java
static int findFirst(int[] arr, int val) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == val) return i;
    }
    return arr.length;
}
static int findLast(int[] arr, int val) {
    for (int i = arr.length - 1 ; i >= 0; i--) {
        if (arr[i] == val) return i;
    }
    return -1;
}
```

```java
static int findOneOrMore,AnyIndex(int[] arr, int val)
  requires: val occurs in arr
  effects:  returns index i such that arr[i] = val
```
而对于这个规范，由于前提条件仅规定存在该值元素，所以返回的索引不确定

### Nondeterministic vs Undetermined

需要明确的是，这里的待确定性（Undetermined）不同于一般意义上的不确定性（Nondeterministic）。后者一般意味着一段代码运行多次，所得到的结果不同；而前者表示同一个输入有多个合法输出
> 不确定性的示例：伪随机或者并发访问

进一步而言，待确定规范可以有一个完全确定的实现，即其实现对于一个确定的输入可以永远只有同一个确定的输出

## Declarative vs. operational specs

声明性规范只给出结果的属性及其与初始状态的关系
操作性规范给出方法执行的伪代码步骤

声明性规范几乎在所有常见下都是首选项，因为他保证了设计者、实现者和使用者的隔离，如果为了方便维护，在方法内部写注释比使用操作性规范更合适

## Stronger vs. weaker specs

强的规范指更弱的前置条件和更强的后置条件，即要求更少而承诺更多。

对于实现的替换：满足强规范的实现总是能满足弱规范。
对于规范的替换：对实现者而言，弱规范直接替代强规范是轻松的，因为绝对不需要修改实现；但是对于客户是毁灭性的，因为要求更多会导致之前传入的参数违法，承诺更少会导致返回结果不符合之前预期
所以对于设计者而言，如果需要迭代软件，总是要用更强的规范替代更弱的规范，并相应修改实现使其满足规范

### Diagramming specifications

对强弱规范和实现的图形想象：实现为一个点，规范为一个范围，强实现的范围为弱实现的范围的子集，点在范围中表示实现符合规范

## Designing good specifications

设计好的规范是得到好的方法的前提

规范的格式应该简洁、易懂、结构良好，而内容并没有明确的规定，不过有几个惯例

### The specification should be coherent

这里的意思是规范应该具有单元性，如果一个规范中处理的逻辑较为复杂，则应该拆分为多个规范

### The results of a call should be informative

返回值应当无歧义，尤其是-1或者null值

### The specification should be strong enough

规范应该足够强，向使用者提供更明确的保证

### The specification should also be weak enough

规范也不能太强，给实现者留下改进的空间

### The specification should use _abstract types_ where possible

规范应当使用抽象数据类型，这也是给实现则会留下改进空间的一部分

### Precondition or postcondition?

当输入不合法时，使用前置条件约束，还是使用后置条件兜底，是一个工程决策的问题

对于使用者而言，使用后置条件的方法更便于调试，因为会返回明确的错误情况，而不是带着未知错误继续运行
对于实现者而言，使用前置条件更方便，因为不用写异常处理的各种内容，只需要相信使用者会遵守前置条件

需要考虑方法的使用范围和检查成本，对于私有、检查成本高的方法可以使用前置条件，否则优先考虑后置条件

#### About access control

上文中提到了使用范围对规范的影响，之前的方法大多为public，但是其实可以发现public方法会带来许多问题，比如接口杂乱、难以调试，所以在后文中会提及保持内部私有的优势

#### About static vs. instance methods

先前的方法基本都是静态类型的，但是也应该学会使用实例方法，比如本文中反复出现的find方法就可以变为intArray类的一个实例方法，传入的int\[]参数从其所在实例中读取即可

## Summary

本文中介绍了三种衡量规范的维度，即确定性、声明性和强弱性。对于设计规范，本文提到了一些惯例，主要是规范的强弱应该适中，并优先选择后置条件而不是前置条件来处理不合法输入，以兼顾用户便捷性和实现灵活性。其实可以把设计规范看作几个矛盾的平衡，比如使用者友好、灵活性、性能之间的平衡，强弱的平衡等，良好的设计规范应该能取得一个平衡点。
