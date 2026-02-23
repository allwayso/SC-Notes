---
created: 2026-02-10 11:57
updated: 2026-02-11T11:58
status: Updating
---
>[!SUMMARY] Table of Contents
>- [[ProblemSet 0 TurtleGraphics#PartⅠ: Set Up and Warm up|PartⅠ: Set Up and Warm up]]
>    - [[ProblemSet 0 TurtleGraphics#Problem 1: Initialize the Git Repo|Problem 1: Initialize the Git Repo]]
>    - [[ProblemSet 0 TurtleGraphics#Problem 2 : Warm up with `may­Use­Code­In­Assignment`|Problem 2 : Warm up with `may­Use­Code­In­Assignment`]]
>        - [[ProblemSet 0 TurtleGraphics#Unit Test|Unit Test]]
>            - [[ProblemSet 0 TurtleGraphics#Anatomy of JUnit |Anatomy of JUnit ]]
>            - [[ProblemSet 0 TurtleGraphics#Unit Test in PS0|Unit Test in PS0]]
>    - [[ProblemSet 0 TurtleGraphics#Problem 3 : Git Commit|Problem 3 : Git Commit]]
>- [[ProblemSet 0 TurtleGraphics#Part II  Turtle graphics and the Logo language|Part II  Turtle graphics and the Logo language]]
>    - [[ProblemSet 0 TurtleGraphics#Problem 4: `drawSquare`|Problem 4: `drawSquare`]]
>    - [[ProblemSet 0 TurtleGraphics#Problems 5—10: Polygons and headings|Problems 5—10: Polygons and headings]]
>        - [[ProblemSet 0 TurtleGraphics#Drawing polygons|Drawing polygons]]
>        - [[ProblemSet 0 TurtleGraphics#Calculating headings|Calculating headings]]
>    - [[ProblemSet 0 TurtleGraphics#Problem 11: Personal art|Problem 11: Personal art]]
# PartⅠ: Set Up and Warm up
## Problem 1: Initialize the Git Repo

熟悉problemSet流程：
1. 在下载文件夹中init一个仓库
2. 在Eclipse中import这个仓库

## Problem 2 : Warm up with `may­Use­Code­In­Assignment`

这里只需要完成一个方法的实现即可，比较有意思的是得参考一下给出的参数说明：
既你所用在作业里的代码，如果是你自己写的，那没毛；如果不是自己写的，那么它必须对所有人公开、不是往期作业、注明来源且不为要求实现的部分
```java
public static boolean mayUseCodeInAssignment(boolean writtenByYourself, boolean availableToOthers,
            boolean writtenAsCourseWork, boolean citingYourSource, boolean implementationRequired) {
        return (writtenByYourself ||(availableToOthers && !writtenAsCourseWork && citingYourSource
                && !implementationRequired));
    }
```
### Unit Test

当我们写了一部分代码时，需要通过test来检查逻辑的正确性，但是对于大规模项目来说，肉眼检查(Visual inspection,指凭肉眼观察测试用例是否符合要求)是不可信赖的,这是因为：
1. 脆弱性：人眼容易看漏看错
2. 耗时性：人眼看不过来大量输出
3. 不可扩展：对小部分代码的检查不能扩展到较大规模

所以我们引入JUnit，进行不生成图形的自动化单元测试，以下是设计单元测试的哲学：
将问题分解为最小的、正交的单位，为每个最小单元写单元测试，以允许复杂系统在有序扩展的同时保持正确性

#### Anatomy of JUnit 

JUnit是按照方法编写的，使用`@Test`来标注测试方法，每个测试方法都是一个独立的测试单元

测试单元之间的关系：
1. 独立性：一个测试单元的执行结果不会影响另一个测试单元
2. 无序性：JUnit不保证测试单元的执行顺序
3. 共享生命周期：每个测试单元的生命周期一致，既启动-运行-还原，可以使用钩子方法（Hook Method）来为所有测试方法初始化和还原测试环境

测试方法的声明：
1. 不接受任何参数
2. 不返回任何参数
3. 必须为public类型，否则无法从外部调用
4. 必须有`@Test`注释(Annotation)，用于在编译阶段识别测试方法
#### Unit Test in PS0

运行`RulesOf6005Test`->Test成功，绿色条显示->修改`may­Use­Code­In­Assignment`，使其错误->观察Failure Trace和JUnit栏结果->重新改正方法

重点是观察错误追踪返回的堆栈信息，它会显示被测代码段、Java库和JUnit中的错误行数

## Problem 3 : Git Commit

演示了一遍本地commit的流程

# Part II  Turtle graphics and the Logo language

这一部分要求使用Logo语言中的海龟图形进行一些基本操作，只能使用forward和turn方法

## Problem 4: `drawSquare`

很简单的一步，只需要先直走在右转，重复四次即可
```java
    public static void drawSquare(Turtle turtle, int sideLength) {
        for(int i=0;i<4;i++) {
            turtle.forward(sideLength);
            turtle.turn(90);
        }
    }
```

## Problems 5—10: Polygons and headings

实现五个方法即可
### Drawing polygons

- Implement `calculateRegularPolygonAngle`  
    There’s a simple formula for what the inside angles of a regular polygon should be; try to derive it before googling/binging/duckduckgoing.  
    
- Run the JUnit tests in `TurtleSoupTest`  
    The method that tests `calculateRegularPolygonAngle` should now pass and show green instead of red.  
    
    If `testAssertionsEnabled` fails, you did not follow the instructions in the Getting Started guide. [Getting Started step 2](https://ocw.mit.edu/ans7870/6/6.005/s16/getting-started/#config-eclipse) has setup you must perform before using Eclipse.
    
- Implement `calculatePolygonSidesFromAngle`  
    This does the inverse of the last function; again, use the simple formula. However, **make sure you correctly round** to the nearest integer. Instead of implementing your own rounding, look at Java’s [`java.lang.Math`](https://docs.oracle.com/javase/8/docs/api/?java/lang/Math.html) class for the proper function to use.  
    
- Implement `drawRegularPolygon`  
    Use your implementation of `calculateRegularPolygonAngle` . To test this, change the `main` method to call `drawRegularPolygon` and verify that you see what you expect.

### Calculating headings

- Implement `calculateHeadingToPoint`  
    This function calculates the parameter to `turn` required to get from a current point to a target point, with the current direction as an additional parameter. For example, if the turtle is at (0,1) facing 30 degrees, and must get to (0,0), it must turn an additional 150 degrees, so `calculateHeadingToPoint(30, 0, 1, 0, 0)` would return `150.0` .  
    
- Implement `calculateHeadings`  
    Make sure to use your `calculateHeadingToPoint` implementation here. For information on how to use Java’s `List` interface and classes implementing it, look up [`java.util.List`](https://docs.oracle.com/javase/8/docs/api/?java/util/List.html) in the Java library documentation. Note that for a list of _n_ points, you will return _n-1_ heading adjustments; this list of adjustments could be used to guide the turtle to each point in the list. For example, if the input lists consisted of `xCoords=[0,0,1,1]` and `yCoords=[1,0,0,1]` (representing points (0,1), (0,0), (1,0), and (1,1)), the returned list would consist of `[180.0, 270.0, 270.0]` .

后两个稍微复杂一点，因为currentHeading不是正常直角坐标系的定义（即与x轴正方向所成角度），需要转换+归一化，具体是实现见代码

## Problem 11: Personal art

- Implement `drawPersonalArt` 实现这个方法即可，我调用了calculationHeadings的方法