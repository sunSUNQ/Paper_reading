# Learning a Classifier for False Positive Error Reports Emitted by Static Code Analysis Tools

> *MAPL’17*   University of Maryland, College Park, USA  Ugur Koc

## Abstract

本文提供了一种新的学习过程，可以预测工具输出的报告是否为误报。

preprocess code -> location of error report -> machine learning techniques -> discover correlations, classifier -> predict

## Introduction

Static Code Analysis(SCA)，现有的工具都存在大量的误报问题，人力分析的开销很大，前人的工作有很多，但是都没有加入报告包含部分代码的结构信息。

1. 首先对代码进行缩减：1、只选用包含漏洞的command line的函数体部分。2、从command line开始后向切片
2. 对于对获取的代码信息进行机器学习
3. 分类器再次进行筛选：advanced machine learning

## Approach

![image-20200312194035157](.\image-20200312194035157.png)

### 2.1 code preprocessing

从之前的工作中分析发现，很多漏洞的涉及到很多代码段，因此需要人工的筛选，然后本文主要进行的是自动化的筛选。

method body：从command line 提取整个函数体

program slicing：使用WALA进行后向切片，从command line开始

### 2.2 learning

两个学习过程：

1. Naive Bayes （Markov property）
2. LSTM（卷积神经网络）

## Case study

### 3.1 SQL injection

只关注于sql注入类型的漏洞，使用FindSecBugs工具作为分析目标。

sql注入类型的漏洞通常都是由于执行的sql语句是从HTTP Cookie中获取的或者是输入获取到的，因此可以通过数据流分析发现。

### 3.2 Data

在Juliet和Owasp两个数据集中选取了Owasp数据集，因为他代码量更多，更加复杂。

### 3.3 Preprocessing

使用Bytecode字节码来表达程序。

首先，忽略异常、基指针、堆结构。然后，entry point尽可能的开进warn函数，对于owasp数据集来说就是将有问题的函数作为入口点。最后，去掉第三方库。

为了方便适用范围更广，取消特殊的字符。

LSTM 分类器：删除 literal expressions 代替specific tokens，data用空格处理，调用指令和被调用指令用token代替

Naive Bayes 分类器：删除除了调用指令的所有参数，调用指令代替为invoker

## Results and Analysis

将数据集分成80%用于训练，20%用于测试。

Recall：正确分类的误报/所有误报

Precision：正确分类的误报/确认误报

Accuracy：正确分类的样本数量

![image-20200312200300167](.\image-20200312200300167.png)

## Conclusion and Future work

提出了一种学习过程，可以对静态分析工具的报告进行误报确认，进一步筛选误报。

使用WALA对command line位置进行后向切片

只选取command line所在的函数体

将两段代码进行不同的正则化处理，删除无用的结构，参数，并进行统一的表示。

用到了两种学习算法，朴素贝叶斯和LSTM。