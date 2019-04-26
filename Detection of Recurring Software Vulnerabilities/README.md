# Detection of Recurring Software Vulnerabilities



## 发表会议

Automated Software Engineering （ASE2010）

## 作者信息

[Nam H. Pham](http://urialon.cswp.cs.technion.ac.il/), [Tung Thanh Nguyen](http://www.cs.technion.ac.il/~mbs/), [Hoan Anh Nguyen](https://levyomer.wordpress.com/) and Tien N. Nguyen

> - Electrical and Computer Engineering Department Iowa State University, USA

## 摘要

​	软件安全漏洞几乎每天都会被发现并造成重大损害。 为了支持前期检测和解决它们，我们对数千个漏洞进行了实证研究，发现其中许多漏洞由于软件重用而重复出现。 基于从研究中获得的知识，我们开发了SecureSync，这是一种自动工具，用于检测重用源代码或库的重复软件漏洞。 SecureSync的核心包括两种技术来表示和计算不同系统中易受攻击代码的相似性。 对117个开源软件系统的176个版本中的60个漏洞进行的评估表明，SecureSync能够高精度地检测重复出现的漏洞，并确定90个版本具有潜在的未报告或修复过的易受攻击的代码，并经过开发商的确。认

## 背景介绍

​	重用代码表示为：相同的代码库的调用、相同的源代码、基于公共框架的开发。

​	重用可以理解为：代码的复制与粘贴、通过复制/重用代码库来开发新的版本

将重复使用代码的分成以下三种：

- Type1：重复出现的脆弱代码片段的代码结构、函数调用、变量和常量等的名称上相同或者高度相似。
- Type2：使用相同的API来实现类似功能
- Type3：抽象意义上的高级重用（相同的算法、协议、标准等等）

## 本文工作

本文提出的**SecureSync**：是一个可以自动检测共享源代码或者库的不同软件中重复出现的漏洞的工具。

#### 可以有以下两种检测方法：

1. A系统中有漏洞报告以及对应的源码和补丁；2、将分析该源码与补丁并将信息存储到知识库中；3、进行google的代码搜索，查找与A共享代码库的B系统；4、查看B是否有相似的代码片段。
2. 给定x系统，将分析是否存在一些库或者代码片段与已知的y的漏洞相似，有的话就报告位置并且给出相应的修补意见。

#### 主要目的：

- 表示有漏洞的已经报告的修补过的代码。
- 检测代码的模式，进而发现相似的代码并报告。

#### 提高性能：

- 对位置敏感的信息序列（LSH）：快速的搜索相似树，只有相同LSH的树才进行比较。（Type1）
- 使用基于集合的过滤来查找候选项：只保留知识库中包含于图的节点具有相同标签的节点的图来进行比较（Type2）

#### 主要贡献：

1. 证明重复漏洞的存在
2. 分析重复代码的两种算法
3. SecureSync

​	之后将code snippet转化为向量，如何获取语义与语法信息：使用AST（抽象语法树）来表示代码段的语法信息，通过将代码转换成<Xs, p, Xt>的形式，表示AST中的一条从叶子节点到另一个叶子节点的路径也就是一条path-context（路径上下文）信息，将这些path-context整合到一个上下文信息包中。然后通过一个全连接层，得到每条路径的attention（权重）信息，以及一个可以代表code snippet代码段的code vector，然后可以使用这两组数据进行模型的训练，然后可以用训练好的模型来预测方法名。

​	扩展：可以应用到检测代码段的含义，恶意代码识别等用处。

## 样例描述（Type1、Type2）

-  CVE-2018-5023（Type1）

> “allows remote attackers to bypass the protection mechanism for codebase principals and execute arbitrary script via the -moz-binding CSS property in a signed JAR file”

![1556244786269](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1556244786269.png)

![1556244748307](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1556244748307.png)

- CVE-2009-0021（Type2）

> allows remote attackers to bypass validation of the certificate chain via a malformed SSL/TLS signature for DSA and ECDSA keys

![1556244974397](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1556244974397.png)

- Type1的样例展示：一般的都是在文本（被调用的函数名称、变量名称、文字、值等）和结构（语句、分支、表达式等结构）相同或者高度相似。
- Type2的样例展示：类似的方法重用、类似的输入检查和输出处理等等

## 相似性检测的方法

![1556245227435](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1556245227435.png)

- 查找所有的代码中是否存在与已知漏洞代码相似的片段
- 生成候选的代码片段
- 对于每一个都进行相似度的比较，其中Sim（B，A）表示B与A的相似度，Sim（B，A‘）表示B与A的补丁代码A’的相似程度。系数设置为0.8。

## 主要方法

### 特征提取和相似性度量（Type1）

![1556246245750](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1556246245750.png)

1. 分析代码生成AST，称为extended AST（xAST）。每个节点表示一个函数调用、一个文字、一个变量、一个操作符等；每个节点的标签包含节点的类型、签名、参数、值等。
2. 将xAST转换为向量，使用Exas方法，生成Ex（F（A）），其中F（A）表示A代码片段的特征。
3. 计算每个向量之间的曼哈顿距离（Manhattan distance）。
4. 基于Text的过滤，将变量名不同的都排除掉。（跨版本的名字映射，防止版本间的变量名更改导致漏报，使用OperV）
5. 基于Structure的过滤，使用局部敏感哈希（LSH）来对哈希值进行比较，差很多的直接丢弃。
6. 将通过上述两种过滤方式留下来的statement进行组合，组合到一个方法里边，然后进行相似度分析。

### 特征提取和相似性度量（Type2）

![1556246306728](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1556246306728.png)

1. 分析代码生成的extended GRaph-based Usage Model，xGRUM，有向的标记非循环图。
   - 每个操作节点表示一个方法调用或者函数
   - 每个数据节点表示一个变量
   - 每个控制节点表示控制结构的分支点（if、while、for、switch）
   - 每个运算符节点表示一个运算符（not、and、or）
   - x与y的边表示控制|数据流的依赖关系
   - label包含名字、类型、关系函数等的token
2. 使用图对齐算法（Graph Alignment Algorithm），分析源代码与补丁后代码的节点信息，如果未改变的节点说明与漏洞无关，删除节点，改变的节点留下来。
3. 然后同样使用Exas方法，将图信息转换为向量Ex（U（A）），U（A）表示图的特征信息。
4. 使用了Text-based的筛选方法。
5. 进行相似性测量。

### 知识库的构建

- 人工的找到有漏洞和补丁的漏洞信息
- 使用Google search来找到具有相似代码的软件
- 将找到的样本进行筛选与xAST或者xGRUM的构建
- 进行相似度分析

### 补丁推荐功能

- Type1：从已知的补丁中推荐
- Type2：加入缺失的函数调用，输入与输出的检查等等。

## 数据集

![1556246903902](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1556246903902.png)

- 对于Type1选择了基于Mozilla框架的开源项目FireFox、Thunderbird、SeaMonkey选择了一共三个开源项目的51个版本，其中有48个漏洞信息。

![1556247052400](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1556247052400.png)

- 对于Type2选择了12个漏洞类型、116个开源项目的125个版本。

## 结果展示

![img](file://C:/Users/varas/AppData/Roaming/Typora/typora-user-images/1556245079662.png?lastModify=1556247050)

​	Report是指手动收集到的有漏洞的报告个数、Group是人工分析的重用代码的种类、RV是总共跑出来的重用漏洞的个数。

## 总结

​	本文报告了一个关于代码重用的漏洞的实证研究。该研究表明，由于在代码重用、API和库重用、或者更高抽象级别（例如规范）上重用源代码，在不同系统中存在许多漏洞。我们还引入了一种自动工具来检测不同系统上的此类重复漏洞。 SecureSync的核心包括两种技术，用于在不同系统中匹配易受攻击的代码。对实际软件漏洞的评估表明，SecureSync能够高精度地检测重复出现的漏洞，并识别即使在成熟系统中尚未报告或修复的多个易受攻击的代码位置。