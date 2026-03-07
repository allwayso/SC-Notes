---
created: 2026-03-06 20:48
updated: 2026-03-07T12:46
status: Updating
topics: Regular expression;grammers
---
## Grammer

Grammer(下称文法)定义了一组sentence(下称语句)，句子是由各种symbol(下称符号)构成的，而这些符号也被叫做terminals(下称终端)，即语法树中没有子节点、不能再被当作变量的常量。

文法是由一组productions(产生式)描述的，每个产生式定义一个nonterminal(非终端)，可以将非终端理解为一个变量，或者是对一种表达方式的抽象。产生式的基本结构：`非终端 ::= 终端、非终端和运算符的表达式`

由于非终端可以由其他非终端或者自己定义，所以一个文法会有一个最上层、抽象程度最高的非终端，被称为root或者start，由它来最终判断一个语句是否符合这个文法。Mit6.005中用lowercase表示非终端。

### Grammar Operators

产生式右侧的expression(表达式)有几种基本运算符：
1. concatenation(连接)：如`x ::= y z`，表示x是由一个y后面跟着一个z构成的
2. repetition：如`x ::= y*`，表示x是由0个或者多个y连接起来构成的
3. union：如`x ::=y | z`，表示x是由y或者z构成的

其他的运算符只是语法糖，但是由于常用，所以可以了解一下：
4. option(选择)：如`x ::=y ?`,表示x是由y或者空句构成的
5. 1+repetition：如`x ::= y+`,表示x是由1个及以上y连接起来构成的
6. character classes：如`x :: [abc]`,表示x是由x或y或z构成的

按照惯例，运算符 `*` 、 `?` 和 `+` 具有最高优先级，选择 `|` 具有最低优先级，可以使用括号来避免错误

### Example: URL

本例子从一个语句实例出发，逐步推倒出表示URL的文法：
```python
http://mit.edu/
```

我们可以定义一个只包含该语句的文法:
```python
url ::= 'http://mit.edu/'
```

但是它对于其他类似格式的语句都不能正确匹配，所以我们试图做一点泛化，将mit.edu部分拆解为两个非终端：
```python
url ::= 'http://' [a-z]+ '.' [a-z]+  '/'
```

现在这个文法能正确匹配诸如http://stanford.edu/，http://google.com/的语句，但是不适配多hostname(主机名)或含有数字的语句。
上面出现的单行产生式，仅由终端和运算符表示，称为regular expression(正则表达式)。正则表达式很紧凑，但是使用非终端的表达式可读性更强:
```python
url ::= 'http://' hostname '/'
hostname ::= word '.' word
word ::= [a-z]+
```

现在我们希望对这个表达式进行泛化，使其能匹配更多主机名，并有一个可选的port number(端口号)：
```python
url ::= 'http://' hostname (':' port)? '/' 
hostname ::= word '.' hostname | word '.' word
port ::= [0-9]+
word ::= [a-z]+
```

注意到主机名是递归定义的，递归步骤是在主机名前加一个word，基本情况是两个word由.分隔。如果使用重复运算符，可以这样表示：
```python
hostname ::= (word '.')+ word
```

当然泛化还没结束，word应被泛化为接受所有合法字符的字符串，结尾的“/”应该被变为由/分隔的路径，http应该泛化为所有合法协议
完整的URL正则表达式相当复杂：
```python
URL ::=^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?(\?.*)?$
```

其中的转义字符、\d等符号将在正则表达式中介绍

### Example: Markdown and HTML

为了简化，这里的两种标记语言仅指定italic(斜体)，其简化表达式分别为：
```python
markdown ::=  ( normal | italic ) *
italic ::= '_' normal '_'
normal ::= text
text ::= [^_]*
```
和
```python
html ::=  ( normal | italic ) *
italic ::= '<i>' html '</i>'
normal ::= text
text ::= [^<>]*
```

注意，这里的markdown表达式是经过二次阉割的，实际上md也接受嵌套，即italic ::= '\_' normal '\_'中的normal实际上应该为markdown，与html类型。这里的阉割只是为了在正则表达式中做区分。
## Regular Expression

正则文法可以看作将根非终端的表达式层层展开，直到不存在非终端为止

上文中阉割版markdown能够化简为一下形式：
```
markdown ::= ([^_]* | '_' [^_]* '_')*
```
其对应的正则表达式可以略去单引号''

而html可以简化为以下形式：
```
html ::=([^<>]* | '<i>' html '</i>)*
```
但是此处的html不能再继续拆解了，也就是html无法用非递归的方式表达，即不存在正则表达式

除了前面出现的几种运算符以外，还有一些特殊运算符(作为终端使用）：
```
.       any single character
\d      any digit, same as [0-9]
\s      any whitespace character, including space, tab, newline
\w      any word character, including letters and digits
\., \(, \), \*, \+, ...
        escapes an operator or special character so that it matches literally
```

由于.号作为终端，所以要使用字符.的时候需要用反斜杠\表示转义，其他字符如 | + * ? 等特殊符号也需要转义，反斜杠本身可以转义自己，即\\\

### Using regular expressions in Java

由于java本身将正则引擎中的转义字符/作为命令开端，所以为了使正则引擎得到转义字符/，在编写正则表达式的时候要使用//来告知编译器我需要传入/本身。更极端一点的，当我希望在正则引擎中转义转义字符，正则引擎需要接受//，则在编写过程中应变为////。

在 Java 中，可以使用正则表达式来处理字符串（参见 `String.split` 、 `String.matches` 、 `java.util.regex.Pattern` ）。

### Context-Free Grammars

可以被文法系统表示的语言被称作Context-free(上下文无关的)，但上下文无关的语言并不一定是正则的。也就是说存在一些上下文有关的文法，他们依赖于上下文中的一些条件；甚至存在无限制文法。

通过alpha->beta抽象式表示：
1. 上下文无关文法：alpha为单个非终端，beta为包含各种元素的表达式
2. 上下文有关语法：这里x和y分别表示上文和下文，但是要求w不能为空$$xAy \to x \omega y$$
3. 无限制文法：alpha和beta都无限制，alpha可以长于beta，符号可以相互通信也可以凭空消失

当然本章只要求理解上下文无关文法和其中更严格的正则文法，下面两种看过就行。
