---
created: 2026-02-13 09:57
updated: 2026-02-28T21:33
status: Completed
topics: Code review ; Good principles of programming
---
>[!SUMMARY] Table of Contents
>    - [[Reading4 Code Review#Code Review|Code Review]]
>    - [[Reading4 Code Review#Style Standard|Style Standard]]
>    - [[Reading4 Code Review#Smelly Example #1|Smelly Example #1]]
>        - [[Reading4 Code Review#Don't Repeat Yourself|Don't Repeat Yourself]]
>        - [[Reading4 Code Review#Comments where needs|Comments where needs]]
>        - [[Reading4 Code Review#Fail Fast|Fail Fast]]
>        - [[Reading4 Code Review#Avoid Magic Number|Avoid Magic Number]]
>        - [[Reading4 Code Review#One Purpose For Each Variable|One Purpose For Each Variable]]
>    - [[Reading4 Code Review#Smelly Example #2|Smelly Example #2]]
>        - [[Reading4 Code Review#Use Good Name|Use Good Name]]
>        - [[Reading4 Code Review#Use Whitespace to Help the Reader|Use Whitespace to Help the Reader]]
>    - [[Reading4 Code Review#Smelly Example #3|Smelly Example #3]]
>        - [[Reading4 Code Review#Don’t Use Global Variables|Don’t Use Global Variables]]
>        - [[Reading4 Code Review#Methods Should Return Results, not Print Them|Methods Should Return Results, not Print Them]]
>    - [[Reading4 Code Review#Summary|Summary]]
## Code Review

代码审查指非原作者对代码的系统、仔细的研究，有两个目的：
1. 提升代码质量：检查bug，预测可能的bug，检查代码清晰度，检查代码风格是否符合代码规范
2. 提高程序员水平：通过研究他人代码了解新语言或新特性

> Reading中发现的神秘黑客字典网站[The New Hacker's Dictionary - Table of Contents](https://www.outpost9.com/reference/jargon/jargon_toc.html#SEC30)

## Style Standard

代码规范是指个人、团体或公司遵从的统一样式，本课程没有要求正式的代码规范，只是将Java官方的规范指南作为参考：[Code Conventions for the Java Programming Language: Contents](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html)

即使没有硬性要求，但是本课程依旧要求养成自己的代码风格，并在自己的项目中从一而终（self-consistent）；并且要求在小组合作项目中能够遵循小组共同约定

不过虽然没有完整的规范文档，下面还是会通过三个例子列出一些公认的规则，更多的规则会在后面的抽象数据类型、并发和线程安全等章节提及

## Smelly Example #1

程序员经常把糟糕的代码称为何异味（Smelly），需要将其清除以保持代码卫生（Code Hygiene）
以下是一个bad smell的 求给定日期为当前年份的第几天 的代码：
```java
public static int dayOfYear(int month, int dayOfMonth, int year) {
    if (month == 2) {
        dayOfMonth += 31;
    } else if (month == 3) {
        dayOfMonth += 59;
    } else if (month == 4) {
        dayOfMonth += 90;
    } else if (month == 5) {
        dayOfMonth += 31 + 28 + 31 + 30;
    } else if (month == 6) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31;
    } else if (month == 7) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30;
    } else if (month == 8) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30 + 31;
    } else if (month == 9) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30 + 31 + 31;
    } else if (month == 10) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30 + 31 + 31 + 30;
    } else if (month == 11) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30 + 31 + 31 + 30 + 31;
    } else if (month == 12) {
        dayOfMonth += 31 + 28 + 31 + 30 + 31 + 30 + 31 + 31 + 30 + 31 + 31;
    }
    return dayOfMonth;
}
```

### Don't Repeat Yourself

这段代码显然出现了大量的重复，对于每个月的逻辑基本一致，然而却重复了11次，这违反了DRY原则：永远不要重复自己
Reading强调重复自己的危险性：复制粘贴的代码块越大，传播的错误风险越大，修改的复杂度越高
这是有道理的，因为如果一段代码需要重复出现多次，说明应该被抽象为一个模块，这样使用的地方只用调用这个模块即可，同时对这个模块的修改也可以同步到每个引用

### Comments where needs

在必要的地方留下合适的注释。这里的必要的地方指：规范、引用他人代码和代码中晦涩之处

虽然[[Reading3 Test#Specification]]中已经提及过规范，但是并没有明确Spec的格式要求。Java的文档注释规范为Javadoc，格式如下：
```
/**
 * 这部分是【简述区】（Summary）。
 * 用一句话描述这个方法或类的用途。
 * <p>
 * 这部分是【详细描述区】。可以使用 HTML 标签，
 * 比如段落标签 p，用来详细解释复杂的逻辑。
 *
 * @param 参数名 参数的含义描述
 * @return 返回值的含义描述
 * @throws 异常类型 抛出异常的情况描述
 */
```

可以参考之前HailstoneSequence方法的规范实现：
```java
/**
 * Compute the hailstone sequence.
 * See http://en.wikipedia.org/wiki/Collatz_conjecture#Statement_of_the_problem
 * @param n starting number of sequence; requires n > 0.
 * @return the hailstone sequence starting at n and ending with 1.
 *         For example, hailstone(3)=[3,10,5,16,8,4,2,1].
 */
public static List<Integer> hailstoneSequence(int n) {
    ...
}
```

除了规范以外，引用他人代码的时候也应该留下注释，以同步被引代码更新和避免侵权，示例如下：
```java
// read a web page into a string
// see http://stackoverflow.com/questions/4328711/read-url-to-string-in-few-lines-of-java-code
String mitHomepage = new Scanner(new URL("http://www.mit.edu").openStream(), "UTF-8").useDelimiter("\\A").next()
```

除了引用和规范以外，对晦涩、关键的代码也应该注释（~~类目~~）：
```java
sendMessage("as you wish"); // this basically says "I love you"
```

`dayOfYear`方法既没有规范，也没有在关键代码处给出注释（虽然好像没什么关键的）
### Fail Fast

如果程序有bug，它应该越早崩溃越好，而不是带着错误继续进行直到错误蔓延。快速失败有三个等级：
1. 静态检查：在编译前就发现问题并解决，确实崩溃的够早
2. 动态检查：通过断言和抛出异常，使程序在发生错误的时刻立即崩溃，这样能很好的定位错误源头
3. 产生错误结果：输出不合预期或者操作台交互错误，这时候已经离发生错误的地点有相当距离了，难以溯源，快速失败失败了

`dayOfYear`方法缺少快速失败机制，对不合法的输入没有检查，特别是日期和月份这些带有明确定义域的参数

### Avoid Magic Number

[魔法数字](https://en.wikipedia.org/wiki/Magic_number_\(programming\)#Unnamed_numerical_constants)指的是数值字面量（Numerical literal），他们在特定上下文中有含义，但是不够明确

`dayOfYear`中存在很多魔法数字：
1. 59和90，可以猜测59其实是31+28，但是意义不够明确，应该显示展示计算过程；
2. 包括1-12表示月份，但是使用枚举量表示会更清楚；
3. 还有每个月的天数，如果存在数组里显然会更容易理解

### One Purpose For Each Variable

每个变量都应该有唯一目的，而名称又是理解其目的的重要依据，所以当程序较为复杂的时候，创造尽可能多的不重名变量

`dayOfYear`中 `dayOfMonth`变量一开始是表示今天的日期，但是在后面又变成了今年迄今为止的天数，这就违反了规则中的唯一目的
不仅如此，一般而言应该保持参数不变性，即传入的参数应该为final值，毕竟方法的未来实现可能需要知道这个参数

## Smelly Example #2

在 `dayOfYear` 中存在一个潜在的错误。它根本无法处理闰年。作为修复该问题的部分，我们假设编写一个闰年方法。

```java
public static boolean leap(int y) {
    String tmp = String.valueOf(y);
    if (tmp.charAt(2) == '1' || tmp.charAt(2) == '3' || tmp.charAt(2) == 5 || tmp.charAt(2) == '7' || tmp.charAt(2) == '9') {
        if (tmp.charAt(3)=='2'||tmp.charAt(3)=='6') return true; /*R1*/
        else
            return false; /*R2*/
    }else{
        if (tmp.charAt(2) == '0' && tmp.charAt(3) == '0') {
            return false; /*R3*/
        }
        if (tmp.charAt(3)=='0'||tmp.charAt(3)=='4'||tmp.charAt(3)=='8')return true; /*R4*/
    }
    return false; /*R5*/
}
```
### Use Good Name

正如“One Purpose For Each Variable”中提及的“*名称是理解目的的重要依据* ”，好的方法和变量名应该具有自我描述性，其中方法名一般为动词，变量名一般为名词。格式上的规范可以参考[[Slide4 Classes,Objects#Naming method]]

示例2的参数y，变量tmp都没有清晰的实际含义，属于糟糕名称

### Use Whitespace to Help the Reader

除了与示例1一样的糟糕命名、缺少注释、逻辑重复以外，示例2还有非常丑陋的缩进，这提示我们要维护整齐的空白结构：
1. 使用`ctrl+f`自动调整行内空白
2. 将`tab`键与空白数量绑定，避免不同编辑器的缩进尺度不一致

## Smelly Example #3

这里是一个第三例smelly code示例，它将阐述本阅读的剩余要点

```java
public static int LONG_WORD_LENGTH = 5;
public static String longestWord;

public static void countLongWords(List<String> words) {
   int n = 0;
   longestWord = "";
   for (String word: words) {
       if (word.length() > LONG_WORD_LENGTH) ++n;
       if (word.length() > longestWord.length()) longestWord = word;
   }
   System.out.println(n);
}
```

### Don’t Use Global Variables

本例中使用了两个全局变量，但是全局变量是危险的，参考[Why Global Variables Are Bad](https://c2.com/cgi/wiki?GlobalVariablesAreBad)，其中的主要观点如下：
1. 命名空间污染（Namespace Pollution）：全局变量占据了全局命名空间，当系统变得庞大时，容易造成命名空间冲突
2. 高耦合（High Coupling）：依赖于全局变量的独立模块之间产生了耦合关系，导致模块难以复用
3. 破坏线程安全（Thread Safety Issues）：多线程环境下，全局变量是天然的竞态条件来源
4. 单元测试困难（Difficult to Test）：模块依赖于全局变量，必须在测试前初始化、测试后清理来恢复环境

### Methods Should Return Results, not Print Them

本例中`countLongWords`将变量n打印出来，但是实际上既然这个方法的意图是求长词数，不如直接返回这个被测量。
原则上来说，方法应该返回结果，而不是打印结果，除非你在debug

## Summary

这集不用summary了，直接看TOC即可

