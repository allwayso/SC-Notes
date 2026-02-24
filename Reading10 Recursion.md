---
created: 2026-02-24 23:22
updated: 2026-02-25T00:52
status: Completed
topics: decompose recursion ques;helper methods;advantage and disadvantage
---
>[!SUMMARY] Table of Contents
>    - [[Reading10 Recursion#Recursion|Recursion]]
>    - [[Reading10 Recursion#Help Method|Help Method]]
>    - [[Reading10 Recursion#Recursive Problems vs. Recursive Data|Recursive Problems vs. Recursive Data]]
>    - [[Reading10 Recursion#Reentrant Code|Reentrant Code]]
>    - [[Reading10 Recursion#When to Use Recursion Rather Than Iteration|When to Use Recursion Rather Than Iteration]]
>    - [[Reading10 Recursion#Common Mistakes in Recursive Implementations  |Common Mistakes in Recursive Implementations  ]]
>    - [[Reading10 Recursion#Summary|Summary]]
## Recursion

递归由基本情况和递归步骤定义，基本情况是递归停止的条件，递归步骤是将大问题分解为更简单、更小的子问题，最终收敛于基本情况

本章以分解所有子串为例讲解：
```java
 1 public static String subsequences(String word) {
 2     if (word.isEmpty()) {
 3         return ""; // base case
 4     } else {
 5         char firstLetter = word.charAt(0);
 6         String restOfWord = word.substring(1);
 7         
 8         String subsequencesOfRest = subsequences(restOfWord);
 9         
10         String result = "";
11         for (String subsequence : subsequencesOfRest.split(",", -1)) {
12             result += "," + subsequence;
13             result += "," + firstLetter + subsequence;
14         }
15         result = result.substring(1); // remove extra leading comma
16         return result;
17     }
18 }
```

此时基本条件为word为空，递归步骤比较长，大致就是将非空word拆分为首字母和剩余子串，遍历剩余子串的拆分：对于每个对象，直接加入result，或加上首字母再加入result

递归的关键就是分解（decompose），即如何从一个问题中抽象出一个拆分逻辑，并将其分解为基本情况和递归步骤。同一个问题的拆分逻辑往往是相似的，不过表现可能不同，除了上述`subsequences`以外，还有许多可能，比如下面这个

## Help Method

```java
/**
 * Return all subsequences of word (as defined above) separated by commas,
 * with partialSubsequence prepended to each one.
 */
private static String subsequencesAfter(String partialSubsequence, String word) {
    if (word.isEmpty()) {
        // base case
        return partialSubsequence;
    } else {
        // recursive step
        return subsequencesAfter(partialSubsequence, word.substring(1))
             + ","
             + subsequencesAfter(partialSubsequence + word.charAt(0), word.substring(1));
    }
}
```
`subsequencesAfter`的拆分逻辑与`subsequences`方法实际上类似，都是将word拆分为首字母和剩余串，但是后者是自上而下拆分后自下而上组装，而前者直接向下拆分为选择首字母或者不选这两个决策分支，表现上更为直观

不过`subsequencesAfter`的规范与`subsequences`并不相同，依赖于一个变量partialSubsequence，不管是出于spec的一致性还是安全性，都不应该将其暴露给客户端，而应该将`subsequences`作为接口并在其中调用辅助方法

## Recursive Problems vs. Recursive Data

递归和斐波那契数列这种问题，具有非常明显的递归特征；但是有些问题适合用递归解决是因为它的层级结构是类似的，比如文件系统

类似文件系统的文件路径,使用递归实现也非常自然：
```java
/**
 * @param f a file in the filesystem
 * @return the full pathname of f from the root of the filesystem
 */
public static String fullPathname(File f) {
    if (f.getParentFile() == null) {
        // base case: f is at the root of the filesystem
        return f.getName();  
    } else {
        // recursive step
        return fullPathname(f.getParentFile()) + "/" + f.getName();
    }
}
```

## Reentrant Code

可重入代码指的是当一个方法正在被调用的过程中，可以再次被调用，它将状态藏在参数和局部变量中，不使用任何静态变量和全局变量。所以可重入代码是安全的，应该尽可能使用可重入代码，在后序的并发中也会提到。

## When to Use Recursion Rather Than Iteration

递归的优势：
1. 逻辑清晰：调用自身而不附带别的判断
2. 安全性强：即可重入性，不使用静态变量和全局变量使其免于很多bug

相比于迭代，递归也有缺点，即由于递归产生堆栈，导致内存开销较大，限制了递归的深度和空间复杂度

## Common Mistakes in Recursive Implementations  

这里提及了递归失败的两种常见情形：
1. 基本情形覆盖不完全：比如斐波那契数列n=0和1都是基本情形，但是可能只覆盖了n=0的情形
2. 递归过程不收敛：递归过程如果不能收敛到基本情形，就会导致无限递归，并由于栈溢出而触发`StackOverflowError`异常

## Summary

本章主要介绍了递归的概念和结构，以及适用场景，由于cpp中学习较多，笔记记录较为简略。