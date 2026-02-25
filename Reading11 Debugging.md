---
created: 2026-02-25 11:32
updated: 2026-02-25T12:08
status: Completed
topics: systematic debugging；debugging strategy
---
>[!SUMMARY] Table of Contents
>    - [[Reading11 Debugging#Reproduce the Bug|Reproduce the Bug]]
>    - [[Reading11 Debugging#Understand the Location and Cause of the Bug|Understand the Location and Cause of the Bug]]
>        - [[Reading11 Debugging#Other tips|Other tips]]
>    - [[Reading11 Debugging#Fix the Bug|Fix the Bug]]
>    - [[Reading11 Debugging#Summary|Summary]]
## Reproduce the Bug

找到一个可重复的、尽可能小的测试用例来复现错误。如果bug是通过回归测试发现的，那只需要用回归测试的测试用例即可；如果bug是用户反馈的，则相对复杂

这里以找高频单词方法为例：
```java
/**
 * Find the most common word in a string.
 * @param text string containing zero or more words, where a word
 *     is a string of alphanumeric characters bounded by nonalphanumerics.
 * @return a word that occurs maximally often in text, ignoring alphabetic case.
 */
public static String mostCommonWord(String text) {
    ...
}
```

当传入shakespeare的作品，结果却不符合预期时，通过以下步骤找到尽可能小的测试用例：
1. 二分之后是否仍然错误
2. 对于一章戏剧是否错误
3. 对于一段独白是否错误

## Understand the Location and Cause of the Bug

可以通过以下步骤定位bug：

1. 研究数据：看看是否有异常提供的堆栈信息
2. 假设：假设引起bug的模块
3. 实验：验证bug是否在该模块中，通过断言、断点、新的测试用例或者打印语句检验
4. 重复：如果假设错误，则更换假设，进一步压缩错误可能发生的区间，直到找到对应模块

### Other tips

**Bug localization by binary search**：二分法找bug，即对模块进行二分测试
**Prioritize your hypotheses**：优先考虑假设，而不是库函数、编译器等相对可靠性较高的部件
**Swap components**：当有充分证据时，可以替换某一个库函数、更换语言版本或者操作系统
**Make sure your source code and object code are up to date**：做好准备工作，总是先pull并清除编译产物后再运行
**Get help**：从论坛、ai或者作者处得到帮助
**Sleep on it**：~~遇到问题就睡大觉~~Trade latency for efficiency.

## Fix the Bug

修复bug不能只是打个补丁，应该认真审视发生错误的原因，按以下步骤处理：

1. 寻找问题本质：是实现错误还是设计错误，还是粗心的错误
2. 检查同伙：审查是否有使用类似逻辑的方法，并检验修复此处bug会不会影响到使用该接口的方法
3. 回归测试：为这个bug写一个测试用例并放在回归测试中，避免未来修改代码重新引入问题

## Summary

思路与[[Slide6 Design,Debugging,Interface#Debugging]]类似，但是通过引入例子来深化内容，并且与前面的[[Reading3 Test]]相呼应。不过由于没有实操过debug，毕竟还是有些纸上谈兵。