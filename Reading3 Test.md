---
created: 2026-02-11 11:58
updated: 2026-02-13T10:00
status: Completed
topics: Test-first Programming; Test Suite ; Unit test vs. Integration test ; Automatically Regression Test
---
>[!SUMMARY] Table of Contents
>    - [[Reading3 Test#Objectives for Today’s Class|Objectives for Today’s Class]]
>    - [[Reading3 Test#Validation|Validation]]
>        - [[Reading3 Test#Testing is Hard|Testing is Hard]]
>        - [[Reading3 Test#Test Attitude：Putting on Your Testing Hat|Test Attitude：Putting on Your Testing Hat]]
>    - [[Reading3 Test#When Making Test：Test-first Programming|When Making Test：Test-first Programming]]
>        - [[Reading3 Test#Specification|Specification]]
>    - [[Reading3 Test#Test Case Construction:Choosing Test Cases by Partitioning|Test Case Construction:Choosing Test Cases by Partitioning]]
>            - [[Reading3 Test#Example: `BigInteger.multiply()`|Example: `BigInteger.multiply()`]]
>            - [[Reading3 Test#Example: `max()`|Example: `max()`]]
>        - [[Reading3 Test#Include Boundaries in the Partition|Include Boundaries in the Partition]]
>        - [[Reading3 Test#Two Extremes for Covering the Partition|Two Extremes for Covering the Partition]]
>    - [[Reading3 Test#Blackbox and Whitebox Testing：Before or After Implements|Blackbox and Whitebox Testing：Before or After Implements]]
>    - [[Reading3 Test#Good Test Habit：Documenting Your Testing Strategy|Good Test Habit：Documenting Your Testing Strategy]]
>    - [[Reading3 Test#Test Evaluation：Coverage|Test Evaluation：Coverage]]
>    - [[Reading3 Test#Part and Whole：Unit Testing and Stubs|Part and Whole：Unit Testing and Stubs]]
>        - [[Reading3 Test#Stub|Stub]]
>    - [[Reading3 Test#Development of Test Suite：Automated Testing and Regression Testing|Development of Test Suite：Automated Testing and Regression Testing]]
## Objectives for Today’s Class

1. 理解测试的价值，了解test-first programming的过程
2. 能够通过划分输入和输出空间，和选择良好测试用例来设计一套测试套件（test suite，一组测试方法的组合）
3. 能够通过计算代码覆盖率(code coveage)来判断测试套件的效果
4. 正确选择黑盒测试/白盒测试(white box)，单元测试/集成测试(integration test)和自动化回归测试(automated regression test)

## Validation

验证的目的是检验程序的正确性，而测试是验证的一个部分

验证包含以下几个部分：
1. 形式推理（Formal reasoning）：也称为varification，指构建一个形式证明，来证明程序的正确性。只能在局部关键程序使用，如调度器、文件系统等
2. 代码审查（Code review）：让别人来仔细阅读代码
3. 测试（Testing）：构建测试套件检验结果

实现完美质量是极其困难的，以下为几种典型的残留缺陷率(residual defect rate，单位为defects/kloc，每千行缺陷数)：
1. 1-10 一般的软件代码
2. 0.1-1 高质量代码，Java库大概在这个水平
3. 0.01-0.1 非常高质、安全的代码，NASA和Praxis也许能达到这个水平

### Testing is Hard

有些方法无法在软件测试中使用：
1. 穷举测试（Exhaustive Testing）：测试所有可能输入和对应输出
2. 随意测试（Haphazard Testing）：除非程序的bug多到随便一个输入都会报错，否则随意测试不太可能找到问题所在
3. 抽样测试（Random Testing）：软件bug的出现对于输入不具备连续性

### Test Attitude：Putting on Your Testing Hat

做测试的时候要摆脱coding心态，作为测试者，需要猛攻每个可能出错的点，而不是“让刚写的代码能跑就行”

## When Making Test：Test-first Programming

尽早测试，甚至在写具体代码前先写测试，这里给出了test-first programming的顺序：
1. 写方法的规范（specification）
2. 根据规范写测试用例
3. 写具体代码，如果通过测试用例则说明功能完备

### Specification

规范是指对方法的输入输出行为的描述，包括输入参数的类型和限定条件，返回值的类型，返回值和输入参数之间的关系。
具体在代码中，规范由方法签名和注释组成，以ps0中的`calculateHeadingToPoint`方法为例：
```java
/**
     * Given the current direction, current location, and a target location,
     * calculate the heading towards the target point.
     * 
     * The return value is the angle input to turn() that would point the turtle in
     * the direction of the target point (targetX,targetY), given that the turtle is
     * already at the point (currentX,currentY) and is facing at angle
     * currentHeading. The angle must be expressed in degrees, where 0 <= angle <
     * 360.
     *
     * HINT: look at http://en.wikipedia.org/wiki/Atan2 and Java's math libraries
     * 
     * @param currentHeading current direction as clockwise from north
     * @param currentX       current location x-coordinate
     * @param currentY       current location y-coordinate
     * @param targetX        target point x-coordinate
     * @param targetY        target point y-coordinate
     * @return adjustment to heading (right turn amount) to get to target point,
     *         must be 0 <= angle < 360
     */
    public static double calculateHeadingToPoint(double currentHeading, int currentX, int currentY, int targetX,
            int targetY) {
        return (90 - Math.toDegrees(Math.atan2(targetY - currentY, targetX - currentX)) - currentHeading + 360) % 360;
    }
```

## Test Case Construction:Choosing Test Cases by Partitioning

测试套件（Test suite）由一组测试用例（Test Case）组成，测试用例既要需要足够小来保证快速运行，又要足够大来验证功能
为了选择测试用例，我们将输入空间划分（Partitioning）为一个个子域（Subdomain），每个子域包含一组输入，这样从每个子域中选取一个测试用例，就得到了测试套件
在划分输入空间时，我们的依据是相似输入和相似行为，这种划分方式通过分隔子集+选取代表的方式，最大化的利用了测试资源

#### Example: `BigInteger.multiply()`

大数乘法`multiply`是Java库内置的方法，其规范如下：
```java
/**
 * @param val another BigIntger
 * @return a BigInteger whose value is (this * val).
 */
public BigInteger multiply(BigInteger val)
```
虽然这个方法虽然明面上的参数只有一个，实际上在理解输入空间的时候，应该理解为两个大数。那么输入空间可以理解为二维空间，横纵坐标分别为两个大数值。

以下是对二维空间的划分过程：
1. 乘法的逻辑划分：显然可以划分为四个象限，即正负值的划分
2. 乘法划分的边界：四象限的边界条件显然为x或y等于0的情况
3. 考虑BigInteger的设定边界：考虑到long以内的加法可以复用基本的整数乘法，所以需要测试long以外的大数的结果

这样的话，对于每个参数，都有0，小负整数，小正整数，极大正整数，极小负整数5种情况，那么二维输入空间就可以分为25个子域，在每个子域中选取一对参数值即可
>Reading中给出的划分方法考虑了1，-1，是因为猜想在这时可能算法逻辑为`return this`，与其他值行为不同，应该区分出来

以下为划分结果示意图，黑点为在每个子域中取的测试用例：

![[Pasted image 20260212104233.png]]

#### Example: `max()`

取最大值的max方法，其实现如下：
```java
/**
 * @param a  an argument
 * @param b  another argument
 * @return the larger of a and b.
 */
public static int max(int a, int b)
```

与multiply方法一致，容易判断输入空间是二维有界空间，其中两个输入为int边界内的整数值，对输入空间做如下划分：
1. 最大值的逻辑划分：a>b|a<b|
2. 逻辑划分的边界：a=b

所以可以将输入空间划分为上述三个子域，测试套件中只需要在每个子域取一个测试用例即可

### Include Boundaries in the Partition

边界测试相当重要，这是因为语言的底层设计（整数溢出）或者程序行为在边界处可能产生突变，所以可以将输入参数的边界作为单独维度分析
如上面的`max`方法中，可以将a分为0，负整数，正整数，最小整数，最大整数，b同理，所以a和b作为划分维度可以在内部分为五部分

### Two Extremes for Covering the Partition

在划分了输入空间了之后，测试用例的数量存在上下限，分别作为最小测试和最大测试：

1. 完全笛卡尔积（Full Cartesian Product）：将每个划分维度的次数相乘。如`max`中对两个参数分别划分为5个部分，对两个参数之间的关系划分为3个部分，理论上应该可以将输入空间划分为75个子域，当然参数内部划分和参数间划分这两个维度之间会有交叉，比如不可能同时a>b,a=0,b=0,使实际划分小于理论数量
2. 维度完全覆盖（Cover Each Part）：保证每个维度的每个部分都至少有一个测试用例覆盖，但不保证每个子域都对应一个测试用例

>完全笛卡尔积满足对划分区域的全覆盖，是最大测试，测试用例数为所有维度乘积；维度完全覆盖只满足对维度投影的全覆盖，是最小覆盖，测试用例数为维度划分数量的最大值

## Blackbox and Whitebox Testing：Before or After Implements

黑盒测试和白盒测试是两种测试策略，前者在写完规范就写测试套件，后者在写完方法实现后再写测试套件，区别就在于当方法实现中对于不同参数范围/组合采取不同行为时，白盒测试可能根据方法内部的选择增加一个维度

需要注意的是，白盒测试虽然知道代码实现的具体细节，但是仍然应该对齐Spec的模糊度，比如规范中说这时要抛出一个异常，而实现中抛出了一个具体异常`MalformedInputException`，此时白盒测试只需要保证异常抛出即可，不要求为具体异常。

>白盒测试蕴含的测试哲学：只要跟着大纲走，具体实现随便做。允许实现在遵循spec的前提下自由改变和拓展

## Good Test Habit：Documenting Your Testing Strategy

在测试类上写测试策略，对于每个测试用例也要写注释，注明cover了哪些维度的哪些部分

测试策略文档如下：
```java
/*
 * Testing strategy
 *
 * Partition the inputs as follows:
 * text.length(): 0, 1, > 1
 * start:         0, 1, 1 < start < text.length(),
 *                text.length() - 1, text.length()
 * text.length()-start: 0, 1, even > 1, odd > 1
 *
 * Include even- and odd-length reversals because
 * only odd has a middle element that doesn't move.
 *
 * Exhaustive Cartesian coverage of partitions.
 */
```

测试用例文档如下：
```java
// covers test.length() = 0,
//        start = 0 = text.length(),
//        text.length()-start = 0
@Test public void testEmpty() {
    assertEquals("", reverseEnd("", 0));
}
```
## Test Evaluation：Coverage

对于白盒测试，需要评判其覆盖率，有三种评判标准：
1. 语句覆盖（Statement Coverage）
2. 分支覆盖（Branch Coverage）：每个判断语句的真假分支都要走过
3. 路径覆盖（Path Coverage）：所有路径都要走过

> 看上去分支覆盖和路径覆盖好像一致，区别在于判断语句串联时（即没有嵌套关系时），路径覆盖走的路径比分支覆盖走的多；有嵌套关系时一致
> 所以路径覆盖>分支覆盖>语句覆盖

Eclipse中内置一个覆盖率检查工具[EclEmma - Java Code Coverage for Eclipse](https://www.eclemma.org/),覆盖率模式包括instruction，line，branch等

## Part and Whole：Unit Testing and Stubs

前文的[[Reading3 Test#Choosing Test Cases by Partitioning|Choosing Test Cases by Partitioning]]实际上是为程序的某个模块（指某个类或者方法）设计测试用例（或者说模块级测试套件）的方法，这种局部的测试被称为单元测试；
与单元测试相对的是集成测试，它测试模块组合甚至整个程序。两种测试各有优势，单元测试保证每个模块的正确性，而集成测试验证模块之间的正确链接。

Reading中举了一个例子，有三个方法：`makeIndex()`，`getWebPage()`和`extractWords()`，其中makeIndex依赖于后两个方法。在测试这三个方法的时候，理论上应该先做后两个方法的单元测试，再做三个模块的集成测试。但是实际开发过程中可能并不如此，比如后两个方法的实现可能是并发进行的，此时仍属于黑盒测试，所以引入了桩程序（Stub）

### Stub

为什么要使用桩程序：
1. 当被测方法的某个依赖方法尚未实现时
2. 当只想考虑被测方法的逻辑，想隔离其依赖方法的影响时
3. 当被测方法的依赖方法过于复杂或不稳定时，比如依赖方法需要联网或者规模极大
4. 当被测方法的边界很难通过真实的底层方法达到时，通过桩程序模拟底层方法达到上层方法的边界

以`makeIndex()`为例，有桩程序的测试流程如下：
1. 为两个底层方法`getWebPage()`和`extractWords()`做单元测试
2. 给`getWebPage()`写桩程序，测试`makeIndex()`的逻辑
3. 给`makeIndex()`写完整的集成测试，使用真实的`getWebPage()`方法

## Development of Test Suite：Automated Testing and Regression Testing

自动化测试：由测试驱动程序（Test Driver）自动完成测试运行并生成结果报告

有了自动化测试，当修改了程序后，需要重新运行测试驱动程序，来防止回归现象（Regression，指修复bug或增加功能时引入新bug）的出现。
很多时候bug是由非测试手段发现的，这个引发错误的输入没有被当前的测试用例考虑到，所以要为这个bug新写一个单独的测试用例，这个测试用例被称为回归测试。
将回归测试加入测试套件不仅防止回归现象，还能保证回退版本时重新引入错误

自动化测试和回归测试共同构成了测试套件的发展过程，在自动化测试-回归测试-自动化测试的过程中测试套件不断丰富和完善

## Summary

本节我们学习了测试的概念，了解了如何为一个模块构造模块级测试套件，并用单元测试和集成测试将模块级测试套件组装为程序级测试套件。当然构造测试套件的过程不可能一劳永逸，在增加功能和遇到意料外的bug时，需要通过增加回归测试来补充测试套件，而这一过程需要由自动化测试来辅助。