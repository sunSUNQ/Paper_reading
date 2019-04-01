# code2vec: Learning Distributed Representations of Code



## 发表会议

46th ACM SIGPLAN Symposium on Principles of Programming Languages (POPL 2019)

## 作者信息

[Uri Alon](http://urialon.cswp.cs.technion.ac.il/), [Meital Zilberstein](http://www.cs.technion.ac.il/~mbs/), [Omer Levy](https://levyomer.wordpress.com/) and Eran Yahav

>* Technion
>* Facebook AI Research

## 摘要

​	我们提出了一种神经模型，用于将代码片段表示为连续分布式向量（embeddings”）。主要思想是将代码片段表示为一个固定长度的向量，可用于预测代码段的语义。通过将代码分解为其抽象语法树AST中的路径集合，并学习聚合它们的同时学习每个路径的原子表示来实现的。
​	可以通过预测方法的准确性来证明我们的方法的有效性。通过在含14M个方法的数据集上训练模型来评估我们的方法。我们训练的模型可以预测到训练数据集中不存在的方法名。此外，我们展示了经过我们训练过的模型是可视化的，并且是可解释的。
​	通过比较相同数据集的相关工作与技术，我们的方法有超过75％准确率的改进，成为**第一个基于大型跨项目语料库成功预测方法名称的方法**。我们训练有素的模型，可视化和矢量相似性可在https://code2vec.org/  上以互动在线演示的形式获得。代码，数据和训练模型可在https://github.com/tech-srl/code2vec 获得.

## 背景介绍

​	想要使用机器学习的方法，前提是将输入处理成向量的模式。对于整个项目文件进行编码不现实，需要的数据集量太大，而且训练的效果也不好，需要大量的CPU与内存资源来支持，因此大多方法都是将代码以文件、函数、代码段的形式来进行训练。本文使用的就是代码片段，也就是函数级别的（code snippet）。

​	之后将code snippet转化为向量，如何获取语义与语法信息：使用AST（抽象语法树）来表示代码段的语法信息，通过将代码转换成<Xs, p, Xt>的形式，表示AST中的一条从叶子节点到另一个叶子节点的路径也就是一条path-context（路径上下文）信息，将这些path-context整合到一个上下文信息包中。然后通过一个全连接层，得到每条路径的attention（权重）信息，以及一个可以代表code snippet代码段的code vector，然后可以使用这两组数据进行模型的训练，然后可以用训练好的模型来预测方法名。

​	扩展：可以应用到检测代码段的含义，恶意代码识别等用处。



## 结果展示

![效果展示](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/result_show.png)

![1554040374206](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/result_AST.png)

​	对于相似的函数内容也可以很好的辨别出不同的语义，会给出多种上下文路径进行预测。同时可以看到在代码段的标出，不同的线表示不同的path-context，线的粗细表示attention程度。



## 主要方法

### path-context

1. 分析代码生成AST
2. 遍历AST，提取AST叶子节点之间的语法路径
3. 每个路径表示成一个AST节点序列，用“↑、↓”表示父子关系
4. 路径组合+首尾的叶子节点，形成path-context

```
(elements, Name↑FieldAccess↑Foreach↓Block↓IfStmt↓Block↓Return↓BooleanExpr, true)
```



### 对上下文的分布式表示

1. 每一条路径和path-context中的叶子节点的值以及路径都被映射到一个向量中，三个向量
2. 每个path-context的三个向量集成为一个向量来表示
3. embedding和attention一起训练得出

### path-attention network

1. 集合了多个path-context的信息，一个向量表示一个代码段
2. 每条路径都包含了attention值，重要的路径关注度大
3. 将attention以及path-context一起表示，实现了模型的可解释性



## 模型解释

![1554041057113](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/model.png)

### 代码表示

![1554041208517](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/path_context.png)

- value_vocab：叶子结点的值的字典
- path_vocabj：path_context的字典
- 全连接层：通过一些数学变换，将三个向量整合成一个定长的向量。
- attention weights：在训练code vector的同时获取到每条路径的重要程度并进行权重赋值，所有的权重和为1.
- tages_vocab：提取出来的函数名标签，用于训练模型



## 数据集

![1554082510075](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/dataset.png)

使用了 10072个Github上开源的java语言仓库，选择了star最多的，关注度最高的。并对所有的项目文件进行拆分打散。

## 实验结果

![1554041630775](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/comparation.png)

- 实验结果对比其他的模型方法来说效果更好，训练时间更短，同时数据集也更复杂。

![1554041703074](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/attention_compare.png)

- 对比与对于每个path-context使用相同权重，与只关注重要的几个path-context与使用细粒度的element-wise来说，在效率保证的情况下，选择我们的模型具有很优秀的效果，同时还具有可解释性。

![1554041816850](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/path_compare.png)

- 使用开始叶子结点-路径-结束叶子结点的三元祖的模式来表示path-context，对比于其他的方法效率

![1554041923238](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/detail_show.png)

![1554041935604](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/detail_AST.png)

- 使用的向量近似方法：

![1554041983833](https://github.com/sunSUNQ/Paper_reading/raw/master/code2vec/image/rules.png)



## 总结
* 代码片段可以有效的表示为上下文路径包
* attention神经网络可以识别多个路径上下文的权重，将他们聚合起来
* 代码片段中有细微的差别也可以区分，并实现预测
* 大型的、项目交叉的代码也可以识别
* 可解释的
