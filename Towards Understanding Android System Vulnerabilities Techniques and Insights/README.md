# Towards Understanding Android System Vulnerabilities: Techniques and Insights

## 发表会议

2019 Association for Computing Machinery （ACM2019）

## 作者信息

![1566911398423](https:\\github.com\sunSUNQ\Paper_reading\raw\master\Towards%20Understanding%20Android%20System%20Vulnerabilities%20Techniques%20and%20Insights\image\1566911398423.png)

## 摘要

​	Android成为针对其应用程序和操作系统中的漏洞的主要攻击的目标。与应用程序漏洞相比，Android中的系统级漏洞的相关研究很少。在本文中，我们通过在2015年8月启动以来的三年内的Android安全公告计划中的所有2,179个漏洞的全民分析，对Android系统漏洞进行了首次的初步探索。另外，我们提出了一个自动分析框架，分层数据库结构，用于爬取，解析，整理和分析漏洞报告及其公开可用的补丁。该框架包括（i）用于查明受漏洞影响的特定漏洞模块的轻量级技术; （ii）研究补丁代码复杂性的可靠方法（iii）基于相似性的算法来聚类补丁代码模式的算法。我们的聚类算法首先提取补丁代码的变化，这些变化不仅简明地反映了语法更改，还保留了重要的语义，然后利用关联传播根据其成对相似性来自动生成聚类。最后我们获得16种漏洞模式，包括已知文献中未知的6种新漏洞模式。除了识别这些有用的模式之外，我们还发现92％的Android漏洞位于低级模块中（主要位于初始库和内核中），而框架层仅涵盖了5％的漏洞；50%的漏洞少于10行可以修复，1,158个案例中有110个甚至只需要一行代码就可以修复。总的来说，我们提供了有关Android系统漏洞的清晰概述以及新见解。

## 背景介绍

​	针对Android的系统框架层漏洞的关注少，相关漏洞数量也少，不利于对底层的系统漏洞的理解与分析。

​	主要有三个分析模块构成对数据的分析进一步得出结论：

- 第一分析器：易受攻击的模块划分

  原理：通过对不同的Android模块分类，进一步找到最易受到影响的模块

  产出：模块分层图。92%的漏洞在C/C++编写的底层模块中（kernel和native libraries）。5%存在与框架中。2.5%存在与应用层。

- 第二分析器：补丁代码的复杂性分析

  原理：排除无关的代码行（import、include、comment等）提取真正的补丁后的代码差异。

  产出：文件层次与行不同粒度的提取。60%只需要修改一个文件。50%修改少于10行代码。

- 第三分析器：漏洞模式提取分析

  原理：相似性算法进行自动聚类。

  提取补丁差异片段 -> 代码文本 -> 相似性矩阵 -> 亲和度传播的方法进行自动聚类

  产出：83个集群，50个少于10个的样例去掉。33个中28个有明显的模式存在，19个有安全方面的模式，9个非安全。从19个中进一步提取出16个漏洞模式。6个是新的漏洞模式。

## 本文主要工作

### Android Security Bulletin Program

>  https://source.android.com/security/bulletin/
>
> 2015年8月份开始的项目

右侧是目录，每种漏洞都有详细的分类，然后cve以及补丁信息等都可以获得。

![1566954626927](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566954626927.png)

### 方法框架

- 从Bulletin网站爬取漏洞原始数据，存入到Vulnerability Metadata DB中
- 爬取从Bulletin网站中获取的补丁URLs获取到漏洞对应的补丁信息，存入到Patch code DB中
- 将收集到的信息进行整理与处理，存入到Cleaned DB中（ASB中的漏洞信息是手动录入的，EoP可能存在多种形式：elevation-of-privilege-vulnerability、elevation_of_privilege、eopv等，需要处理）
- 通过三个分析器，将数据进行分析与处理生成结果进一步分析

![1566954723856](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566954723856.png)

### 3.1数据库设计

- Android漏洞可能与多个补丁关联
- 一个补丁可能包含多个关联文件
- 一个文件可能包含多个修改代码片段
- 一个片段可能有多行代码

![1566957048017](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566957048017.png)

![1566957129591](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566957129591.png)

- 设计之后的数据库结构可以通过SQLite的内置函数进行直接查询

![1566957230305](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566957230305.png)

### 3.2漏洞模块提取

- 公开可用的补丁信息

  提取部分URL作为模块位置信息

  CVE-2018-9355：`platform/system/bt/bta/dm/bta_dm_act.cc` 中提取 `platform/system/bt`蓝牙堆栈的问题（Bluetooth stack）

- 网页本身的信息

  提取HTML内容的信息

  CVE-2016-3900： `<h3 id="eopv-in-servicemanager">` 中`eopv`是漏洞类型、`servicemanager`指定了模块

### 3.3提取补丁信息

- 删除无用行：
  - +/- 后边的空白行
  - 不考虑C/C++和Java到的include、import等行
  - 删除注释 /*... <!-- --!> * 等
  - 删除测试行与测试文件 UriTest.java

-  提取补丁信息
  - 从HTML中提取add、del、ctx、hunk等关键字
  - add、del可以用于计算连续的代码行
  - ctx、hunk可以作为停用词

$$
countFlag = max（countAdd，countDel）
$$

### 3.4补丁模式分类

- 在提取差异的时：既要反应语法的变换，还要保存相关标记维持重要语义信息

![1566959009636](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566959009636.png)

- 聚类算法流程
  - 从diff中提取信息得到diff change
  - 相似性矩阵的生成（每行是一个代码文本与其他的文本的相似度）计算字符串之间的距离（Jaro distance, **Jaro-Winkler distance**, Levenshtein distance, and Damerau-Levenshtein distance）
  - 利用亲和传播算法进行聚类生成集群

![1566959168439](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566959168439.png)

## 数据分析结果

- 从Android Security Bulletin中发现近三年的漏洞公有2179个，1349个有补丁信息，从1349个有补丁的漏洞中提取了940个代码片段。
- 80%以上都是高危的漏洞

![1566959436290](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566959436290.png)

- java编码的漏洞（7.25%）比C/C++的少很多（92.75%），1349个补丁中java占154个，C/C++占1164个
- kernel的漏洞占总数的65.7%、Native libraries占总数的23.9%（两个层都引入了大量的外来库，说明外来库的安全性比android本身的安全性更差）
- 媒体相关（media）：高风险（Media Framework、Media Libs、Media、Sound、Video等）
- WIFI相关（WI-FI）：从kernel到框架到应用层都很多
- 电话相关（Telephony）：应用层12个、框架层5个、硬件层一个

![1566959630410](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566959630410.png)

- 1/3的高风险漏洞在decoder模块中，与ih264d相关度高、libstagefright媒体库、WLAN、bootloader等需要多注意

![1566960050562](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566960050562.png)

- 60%的漏洞都可以在一个文件中修复完，80%在两个文件内
- 一个补丁很多文件都需要修改是：版本修复或者系统更新导致的

![1566960211457](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566960211457.png)

- 50%的漏洞可以在10行内修复、1/3的可以在5行内修复、1/5的可以在两行内修复
- 1158个补丁中有110个只需要一行就可以修复（很多漏洞都是实施错误）

![1566960334711](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566960334711.png)

### 模式提取与聚类分析

​	940个短代码片段上运行聚类算法获取到83个聚类，其中50个每个聚类少于10个片段因此舍弃。剩下的33个中有28个是有明显的模式的，其中19个是与安全相关的模式，从19中提取了16个补丁模式，其中有六个是其他工作中没有提到过的漏洞类型。

![1566960591031](C:\Users\varas\AppData\Roaming\Typora\typora-user-images\1566960591031.png)

1. P1（new）：kernel地址泄露

   当打印%p的时候会泄露kernel的敏感数据信息。28个漏洞

2. P2（new）：引用引起的Android服务错误检索

   Android系统开发人员通过引用服务来进行编写代码，但是服务指针可以被另一个线程或者系统Binder进行调用锁死。10个漏洞。

3. P3（new）：Android Parcelable序列化不一致

   跨不同进程的数据需要序列化与反序列化Parcelable对象，writeToParcel()和readFromParce()类型不一致可能会使攻击者利用其进行提权。7个漏洞。

4. P9（new）：不完整的C++摧毁

   出现在一些Android媒体编码器中，C++摧毁未完成，内存缓冲区扔存在数据可以被控制。

5. P12（new）：缺少某些参数，导致逻辑缺陷

   详细逻辑不通，但都与缺少参数和相应处理逻辑有关

6. P14（new）：忘记设置const/transient

   const：防止被修改。transient：防止被初始化

7. P4&P7：特定与Android的漏洞模式

   P4：错误的将敏感的Android组件导出到其他应用程序中

   P7：缺少权限或UID的检查

8. P5&P8&P13&P15：与溢出相关的漏洞模式

   5、8、13：堆栈溢出 15：整数溢出

   缺少边界检查、未执行或者错误的IF语句检查条件或忘记处理某个分支

9. P10&P11：未初始化导致的漏洞

   P10：使用memset来初始化内存缓冲区

   P11：忘记为某个变量分配初始值

   可能被利用来泄露有关内存布局的信息

10. P6：use-after-free和double free

    不正确的使用free()

11. P16：缺少锁与关锁的引起的数据竞争

    锁的错误使用

### 本文研究的结论

1. 2179个漏洞中80%以上都是高危漏洞。
2. 漏洞的模块区分有利于研究人员开发。
3. 实施错误是主要漏洞来源，规避很重要。
4. 本文模式可以直接用于静态漏洞分析。

## 结论

​	在本文中，我们通过对近三年Android安全公告计划中的2,179个漏洞及其1,349个公开可用补丁进行全面分析，对Android系统漏洞进行了系统性的研究。我们提出了一个自动分析框架及三个分析器，用于分析易受攻击的模块，补丁代码复杂性和漏洞模式。我们设计了一种基于相似性的算法，该算法可以提取补丁代码的差异，并利用关联性传播自动聚集补丁代码模式。通过这个分析框架，我们确定了Android漏洞在不同模块中的分布，研究了Android补丁代码的复杂性，并成功获得了16个漏洞模式，其中包括6个未知的漏洞模式。未来，我们计划通过支持长代码片段来进一步改进我们的聚类算法，并随着时间的推移逐步改进我们的分析结果。
