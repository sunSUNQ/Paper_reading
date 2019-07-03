# Gadget Inspector（Automated Discovery of Deserialization Gadget  Chains）

> 作者：Ian Haken  Senior Security Software Engineer, Netflix 

## 前言

​		随着反序列化漏洞的层出不穷，发序列化漏洞从影响很小到如今的报出很多高危的漏洞，反序列化漏洞的相关研究一直有很多。本文针对是**什么使反序列化漏洞可利用，什么样的反序列化漏洞可以利用，以及如何评估给定程序中反序列化漏洞的风险**这三部分进行研究。主要关注与Java语言的反序列化漏洞，但是相关的思想也可以应用于C#以及PHP。主要的贡献就是一个**开源的用来构造可利用的反射链自动化工具**，极大的缩减了人工构造反射链的时间。

## 已有工具对比

- Ysoserial、Marshalsec：只能针对已有的漏洞进行构建payload
- Java Deserialization Scanner：从已知的Ysoserial中的反射链进行扫描
- NCC Group Burp Plugin：主要关注与JSON的payload
- Joogle：大量的人工操作，自动化低

## 新工具需要满足的条件

- 确定classpath上存在哪些反射链可以进行利用
- 确定这些漏洞可能产生的影响（例如RCE，SSRF，DoS等）
- 提供（有限的）高估影响而不是低估
- 轻松操作应用程序的整个classpath; 给定多种源语言（如Groovy，Scala，Clojure，Kotlin等），它应该在Java字节码上运行
- 理解不同的反序列化库以及每个库可能有的反射链的限制

## Gadget Inspector

![1562071570259](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562071570259.png)

- 输入是一整个程序或者是一个库文件
- 输出就是一段一段的反射链

### STEP1： Enumerate class/method  hierarchy

- 生成类继承的层次结构以及方法重写的层次结构（JDK反射API可以轻松完成此步骤）

![1562071877738](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562071877738.png)

### STEP2：Passthrough Dataflow Discovery

- 默认分支都可到达，默认分支可控就分支下可控

![1562073736842](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562073736842.png)

### STEP3：Passthrough Callgraph Discovery 

- 函数参数和方法调用都创建一个数据流分析，使用符号执行就可以实现。

![1562073804800](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562073804800.png)

### STEP4：Gadget Chain Source Discovery

- 使用第一步生成的层析结构信息来获取调用链的源方法（source method）

![1562073923644](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562073923644.png)

### STEP5：Call Graph Search 

- 从第三步和第四步中获取的所有进行进行搜索，获取所有满足条件的调用链。
- 需要基于“interesting method”列表，一旦发现列表中的方法就输出反射链。

![1562073998310](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562073998310.png)

## 实验结果

- 从mvnrepository.com以及javalibs.com网站上获得的100个最流行的java库做实验。

![1562119654263](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562119654263.png)

- 可以很好的发现之前的已经公布反射链的存在

![1562120010711](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562120010711.png)

- 发现了一些新的反射链的存在

![1562120080963](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562120080963.png)

![1562120109879](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562120109879.png)

![1562120123190](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562120123190.png)

![1562120139985](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562120139985.png)

![1562120563674](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562120563674.png)

![1562120550215](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562120550215.png)

![1562120595977](https://github.com/sunSUNQ/Paper_reading/raw/master/Automated%20Discovery%20of%20Deserialization%20Gadget%20%20Chains/image/1562120595977.png)

## 结论

​		鉴于关于反序列化漏洞的研究和演示不断出现，很明显这类漏洞仍然没有消失。Gadget Inspector，反射链可能更加复杂和微妙，也会进一步的推进研究进展。我们认为，在反序列化漏洞研究中，使用工具进行端到端自动发现新的或特定于上下文的反射链是未开发的领域。使用所描述的方法，我们已经在开源库中发现了许多新的，高危的反射链。Gadget Inspector还使我们能够快速识别Netflix内部服务中反序列化漏洞，并对其进行适当的优先级排序。在分析应用程序时，安全研究人员需要花费数天和数周的时间来构建反射链，但是Gadget Inspector能够在几分钟内生成结果。因此我们相信Gadget Inspector有显着缩短评估反序列化漏洞风险的时间以及缩短创建反序列化漏洞利用所需的时间。Gadget Inspector是一个原型工具，具有上述许多限制和假设，但是是开源的，我们鼓励研究人员提供反馈，提交贡献以改进它，或利用这些想法构建更强大的工具。

## 参考链接

[BlackHat USA 2018 | 次日议题精彩解读](https://www.anquanke.com/post/id/155464#h3-10)

[Gadget Inspector](https://github.com/JackOfMostTrades/gadgetinspector)
