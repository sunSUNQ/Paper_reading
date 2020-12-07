# Effective and Efficient API Misuse Detection via Exception Propagation and Search-Based Testing

> 2019 ACM University College London，London, United Kingdom，Maria Kechagia



## Abstract

Catcher：一种API滥用检测方法，将静态异常传播分析与基于自动搜索的测试用例生成相结合，可有效，高效地找出客户端应用程序中容易发生崩溃的API滥用。

使用Catcher测试了21个java应用程序，可以自动修复243个由于API滥用引起的程序崩溃。

Catcher可以检测出来77个API的错误使用，EvoSuite没有检测出来。Catcher比EvoSuite快八倍。而且发现，Catcher触发的异常基本都是意料之外的，在文档和源代码级别都没有记录这些异常。



## Introduction

正确的使用API不是一个简单的事情：需要实时更新；需要掌握库的用法以及各个API；需要客户端程序足够健壮。

API的误用会导致一系列的安全问题，对用户输入的验证不足，资源误用，甚至增加攻击面。

静态API分析的主要瓶颈：过多的误报，以及随之而来开发人员大量的人力物力去分析误报。

动态分析工具可以触发程序崩溃，并且基本没有误报。但是必须在巨大的程序执行路径所有空间与时间开销中做出平衡。

本文做出的主要贡献：将搜索空间限定在容易误用的API调用位置，自动测试集可能触发异常的位置。Catcher将静态异常检测分析与生成测试用例结合起来，可以有效的发现应用软件中对API的误用。1、检测可以造成应用软件中的API误用触发crash。2、生成测试样例自动触发这些crash。

首先，使用Soot来进行call调用分析，预测运行时的程序可能有哪些异常没有被捕获，对于调用特定API的callsite，为每一个生成测试用例创建搜索空间。然后，使用传统的代码覆盖率算法和选定的候选误用API用于EvoSuite生成测试样例用于触发异常。

为了验证系统，1、EvoSuite是否可以有效的发现API误用。2、在检测API误用引起的crash上，catcher能否改进Evosuite。3、验证发现的异常，是否在文档或者可以被捕获。

实验：检测了21个java的应用程序对于JAVA jdk 1.8.0_181版本的API误用。Catcher可以发现243个API的误用，其中77个没有被EvoSuite检测出来，并且需要的时间只有20%。

贡献：

- 评估了EvoSuite在使用传统方法进行API误用上的能力和效率的分析。
- 结合静态异常传播分析和基于搜索的软件测试技术，以最大化被测软件中发现崩溃的API滥用的数量，并最小化时间
- 21个Java项目的评估，提出解决方案的有效性和效率。 
- 提供了研究数据以及Catcher的源代码以及对结果进行筛选处理的脚本。



## Background

对于API误用的样例，在使用nnextToken函数的时候没有对st的大小进行check，也没有添加NoSuchElementException异常处理。

如果st小于3个token的情况，则会抛出一个新的异常，引发程序崩溃。

![image-20201023150252784](https://github.com/sunSUNQ/Paper_reading/raw/master/Effective%20and%20Efficient%20API%20Misuse%20Detection%20via%20Exception%20Propagation%20and%20Search-Based%20Testing/images/image-20201023150252784.png)

API误用有很多种，本文主要关注与触发crash相关的误用情况（代表了大多数的误用情况）。有很多的静态分析方法可以分析出来这些位置，但是都存在大量的误报，需要结合动态分析去具体实现触发，减少误报情况。分析API误用的研究工作有很多，本文是第一个针对crash类型的API误用与自动生成测试样例结合起来的工作。



## Catcher approach

![image-20201023152250684](https://github.com/sunSUNQ/Paper_reading/raw/master/Effective%20and%20Efficient%20API%20Misuse%20Detection%20via%20Exception%20Propagation%20and%20Search-Based%20Testing/images/image-20201023152250684.png)

（i）使用soot静态异常传播分析构建了被测API的调用图，包含可能在运行时引发的异常

（ii）使用调用图，使用异常流分析来分析api调用，进而识别滥用行为

（iii）然后使用规则过滤掉，直接处理的传播异常（在方法中使用try catch构造或throws子句）

（iv）剩余的候选滥用行为成为测试用例生成的想要覆盖的目标。 

最后。使用EvoSuite生成的测试套件将包含测试样例，涵盖了应用程序中的目标api调用并触发了异常。



### 3.1 静态异常检测

 3.1.1 构建callgraph

构建**带注释的调用图**，该图的节点表示api中的方法，边表示调用依赖关系。节点用运行时可能触发的异常列表进行注释。并根据客户端和api之间的方法调用将其连接到api的调用图。在生成的全局调用图上，我们识别出第一组候选：带有注释异常的api节点的客户端节点。通过异常传播分析可以发现其他滥用情况。

![image-20201023155952780](https://github.com/sunSUNQ/Paper_reading/raw/master/Effective%20and%20Efficient%20API%20Misuse%20Detection%20via%20Exception%20Propagation%20and%20Search-Based%20Testing/images/image-20201023155952780.png)



### 3.3.2异常传播分析

扩大候选滥用行为的范围：使用可达性分析将异常从api传播到客户端。具体来说，对于api的每个节点Napi，其带注释的异常都向后传播到其所有相邻节点Nj（深度1）。递归地为每个节点Nj完成传播。当遇到全局图中的第一个客户端节点时传播路径结束（深度k）。所有具有从api节点传播的异常的客户端节点都是候选滥用行为，因为它们可能会暴露api抛出的异常。
候选滥用的数量随着我们考虑的深度k呈指数增长。考虑深度k≤4的调用来平衡Catcher。



## 3.2 过滤

可以通过编程语言元素的结合在客户端满足api使用约束。try catch 以及throws两种异常处理可以通过静态分析轻松识别。

（i）对try catch内的api进行调用，以捕获异常传播。
（ii）在方法内对api进行调用，使用throws子句声明异常传播。

如果api方法抛出IOException且具有Exception的catch子句，则我们的规则集将过滤掉相关的候选滥用。 剩余的候选滥用作为输入用于，Catcher基于搜索的测试用例生成。



## 3.3 基于搜索的测试样例生成

问题1：对于class C， M是候选api误用列表，目的是找到T测试样例集，对于C的M滥用列表尽可能多的触发。

如何判断测试样例t成功的触发某个误用m：

（1）t涵盖了类C中的候选api调用，
（2）t触发传播异常，
（3）跟踪崩溃堆中的最后一个跟踪堆元素是误用的api方法。

使用了一个启发式的搜索方法。

3.3.1 启发式

考虑了三种最先进的启发式搜索技术。 

线覆盖率，线覆盖率定义为approach level（al）和归一化分支距离normalized branch distance（bd）的结合：用于测试用例tj和分支bk。 这种启发式方法广泛应用于白盒测试中，它可以计算测试用例tj覆盖api call site的距离。
考虑了包含潜在api错误使用的caller的输入覆盖率icmi（tj，bk）和输出覆盖率ocmi（tj，bk）。 这两种启发式方法是黑盒的，在测试样例生成过程中增加输入和输出多样性，更加多样化的输入/输出可以增加触发意外行为的可能性。

![image-20201028100015113](https://github.com/sunSUNQ/Paper_reading/raw/master/Effective%20and%20Efficient%20API%20Misuse%20Detection%20via%20Exception%20Propagation%20and%20Search-Based%20Testing/images/image-20201028100015113.png)

3.3.2 搜索算法

使用Catcher覆盖尽可能多的候选api滥用是一个多目标问题，因为类可以包含要覆盖的多个候选滥用目标。作为搜索算法选择了动态多目标排序算法（DynaMOSA），这是一种同时优化多个覆盖目标的最新的多目标算法，也有其他研究工作表示，这个方法的效果和效率都更高。

在DynaMOSA中，覆盖率目标与搜索目标相对应，根据被测类的控制依赖关系图中的结构依赖关系进行优先级排序。 搜索从优化级较高位置的目标开始。 当其他目标满足其父目标时（例如，在尝试到达分支n + 1之前先到达分支n），则会在搜索中逐步限制其他目标。 启发式算法，算法执行如下：

Initialization.。搜索从M的候选滥用开始。生成一组随机测试用例以产生初始集合。
Selection。DynaMOSA使用偏好排序。对于每个候选滥用mi，偏好排序会采用最佳的测试用例，并将其插入下一个集合中。然后通过非支配排序算法对其余测试用例进行排序和选择。
Reproduction。每一次迭代中，使用竞争选择来选择父母，并通过应用交叉和变异算子来创建新的测试用例。
Objective update。一旦测试用例到达api call site，就必须抛出mi中的异常，并通过相同的方法传播该异常。当相同的方法触发并传播异常时确认该测试样例为测试样例集的一部分。
Termination。迭代过程一直持续到M被覆盖或搜索时间结束为止。



## 测试

### 4.1 

研究的内容包括Java客户端应用程序和它们使用的第三方api。 我们选择了21个开源Java项目的最新版本，其名称和特征如表1所示。**众所周知的，定期维护的，多样化，并且已经在相关文献中用于评估性能**。他们的源代码和文档可公开获得，以确保结果的可重复性。

![image-20201028110051391](https://github.com/sunSUNQ/Paper_reading/raw/master/Effective%20and%20Efficient%20API%20Misuse%20Detection%20via%20Exception%20Propagation%20and%20Search-Based%20Testing/images/image-20201028110051391.png)

考虑Java jdk（版本1.8.0_181）的api。Java平台是众所周知的，着重研究Java平台的api，方法可以用于验证其他第三方api的用法，包括那些不像Java api那样被广泛记录的第三方api。

### 4.2

RQ1：现有的测试样例生成工具在发现api滥用方面如何发挥作用？

是第一个评估此类工具的人。

RQ2：Catcher是否会提高现有基于测试覆盖率的方法在检测api滥用方面的性能？

只关注异常传播方面，就在生成测试样例的时候大大的减少了程序的搜索空间和时间开销。比较了Catcher和EvoSuite用于api滥用检测的有效性和效率。

RQ3：Catcher主要关注哪种类型的API误用？

constraint misuses and exception handling misuses 约束误用和异常处理误用

### 4.3 基准选取与参数设置

RQ1，选择EvoSuite作为基线。 EvoSuite是最新的用于为Java类生成单元测试套件工具。与其他工具相比，它显示了具有更高代码覆盖率和更好的故障检测能力的测试能力。 EvoSuite实现了各种搜索算法来生成测试用例。使用了动态多目标排序算法（DynaMOSA），是因为它优于其他多目标和单目标方法。

RQ2，比较了EvoSuite和Catcher。 EvoSuite和Catcher使用相同的搜索算法（即Dy naMOSA）和相同的测试用例生成引擎。 EvoSuite和Catcher中的DynaMOSA算法之间的差异取决于它们优化的目标。 前者针对所有源代码元素（例如，分支，行）进行代码覆盖率优化。 后者仅针对候选api滥用（源代码中的特定行）以及包含候选滥用的方法调用的输入和输出范围。

搜索算法需要设置各种参数，可能会影响研究结果。
但是，其他研究表明，基于搜索的软件工程中的参数与默认参数值相比并没有提供实质性的改进。因此使用建议的默认参数值。

## 4.4 实验

表1的最右边一栏中报告了基准测试中每个项目的候选滥用情况。
为了解决EvoSuite和Catcher的随机性，在每个被测类上对每个工具运行了25次。EvoSuite，要测试的类是基准项目中的所有类。Catcher，受测试的类仅是那些被识别为具有候选api滥用的类。

对于Catcher，我们执行了905次（类）×25次（重复）≈22,625次搜索执行，每次执行的搜索预算为3分钟。对于EvoSuite，类别数增加到8,409，对应于≈210,225搜索执行，每个搜索预算有3分钟。

<img src="https://github.com/sunSUNQ/Paper_reading/raw/master/Effective%20and%20Efficient%20API%20Misuse%20Detection%20via%20Exception%20Propagation%20and%20Search-Based%20Testing/images/image-20201028110051391.png" alt="image-20201028110051391" style="zoom:50%;" />

在每次运行中收集了生成的测试套件以及完成搜索所需的总运行时间。 我们使用收集的数据来回答RQ1和RQ2。 每次搜索结束时重新执行生成的测试套件，以识别触发异常的测试用例。 然后，我们将相应的崩溃堆栈跟踪与通过异常传播分析确定的候选api滥用列表进行了比较。 如果Catcher和EvoSuite生成了一个测试用例tj，它触发了从API传播到客户端的异常（与mi相同），则暴露了目标滥用mi。 换句话说，该检测要求满足以下两个条件：（i）由tj触发的异常的名称与传播的异常mi的名称重合； （ii）mi的调用站点链出现在由tj触发的异常的堆栈跟踪中。

为了评估RQ1和RQ2的有效性，计算了依赖运行中Catcher和EvoSuite暴露的滥用数量。为了衡量效率，我们计算了Catcher和EvoSuite在每次独立运行中对每个项目所花费的总执行时间。 

Catcher运行时间：（1）静态异常分析确定潜在的api滥用所需的时间，（2）测试用例的生成时间和（3）后处理。

EvoSuite运行时间包括（1）搜索预算和（2）后处理。

后处理，删除测试用例中对覆盖率没有贡献或不会触发异常的语句

注意，Catcher使用了EvoSuite的后处理引擎。


RQ3，通过分析并重新执行生成的测试来检查Catcher检测到的api滥用情况，并检查客户端应用程序中的api调用程序以及滥用的api本身的源代码和文档（Javadoc）。

使用脚本来部分自动化分析，该脚本检查所传播的异常是否被充分记录（i）在Java jdk的api javadoc中和（ii ）客户端应用程序中调用函数的源代码注释中的。

# result

RQ1。表2报告了在25个运行中检测到的每个项目的中位数，四分位间距（IQR）和API错误使用的总数。在21个测试项目中，EvoSuite平均可以检测到123个与崩溃相关的api滥用。如果我们考虑在25次运行中至少一次检测到所有api滥用，则检测到的滥用总数为166。尽管Evo Suite可以通过最大化代码覆盖率来检测某些滥用，但是对于某些项目，结果的可变性非常高。因此，在两个独立的运行之间情况差异很大。为了获得更可靠的结果需要多次运行EvoSuite，并相应增加整体运行时间。

![image-20201028145319429](https://github.com/sunSUNQ/Paper_reading/raw/master/Effective%20and%20Efficient%20API%20Misuse%20Detection%20via%20Exception%20Propagation%20and%20Search-Based%20Testing/images/image-20201028145319429.png)

让我们考虑一下图3中的示例，该示例由Catcher而非EvoSuite针对项目apache-commons-math中的类KthSelector检测到api滥用。 ArrayIndexOutOfBounds异常在api Arrays.rangeCheck的第120行引发，并在select方法中传播回客户端应用程序。执行测试用例时，客户端方法select使用大小为17的数组和变量begin = 12和end = 19作为参数来调用Arrays.sort方法。变量end的值大于数组的大小，在Arrays.rangeCheck中引发了异常。注意，这两个变量的值是在第84-113行的while循环内计算的。在调用api之前，客户端选择的方法应该验证输入数据（例如，数组的长度）。

![image-20201028150441185](https://github.com/sunSUNQ/Paper_reading/raw/master/Effective%20and%20Efficient%20API%20Misuse%20Detection%20via%20Exception%20Propagation%20and%20Search-Based%20Testing/images/image-20201028150441185.png)

表4列出了Catcher和EvoSuite所检测到的api滥用次数，以及一种方法（例如Catcher）所检测到的误用次数，而另一种方法（例如EvoSuite）却未检测到。 我们观察到， 通过手动调查发现EvoSuite可以检测到这两种滥用，这归功于weak mutation coverage弱突变覆盖率。 未来的工作将致力于调查Catcher中的其他覆盖标准，包括弱突变覆盖率。

![image-20201028151113777](https://github.com/sunSUNQ/Paper_reading/raw/master/Effective%20and%20Efficient%20API%20Misuse%20Detection%20via%20Exception%20Propagation%20and%20Search-Based%20Testing/images/image-20201028151113777.png)

表3报告了Catcher和Evo Suite的运行时间。 catcher所需的时间不到Evo Suite总花费时间的20％。 在每个项目的基础上，Catcher的平均速度提高了80％，而项目整齐的速度最高可提高96％。

![image-20201028151346319](https://github.com/sunSUNQ/Paper_reading/raw/master/Effective%20and%20Efficient%20API%20Misuse%20Detection%20via%20Exception%20Propagation%20and%20Search-Based%20Testing/images/image-20201028151346319.png)

RQ3。根据我们的定性分析结果，我们确定了api参考文档和客户端应用程序中api使用之间的三种类型。表5列出了每种类型的滥用数量。

类型＃1。第一种滥用类型是**完整的api文档to不正确的客户端**。此类别包括api方法的文档中列出的传播的异常。但是，这些异常没有再调用函数中处理，通过检查条件或try catch构造来处理引发的异常，也没在客户端应用程序的Javadoc中进行记录。Catcher触发的绝大多数误用（82％）属于这一类别。

类型＃2。第二种类型的滥用是**不完整的api文档–未觉察客户端**。该类别包括api参考文档中未列出的传播异常，这可能导致客户端应用程序开发人员滥用api。发现大约10％的误用属于此类别。

类型＃3。第三类滥用是指**完整的api文档to一致客户端**。此类包括api方法文档中列出的传播的异常，但是客户端明确选择不在源代码中处理它们。客户端应用程序的开发人员知道传播的异常，并将其列在文档中。但api滥用仍然保留在源代码中，并可能导致崩溃。发现大约8％的误用属于此类。

# discuss

新的研究可能会考虑检测不同类型的api滥用，例如与安全性和能源效率问题相关的滥用。

将EvoSuite与Catcher中实现的静态扩展传播分析相结合，比仅使用默认标准更为有效。因此，我们选择结合静态分析和基于自动搜索的测试用例生成优势的更多研究。



# conclusion

一种验证技术Catcher，该技术结合了静态异常传播分析和基于搜索的测测试样例生成，可以有效快速的识别客户端程序中的api滥用。我们针对21种Java应用程序进行了验证，我们的结果表明，Catcher能够有效地生成测试案例，以发现243个api滥用导致崩溃的情况。收集的结果表明，Catcher可以发现更多的api滥用（77例），而EvoSuite所花费的时间还不到20％。总体而言，静态异常传播分析和基于搜索的测试相结合可以显着提高api滥用的检测能力。
将来目标是在以下方面扩展Catcher：（i）支持更长的异常传播链，以涵盖深层嵌套的api调用；（ii）在分析中考虑第三方库，以涵盖它们引入的运行时异常处理，（iii）将Catcher扩展为涵盖其他类型的api滥用。
