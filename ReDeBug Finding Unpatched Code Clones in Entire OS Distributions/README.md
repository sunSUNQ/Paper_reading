# ReDeBug: Finding Unpatched Code Clones in Entire OS Distributions

> Jiyong Jang, Abeer Agrawal, and David Brumley 
>
> Carnegie Mellon University Pittsburgh, PA, USA
>
> 2012，S&P

## 摘要

程序员不应该两次修复相同的bug。但是，当有问题代码的补丁没有及时覆盖到所有代码克隆时，通常会发生这种情况。未修补的代码克隆代表潜在的错误，因此对于这种代码克隆快速检测非常重要。在本文中，我们介绍了**ReDeBug，一个用于在系统不同规模代码库中快速查找未修补的代码克隆的系统**。虽然以前有关于代码克隆检测的工作，但ReDeBug有一个独特的设计点，它使用快速的，基于语法的方法，可扩展到系统不同规模的代码库，包括用许多不同语言编写的代码。与以前的方法相比，ReDeBug可能会找到更少的代码克隆，但可以获得规模，速度，降低错误检测率，并且与语言无关。我们通过检查Debian Lenny / Squeeze，Ubuntu Maverick / Oneiric，所有SourceForge C和C ++项目以及未修补代码克隆的Linux内核中的所有包中的所有代码来评估ReDeBug。 ReDeBug以700,000 LoC / min处理超过21亿行代码以构建源代码数据库，然后通过在商品桌面上在8分钟内检查376个Debian / Ubuntu安全相关补丁，在当前部署的代码中找到15,546个未修补的已知易受攻击代码克隆情况。我们通过确认最新版Debian Squeeze软件包中的145个真正的错误来展示ReDeBug对真实程序的影响。

## 介绍

已有的工作存在问题与ReDeBug解决方案：

1. 可扩展性：目前的方法着重于尽可能多地找到代码克隆，而不是扩展。（ReDeBug处理迅速，可以作为正常开发或者补丁过程的一部分）
2. 缺乏对许多不同语言的支持：目前的研究首先解析程序并使用基于高级代码表示的各种匹配启发式算法，如CFG和解析树。 但是，为许多不同的语言实现强大的解析器是一个非常困难的问题。（规范化使ReDeBug能够识别用许多不同语言编写的程序中的各种潜在安全漏洞）
3. 高误报率：用于找到更多代码克隆的高级启发式算法引入了高误报率，即报告了大量错误代码克隆。（ReDeBug通过使用接近精确匹配，而不是先前的代码克隆工作所采用的模糊匹配，来降低误报率）

我们使用ReDeBug来检查Debian 6.0 Squeeze（348,754,939 LoC 1），Debian 5.0 Lenny（257,796,235LoC），Ubuntu 11.10 Oneiric（397,399,865 LoC），Ubuntu 10.10 Maverick（245,237,215LoC），Linux内核（8,968,871 LoC）中未修补的代码克隆，以及SourceForge（922,424,743 LoC）的所有C / C ++项目。到目前为止，ReDeBug通过检查376个Debian / Ubuntu安全相关补丁，在总计2,180,581,868 LoC中找到了15,546个未修补的代码克隆。这些补丁解决了各种问题，从缓冲区溢出，到信息泄露漏洞，再到拒绝服务漏洞。我们的测量结果表明，即使ReDeBug更简单，它实际上也找到了与现有方法相当数量的代码克隆。

## 主要贡献

- 分析整个操作系统分布，以了解未修补的代码克隆的当前趋势。ReDeBug是第一个探索超过21亿行整个操作系统发行版的工具。展示了未修补的代码克隆是现代发行版中反复出现的问题，并且从Debian Lenny / Squeeze，Ubuntu Maverick / Oneiric发行版，Linux内核和SourceForge中找到了15,546个未修补的代码克隆。到目前为止，ReDeBug已经确认了145个真正的错误。
- ReDeBug在可扩展性，速度和误报率方面为代码克隆检测提供了新的设计空间。特别ReDeBug在日常环境中可以被开发人员使用，以便通过快速查询已知漏洞来提高代码的安全性。
- 提供了操作系统分布中克隆代码总量的第一个经验计算。

## ReDeBug介绍

### ReDeBug的目标

1. 专注于未修补的代码克隆
2. 扩展到大型和多样化的代码库，如OS  distributions
3. 最小化误报率
4. 尽可能模块化和提供用户可选择的参数
5. 尽可能地与语言无关。

### ReDeBug系统的核心使用以下步骤完成目标

1. 规范化处理每个文件。 默认情况下，ReDeBug删除典型的语言注释，删除所有非ASCII字符，删除除新行之外的冗余空格，并将所有字符转换为小写。 如果文件是C，C ++，Java或Perl（由扩展名或UNIX文件命令标识），我们也会忽略大括号。

2. 基于新增行和正则表达式子串将标准化文件标记。

3. 在token流上滑动长度为n的窗口。 每个n个token被认为是要比较的代码单元。

4. 给定两组fa和fb的n-tokens，我们计算相同的代码数量。当找到未修补的代码克隆时，如果fa是原始的错误代码片段，我们计算：

   ![1568980811766](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568980811766.png)

   当我们想要测量文件之间的代码克隆总量时，我们计算相同的代码的token串百分比：

   ![1568980855370]https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568980855370.png)

   通过任一计算，通常仅考虑相似性或包含性大于或等于某个预定阈值θ的情况。

5. 对已识别的未修补代码克隆执行完全匹配测试，还使用编译器来识别代码克隆何时可能是死代码。

### 使用diff文件来作为数据源

ReDeBug查找修补程序所在的未修补的UNIX代码克隆统一diff格式。 统一的diff补丁由一系列差异组成。每个块包含更改的文件名，以及一系列添加和删除。添加的源代码行以“+”符号为前缀，删除以“ - ”符号为前缀。行更改表示为删除原始行并添加更改的行。

原始的错误代码包括补丁删除的所有代码。 但是，查找已更改的行是不够的：我们还必须考虑补丁的周围上下文。图中补丁一可以明显的发现更改的是strcpy函数，但是补丁二需要考虑上下文信息。

![1568981519461](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568981519461.png)

### 整体框架

![1568981642959](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568981642959.png)

1. **预处理源数据**。

   用户获取其使用的所有源文件。Debian是使用apt工具完成的。然后ReDeBug自动：
   1）执行核心系统中描述的规范化和标记工作。
   2）在token上移动一个长度为n的窗口。对于每个窗口，生成的n-tokens哈希后发送到到Bloom匹配测试器中。
   3）以原始数据格式存储每个源文件的Bloom过滤器过滤后的结果。

2. **检查未修补的代码克隆**。

   用户获得统一的diff补丁。然后ReDeBug自动：
   1）从补丁的源代码中提取原始代码片段和上下文信息的标记。
   2）规范化并标记提取的原始漏洞代码片段。对于C，C ++，Java和Python，我们删除c上下文行中的注释。
   3）将n-token窗口哈希为一组值。
   4）对每个n-token窗口哈希值执行Bloom匹配测试。

3. **对报告的克隆代码进行处理**。 

   给定代码的未修补代码克隆，ReDeBug自动：
   1）执行精确匹配测试以去除Bloom匹配测试的误报。
   2）排除在构建时未包含的死代码。
   3）仅向用户报告非死代码。

### 使用Bloom匹配器的原因

- false positive很少
- 没有 false negatives（参考文献中详细介绍）

### 相似性比对算法

实现SIMILARITY集合相似性比较，使用位向量来加速成对比较，为SIMILARITY创建位向量时使用单个散列函数，而不是多个散列函数（参考文献中详细介绍），使用特征散列对位向量中的所有元素进行编码，使用SIMILARITYbv算法。

计算操作系统整体分布相似性：
1）获取所有源代码。
2）对于每个文件，进行规范化和标记化。
3）对于每个n长度token序列i进行计算，并将结果在相应文件位向量m中设置第d位。
4）在每对向量m之间计算SIMILARITYbv。

不同粒度的相似性：ReDeBug可以从文件中提取函数，并使用算法计算函数的相似性。还可以计算n-token的相似性总分数，其表示找到的代码差异程度。 

### 查找未修补的代码算法

类似于相似性检测的算法，区别：首先，预先处理源代码以构建数据库。在计算相似性时比较每对文件时会在数据库查询补丁。第二个区别是我们使用Bloom匹配器进行未修补的代码克隆检测，而使用特征哈希进行相似性检测。

## 框架具体实现 

### 简介

ReDeBug在大约1000行C代码和250行Python中实现。 规范化在Python代码中进行了修改。 我们使用QuickLZ库执行压缩/解压缩，同时将QLZ_COMPRESSION_LEVEL设置为3以获得更快的解压速度。

### 系统环境

在Linux 2.6.38（3.4 GHz Intel Core i7 CPU，8GB内存，256 GB SSD驱动器）的台式机上执行了所有实验，以查找未修补的代码克隆（构建和查询数据库），使用了8个线程构建数据库并查询错误。

### 数据集

收集了两次源代码数据集：2011年初和2011年末。首先在2011年1月/ 3月收集了2011年初数据集（Σ1）：Debian 5.0 Lenny和Ubuntu 10.10 Maverick的所有源代码包，以及所有公共SourceForge C / C ++项目使用版本控制系统。 2011年11月，我们准备了2011年末数据集（Σ2）：Debian 6.0 Squeeze和Ubuntu 11.10 Oneiric的所有源代码包。表I显示了数据集的详细分解。该数据集包含大量以多种语言编写的项目，包括C，C ++，Java，Shell，Perl，Python，Ruby和PHP。

![1568984014988](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568984014988.png)

对两个数据集的简单介绍：

![1568984269638](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568984269638.png)

### 默认参数

diff文件中的默认上下文是3行代码。在未修补的代码克隆的所有实验中，设置θ= 1，需要在未修补的副本中找到来自原始错误代码段的所有n个token才报告错误。m是Bloom匹配器的大小，N是要在Bloom匹配器中进行哈希的n-tokens数量。使用256KB大小的Bloom，利用了3个快速哈希函数：djb2，sdbm和jenkins。

### 实验结果总结

- 为每个源代码数据集创建数据库。下图显示了数据库构建时间。Ubuntu Maverick和Debian Lenny的数据库构建大约需要6个小时。 SourceForge的数据库构建大约需要23.3小时。实验表明，随着源代码大小的增加，构建数据库的时间会线性增加。 一旦ReDeBug构建了初始数据库，就可以通过仅添加/更改数据库的相关部分来快速完成增量更新。

  ![1568984616153](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568984616153.png)

- 表中描述了生成的数据库大小以及数据库中的项目和文件数。

  ![1568984682998](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568984682998.png)

- 描述了向每个数据库查询1,634个与安全相关的补丁的时间。 随着的数据库中的文件数目增长，查询漏洞的时间呈线性增长。 

  ![1568984789666](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568984789666.png)

- 图显示了将不同数量的漏洞与整个数据库进行比较所花费的时间。查询时间具有非常平缓的向上斜率。
  ![1568984889584](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568984889584.png)

- 1）{δ1＆δ2}→Σ1：查询δ1和δ2到Σ1以测量检测到的未修补代码克隆的数量，这近似于当补丁可用时攻击者可能发现的多少（可能）易受攻击的项目。

  ![1568985338433](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568985338433.png) 

- 2）{δ1＆δ2}→Σ2：测量了Σ2中δ1和δ2识别出多少未修补的代码克隆，这个评估大致反映了新版操作系统的对以前发布的安全相关补丁的相应程度。 

- 3）δ1→Σ1对δ1→Σ2：比较了Σ1中的1,838个未修补的代码克隆和Σ2中的1,991个未修补的代码克隆的δ1，我们发现1,379个未修补的代码克隆已经存在。

  ![1568985357772](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568985357772.png)

- 图描绘了发现补丁克隆的频率分布。

  ![1568985407056](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568985407056.png)

- 表显示了具有不同大小n的已识别的未修补代码克隆的数量。

  ![1568985460015](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568985460015.png)

- （然后介绍了一些样例，为啥没检测出来还有成功检测的样例）

- 为了理解代码重复的总量，运行了一个大规模的实验来测量整个Debian Lenny源代码库中的相似性。

  ![1568985637815](https://github.com/sunSUNQ/Paper_reading/raw/master/ReDeBug%20Finding%20Unpatched%20Code%20Clones%20in%20Entire%20OS%20Distributions/image/1568985637815.png)



## 结论

在本文中，我们介绍了ReDeBug，一种专为未修补的代码克隆检测而设计的体系结构。 ReDeBug旨在实现整个操作系统分部Bug发现了15,546个未修补的代码克隆，这可能代表了真正的漏洞。 我们通过在最新版本的Debian Squeeze软件包中确认145个真正的错误来展示ReDeBug的实际影响。 我们相信ReDeBug可以成为常规开发人员在日常开发中增强代码安全性的现实解决方案。

