---
title: 论文粗读——《The Evolved Transformer》
date: 2020-02-05 15:44:27
tags: [Transformer, 论文阅读]
---

今天阅读了论文《The Evolved Transformer》，该论文使用了神经架构搜索方法找到了一个更优的transformer结构。下面是阅读过程中的笔记。

<!--more-->

> 参考：
> https://arxiv.org/pdf/1901.11117.pdf
> https://github.com/tensorflow/tensor2tensor/blob/master/tensor2tensor/models/evolved_transformer.py
> https://blog.csdn.net/jasonzhoujx/article/details/88875469

## Introduction

### 主要贡献

- 神经架构搜索
  - tournament selection architecture search 
  - warm start
  - Progressive Dynamic Hurdles（PDH）
- 搜索出了一个新的transformer架构：Evolved Transformer

## Methods

### 搜索空间

- encoder stackable cell
  - 6个NASNet-style block
    - 左右两个block将输入的hidden state转成左右两个hidden state再归并成为一个新的hidden state，作为self-attention的输入
- decoder stackable cell
  - 8个NASNet-style block

![图片](<http://q503tsu73.bkt.clouddn.com/et-transformer.png?e=1580892803&token=05Ii263bPN3Z-CT3JPRaRfWi5sXIj8pwX6V1bN2j:ln2axf1iGaz9DI2q6j1kLn1lAx8=&attname=>)

- 搜索空间branch
  - Input：分支可以从输入池中选择一个隐藏状态作为当前block的输入。单元中的第i个block可以从[0, i]个隐藏状态中进行选择，其中第j个隐藏状态表示该cell中第j个block的输出，第0个候选项为单元的输入。
  - Normalization：归一化项提供了两个选项， [LAYER NORMALIZATION (Ba et al., 2016), NONE]
  - Layer：构造一个神经网络层，提供的选项包括：
    - 标准卷积
    - 深度可分离卷积
    - LIGHTWEIGHT 卷积
    - n头注意力层
    - GATED LINEAR UNIT
    - ATTEND TO ENCODER（decoder专用）
    - 全等无操作
    - Dead Branch，切断输出
  - Relative Output Dimension：决定神经网络层输出的维度。
  - Activation：搜索中激活函数的选项有[SWISH, RELU, LEAKY RELU, NON]
  - Combiner Function：表征的是左枝和右枝的结合方式，包括{ADDITION、CONCATENATION、MULTIPLICATION}。如果左右枝最终输出形状不同，则需要使用padding进行填充。短的向量向长的向量对齐，当使用加法进行结合时使用0填充，当使用乘法进行结合时使用1填充。
  - Number of cells：纵向叠加的cell的数量，搜索范围是[1,6]

### 演进过程

- 锦标赛选择（Tournament Selection）：
  - tournament selection算法是一种遗传算法，首先随机生成一批个体, 这些个体是一个个由不同组件组成的完整的模型，我们在目标任务上训练这些个体并在验证集上面计算他们的表现。
  - 首先在初始种群中进行采样产生子种群，从子种群中选出适应性（fitness）最高的个体作为亲本（parent）。被选中的亲本进行突变——也就是将网络模型中的一些组件改变为其他的组件——以产生子模型，然后在对这些子模型分配适应度（fitness），在训练集和测试集上进行训练和验证。
  - 对种群重新进行采样，用通过评估的子模型代替子种群中的fitness的个体以生成新的种群。
  - 重复上面的步骤，直到种群中出现超过给定指标的模型。
- 渐进式动态障碍（Progressive Dynamic Hurdle）：
  - 实验使用的训练集是WMT14英语到德语的机器翻译数据集，完整的训练和验证过程需要很长的时间，如果在所有的子模型上进行完整的训练和验证过程将会耗费很大的计算资源。因此论文中使用渐进式动态障碍的方法来提前停止一些没有前景的模型的训练，转而将更多的计算资源分配那些当前表现更好的子模型。具体来说就是让当前表现最好的一些模型多训练一些step。
  - 假设当前种群经过一次锦标赛选择，生成了m个子模型并且加入到了种群中，这时候计算整个种群fitness的平均值h0，下一次锦标赛选择将会以h0作为对照，生成的另外m个fitness超过h0的子模型可以继续训练s1个step，接着进行种群中的所有的其他个体会继续训练s1个step，然后在新的种群中生成h1，以此类推知道种群中所有的个体的训练step都达到一个指定值。
  - 如果一个子模型是由第iii次锦标赛选择之后的亲本生成的，那么验证的过程将会进行iii次。第一次为该模型分配s0次的训练step并且在验证集上进行验证，若验证的fitness大于h0则再分配s1次训练step，再验证，再与h1比较，只有子样本通过h0,h1,...,hi次比较才能作为新的个体加入到新的种群中。

## Experiment

- 机器翻译
  - 在初始的10K step使用0.01的learning rate
  - Transformer
    - inverse-square-root decay to 0 at 300K steps：$l r=s t e p^{-0.00303926^{\circ}}-.962392$

  - Evolved Transformer
    - single-cycle cosine decay
  - every decay was paired with the same constant 0.01 warmup.
  - 大模型使用高一点的dropout（0.3），小模型使用0.2 dropout
  - beam-size=6, lenth-penalty=0.6, max-output=50
- 语言模型
  - 跟机器翻译差不多，去掉了label smooth, intra-attention dropout=0.0
- Search Configuration
  - populatino=100
  - mutation=2.5%
  - fitness: negative log perplexity

## Result

- 最终搜索出来的模型结构

![图片](<http://q503tsu73.bkt.clouddn.com/et-transformer2.png?e=1580893009&token=05Ii263bPN3Z-CT3JPRaRfWi5sXIj8pwX6V1bN2j:aAFv3BbiOEmd1YLT9aabOYCYURE=&attname=>)

- embedding_size=768, 6 encoder, 6 decoder
- attention_head=16
- ET比Transformer可以在更小的模型上达到更好的效果，当模型增大时两者的差距就不大了（可能因为模型越大越容易过拟合，而且单独增加embedding_size可能不起作用，需要和depth共同增加）