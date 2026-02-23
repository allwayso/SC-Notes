---
created: 2026-02-19 20:50
updated: 2026-02-23T00:21
status: Updating
topics: Test；Spec
---
## Overview

**工作流要求**：
1. 仔细研究规范的要求
2. 按照规范写测试
3. 根据规范写实现
4. 修改实现和测试用例

本次作业不仅对实现评测，也对测试进行评测，**测试的评判标准**如下：
1. 使用|输入空间划分：[[Reading3 Test#Test Case Construction Choosing Test Cases by Partitioning]]
2. 为测试写注释：描述划分方式和测试用例选取方式
3. 使用虚拟测试用例：不需要大量真实twitter数据，只需要虚拟的、精细的、小量的测试用例即可
4. 能够发现bug：将使用错误实现来检查测试是否能发现bug
5. 能够允许等价实现：将使用等价正确实现来检查测试是否具有普适性
6. 把每个测试用例单独放在一个方法中：避免因为使用一个巨大测试用例而找不到错误
7. 满足前置条件的要求：测试用例应符合前置条件的约束，对前置条件以外的输入允许任意行为
8. 测试后置条件中的特殊情形：如果后置条件对某个特殊情形做出了要求，则测试用例应包含这个特殊情形

**不应改变的内容**：
1.  `Tweet` , `Timespan` , and `TweetReader`三个类不应改变任何内容
2. `Extract` , `Filter` , `SocialNetwork` , `ExtractTest` , `FilterTest` , and `SocialNetworkTest`的类名不应改变
3. 不应改变任何方法的签名和规范



