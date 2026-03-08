---
created: 2026-03-07 21:44
updated: 2026-03-08T11:34
status: Completed
topics: Antlr；Grammer to source code；Parse tree； Abstract syntax tree
---
>[!SUMMARY] Table of Contents
>- [[Reading18 Parser Generator#Parser Generators|Parser Generators]]
>    - [[Reading18 Parser Generator#An Antlr Grammar|An Antlr Grammar]]
>    - [[Reading18 Parser Generator#Generating the parser|Generating the parser]]
>    - [[Reading18 Parser Generator#Calling the parser|Calling the parser]]
>- [[Reading18 Parser Generator#Constructing an abstract syntax tree|Constructing an abstract syntax tree]]
>- [[Reading18 Parser Generator#Summary|Summary]]
# Parser Generators

解释器生成器以文法作为输入，自动生成可以使用该语法解析字符流的源代码。本章使用的Parser Generator为Antlr。

## An Antlr Grammar

以下是输入Antlr生成器的html文法源文件：
```python
grammar Html;

root : html EOF;
html : ( italic | normal ) *;
italic : '<i>' html '</i>';
normal : TEXT; 
TEXT : ~[<>]+;  /* represents a string of one or more characters that are not < or > 
```

这里的文法规则与上一章讲述的差不太多，不过将`:==`替换为了`:`；同时这里`~`表示非，而等价的传统正则表达式为`[^<>]`

接下来解析一下这个源文件：
1. root：最高层级的非终端，可以自由命名
2. html : ( normal | italic ) \*：这表示Antlr的语法规则与正则表达式基本一致

## Generating the parser

接下来的步骤就是使用Antlr来生成解析整数加法运算式的源代码，并通过源代码做一些事情。

首先先理解一下`IntegerExpression.g4`中的文法规则：
```java
grammar IntegerExpression;
import Configuration;

root : sum EOF;
sum : primitive ('+' primitive)*;
primitive : NUMBER | '(' sum ')';
NUMBER : [0-9]+;

```

注意这里出现了嵌套结构，sum由一个primitive（基本情况）或者多个由+号连接的基本情况表示，而基本情况为数字或者由()括号包裹的sum

以`(1+2)`为输入流来理解文法：
```
(root (sum1 (primitive1 ( (sum2 (primitive 2) + (primitive 3)) ))) <EOF>)
entering root 
entering sum //第一步，找sum 1
entering primitive //sum 1由primitive 1构成，找primitive 1
terminal ( //遇到终端(，发现与primitive中的(sum)规则一致
entering sum //接着找primitive1中的sum2
entering primitive //sum2又由primitive2构成，又开始找primitive2
terminal 1 //找到终端1，发现这是一个primitive，所以primitive2=1
exiting primitive // 回溯，按照sum的结构找sum2的后续部分，即+和primitive3
terminal + //找到终端+
entering primitive //接着找primitive3
terminal 2 //找到终端2，发现这是一个primitive，所以primitive3=2
exiting primitive //回溯，发现sum2已经完整
exiting sum //继续回溯，继续找primitive1的剩余部分，即右括号
terminal ) //找到终端）
exiting primitive //回溯，primitive1已经完整
exiting sum //继续回溯，sum1已经完整
terminal <EOF> //找到结束符，此时整个root已经完整
exiting root //root完整，匹配结束
```

以上遍历流程实际上就是解析树的深度优先遍历：
![[Pasted image 20260308110625.png]]

接下来介绍一下Antlr产生的源代码：
1. **lexer（分析器）**：以字符流作为输入，并产生终端流（Antlr 称之为 token）作为输出，如 `NUMBER` 、 `+` 和 `(` 。对于 `IntegerExpression.g4` ，生成的词法分析器被称为 `IntegerExpressionLexer.java`
2.  **parser（解析器）**：接受lexer输出的token流，生成解析树。本例中生成的解析器称为 `IntegerExpressionParser.java
3. **tree walker（树遍历器）**：允许编写代码遍历解析器生成的解析树，如下所述。本例中生成的树遍历器文件是接口 `IntegerExpressionListener.java` ，以及接口的空实现 `IntegerExpressionBaseListener.java`

除此之外，Antlr还生成了一些.tokens文件，如`IntegerExpression.tokens` 和 `IntegerExpressionLexer.tokens`，包含找到的终端符号，用于文法嵌套文法时使用

使用Antlr需要注意以下几点：
1. 不要修改Antlr产生的源文件，要改就改.g4文件
2. 每次改完.g4文件，记得重新调用java -jar指令更新产生的源文件
3. 调用完指令之后，记得在eclipse中f5刷新

## Calling the parser

生成了这四个源代码文件之后，大致通过以下步骤使用这些工具：
```java
 CharStream stream = new ANTLRInputStream(string);//将字符串变为字符流
 IntegerExpressionLexer lexer = new IntegerExpressionLexer(stream);//分析器接受字符流
 TokenStream tokens = new CommonTokenStream(lexer);//分析器将字符流变为终端流
 IntegerExpressionParser parser = new IntegerExpressionParser(tokens);//解析器接受终端流
 ParseTree tree = parser.root();//生成解析树
 System.err.println(tree.toStringTree(parser));//打印解析树，通过嵌套结构表示，如(root (sum (primitive ( (sum (primitive 1) + (primitive 2)) ))) <EOF>)
 Trees.inspect(tree, parser);//可视化生成解析树窗口
 new ParseTreeWalker().walk(new PrintEverything(), tree);//用监听器listener遍历解析树，给出完整解析步骤
 MakeIntegerExpression exprMaker = new MakeIntegerExpression();//一个监听器对象，包含空栈和遍历逻辑
 new ParseTreeWalker().walk(exprMaker, tree);//用exprMaker遍历tree，将产生的原子化的表达式压入exprMaker中的栈
 System.out.println(exprMaker.getExpression());//输出exprMaker的最终表达式，如3+(1+2)变为(3)+((1)+(2))
```

# Constructing an abstract syntax tree

对整数加法运算进行抽象，其递归的抽象数据类型定义：
```python
IntegerExpression = Number(n:int)
                    + Plus(left:IntegerExpression, right:IntegerExpression)
```

当使用ADT表示语法时，它通常被称为抽象语法树。本例中，我们构造IntegerExpression接口，并为其实现两个实例类Number和Plus即可。下面内容不再赘述，与课程主干相干不大。

# Summary

本章可以看作上一章文法的java实战，运用了Antlr工具，通过一个特定形式的文法文件，自动生成分析器、接收器和遍历器，实现自动文法匹配。
