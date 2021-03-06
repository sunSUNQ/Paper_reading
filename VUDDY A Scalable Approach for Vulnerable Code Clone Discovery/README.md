# VUDDY: A Scalable Approach for Vulnerable Code Clone Discovery

> Seulbae Kim, Seunghoon Woo, Heejo Lee, Hakjoo Oh 
>
> Department of Computer Science and Engineering Korea University

## 摘要

开源软件（OSS）的生态系统规模不断扩大。代码克隆指的是在软件系统内或之间复制和粘贴的代码片段 。尽管代码克隆可能会加速软件开发过程，但它通常会严重影响软件的安全性，因为漏洞很容易通过代码克隆传播。这些易受攻击的代码克隆随着OSS的增长而增加，可能污染许多系统。尽管数十年来研究人员一直试图检测代码克隆，但大多都无法扩展到不断增长的OSS代码库的大小。缺乏可扩展性使软件开发人员无法轻松管理代码克隆和相关漏洞。此外，大多数现有的克隆检测技术过度关注于检测克隆存在，这削弱了它们准确找到“易受攻击”克隆的能力。
​在本文中，我们提出了**VUDDY，一种可扩展的，用于检测易受攻击的代码克隆的方法**，它能够高效，准确地检测大型软件程序中的安全漏洞。它的极端可扩展性是通过利用功能级粒度和长度过滤技术来实现的，该技术可减少签名比较的数量。这种高效的设计使VUDDY能够在14小时17分钟内预处理数十亿行代码，之后只需要几秒钟来识别代码克隆。此外，我们设计了一种安全感知抽象技术，使VUDDY能够适应克隆代码中的常见修改，同时即使在应用抽象后也能保留易受攻击的条件。这将VUDDY的范围扩展到识别已知漏洞的变体，具有高精度。在本研究中，我们通过利用现有机制比较来描述其原理并评估其有效性和有效性，并展示其检测到的漏洞。 VUDDY在可扩展性和准确性方面优于四种最先进的代码克隆检测技术，并通过检测广泛使用的软件系统（如Apache HTTPD和Ubuntu OS Distribution）中的0day漏洞证明了其有效性。

## 背景介绍

代码克隆的存在它会增加软件维护成本，降低代码质量，产生潜在的法律冲突，甚至传播软件漏洞。即使供应商在原始程序中发现漏洞后立即发布补丁，也需要一段时间才能使克隆原始程序易受攻击代码的每个程序完全部署补丁程序。许多研究人员提出了代码克隆检测技术来解决与克隆相关的问题。 然而，很少有技术支持可扩展的方式准确地发现漏洞。

我们提出了VUDDY（VUlnerable coDe clone DiscoverY），一种用于代码克隆检测的可扩展方法。 该方法可以在大量代码库中准确地发现漏洞。 为了从安全角度实现高度可扩展但准确的代码克隆检测的目标，我们将程序中的功能用作代码克隆检测的单元。 由于函数同时提供了代码的语法信息和符号信息，因此能够保证安全性问题检测克隆的能力。同时由于抽象的设计以及规则的设计，VUDDY也可以检测未发现的漏洞。独特的分类器的设定还可以缩小对于函数体的搜索空间，VUDDY可以检测大规模的代码量。

## 主要贡献

- 可扩展的克隆检测：我们提出了“VUDDY”，一种可扩展但准确的代码克隆检测方法，它采用了强大的解析和新颖的功能指纹识别机制。 VUDDY在14小时17分钟内处理了十亿行代码。
- 保留漏洞的抽象：我们提出了一种优化的有效抽象方案，这允许VUDDY检测未知的易受攻击的代码克隆，以及目标程序中的已知漏洞。VUDDY检测到24％的易受攻击的克隆是已知漏洞的未知变种。
- 自动化漏洞获取：我们引入了一种全自动方法，通过利用安全补丁信息从Git存储库中获取已知的易受攻击的函数。
- 开放服务：自2016年4月起，为VUDDY提供免费开放式服务。在实践中，VUDDY正在开源社区和物联网设备制造商中使用检查他们的软件。在过去的11个月中，开放服务查询了140亿行代码，检测到144,496个易受攻击的函数。请参阅https://iotcube.net/。

### 代码克隆分类

1. 完全克隆：未更改任何位置，直接复制粘贴
2. 重命名克隆：语法标识克隆，除了类型，标识符，评论和空格其他未修改。
3. 重构克隆：进一步的结构修改（例如，删除，插入或重新排列语句）应用于重命名的克隆以产生重组的克隆。
4. 语义克隆：有相同或者相似的功能。
5. **VUDDY涵盖第一种和第二种。**

### 代码检查单元

1. 字符：int、i、=、0、；
2. 行：int i=0；
3. 函数：一个函数体
4. 文件：代码文件。
5. 项目：一整个完整的项目。
6. **VUDDY以函数为单位**

## 主要方法介绍

![1568963248911](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568963248911.png)

### S1 Function retrieval（函数提取）

VUDDY使用解析器来进行函数提取，执行语法分析用来识别形参，使用的数据类型，局部变量和函数调用。

### S2 Abstraction and normalization（抽象和正规化）

因为只检测类型1和2两种代码克隆方式，因此应该在比对函数指纹之前进行代码的抽象操作， 分成多个抽象等级。

- 0级（No abstraction）：为了检测类型1的代码克隆，不对代码进行抽象。

  ![1568966810268](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568966810268.png)

- 1级（Formal parameter abstraction）：获取函数体中所有的参数，并将所有的参数替换为FPARAM

  ![1568966820210](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568966820210.png)

- 2级（Local variable abstraction）：替换所有的变量为LVAR。

  ![1568966829451](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568966829451.png)

- 3级（Data type abstraction）：代替所有的数据类型为DTYPE。

  ![1568966838713](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568966838713.png)

- 4级（Function call abstraction）：代替所有的函数调用名为FUNCCALL。

  ![1568966848298](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568966848298.png)

然后删除注释，空格，制表符和换行符以及将所有字符转换为小写来对抽象的函数体进行标准化。 

### S3 Fingerprint generation（指纹生成）

每一个函数的指纹表示成一个二元组，标准化后的函数字符串作为其中一项，字符串的hash作为另外一项。

![1568967029819](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568967029819.png)

VUDDY将元组存储在dict中，将键映射到值，其中长度值（即元组的第一个元素）是键，而共享相同键的哈希值则映射到每个键。长度少于50的忽略掉。具体实例如下：

![1568967094649](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568967094649.png)

### S4 Key lookup（关键字匹配）

VUDDY通过迭代dict中的每个键，在目标指纹字典中查找密钥的存在（即，预处理函数的长度）来执行第一次测试。如果查找失败，则断定目标程序中没有克隆。 如果成功找到，进入下一个子步骤：哈希查找。

平均复杂度O(1)，最坏O(n)。

### S5 Hash lookup（哈希值匹配）

搜索映射到整数键的集合中是否存在哈希值。 如果发现了哈希值，则该函数被认为是克隆。

## 具体实现

### 数据集来源

检测从小到大的真实项目漏洞。 为了在建立漏洞数据库时从可靠的软件项目中获取易受攻击的功能，我们利用着名和权威的开源项目的Git存储库：Google Android，Codeaurora Android Project，Google Chromium Project，FreeBSD，Linux Kernel，Ubuntu Trusty ，Apache HTTPD和OpenSSL。 然后从目标程序中搜索易受攻击函数的代码克隆。

### 建立数据库

收集易受攻击的代码和建立漏洞数据库的过程是完全自动化的。从Git提交日志重构易受攻击功能的过程包括以下步骤：
1）git clone ：将指定的Git存储库下载到本地目录中。
2）git log --grep ='CVE-20'：将搜索有关漏洞和CVE的提交。
3）git show：将显示完整的提交日志，其中包含与CVE相关的漏洞的描述，以及统一diff格式的安全修补程序信息。
4）filter commits：列出可能会提取不适合漏洞检测的提交。
5）git show：显示了该文件的旧的未修补版本。然后从文件中检索易受攻击的函数。

原补丁信息以及提取出的易受攻击的代码片段。

![1568968112407](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568968112407.png)

将相同的方法应用于9,770个漏洞补丁，收集了5,664个易受攻击的函数，这些函数可以处理1,764个独特的CVE。这些易受攻击的函数的漏洞涵盖多种类型，例如缓冲区溢出，整数溢出，输入验证错误，与权限相关的漏洞等。 最短的易受攻击的函数在抽象和规范化之后由51个字符组成。单行函数（例如，通过调用另一个函数返回的保护函数）被排除在数据库之外，因为这些函数在应用抽象时经常导致误报。

### 实施方案选择

代码分析器：使用ANTLR来构建代码分析器。

哈希算法选择：MD5哈希算法。

dict生成：python内置字典。

## 实验部分

### 环境

在Ubuntu 16.04的机器上进行实验来评估VUDDY的执行和检测性能，该机器具有2.40 GHz Intel Zeon处理器，32 GB RAM和6 TB HDD。

### 数据集

从GitHub收集了目标C / C ++程序。 在2016年1月1日至2016年7月28日期间发布或者更新并且至少一个star的项目。 存储库克隆过程7周完成，收集25,253个Git存储库。 除了Github项目还下载了几款Android智能手机的固件。

### 实验过程

- 效率评测

  首先，我们根据不同的目标程序大小，针对四种公开可用的技术（SourcererCC，ReDeBug，DECKARD和CCFinderX）评估了VUDDY的可扩展性。关注工具的可扩展性时处理真实项目，通过从收集的25,253个Git项目中随机选择项目，生成不同大小的目标集，从1 KLoC到1 BLoC。 所有实验每次迭代五次（SourcererCC除外，我们迭代两次）以确保结果可靠。实验结果以及不同工具的配置选择如下（说明VUDDY效率高，并且具备对大规模代码的处理能力）：

  ![1568969276067](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568969276067.png)

  ![1568969321435](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568969321435.png)

  ![1568969584748](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568969584748.png)

- 准确率评测

  使用每种技术进行克隆检测，然后手动检查每个报告的克隆。

  ![1568969599565](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568969599565.png)

- 和ReDeBug对比实验

  ![1568969727698](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568969727698.png)

  ![1568969747935](https://github.com/sunSUNQ/Paper_reading/raw/master/VUDDY%20A%20Scalable%20Approach%20for%20Vulnerable%20Code%20Clone%20Discovery/image/1568969747935.png)

## 结论

在本文中，提出了VUDDY，这是一种可扩展且精确的易受攻击代码克隆的发现方法。VUDDY的设计原则旨在通过功能级粒度和长度过滤器扩展可扩展性，同时保持准确性，以便能够从快速扩展的开源软件项目中检测易受攻击的克隆情况。VUDDY采用漏洞抽象方案，使其能够发现24％以上的未知漏洞变种。VUDDY实验证明其功效和有效性。结果表明，VUDDY实际上可以从大型代码库中检测出众多易受攻击的代码克隆，具有前所未有的可扩展性和准确性。在案例研究中，我们介绍了VUDDY发现的几个案例。我们坚信，当需要可扩展性和准确性时，VUDDY是用于保护各种软件的必备方法。
我们的工作可以扩展到多个方向。首先，计划通过改进解析器和扩展漏洞数据库来继续提高VUDDY的性能。它将提高VUDDY的速度并提高检测率。此外，我们将尝试将我们的方法与其他类型的漏洞检测技术（例如，模糊器）相结合，这将允许更复杂的漏洞检测。
