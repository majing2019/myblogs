---
title: 论文阅读——《Towards a Human-like Open-Domain Chatbot》
date: 2020-02-04 21:17:57
tags: [Chatbot, 论文阅读]
categories: [技术]
---

今天阅读了谷歌最新出的一篇论文，《Towards a Human-like Open-Domain Chatbot》，主要提出了端到端对话机器人的一种评测方法和模型框架。下面是阅读过程中的笔记。

<!--more-->

> 参考：
> https://ai.googleblog.com/2020/01/towards-conversational-agent-that-can.html
> https://arxiv.org/pdf/2001.09977.pdf
> https://github.com/google-research/google-research/tree/master/meena

## Introduction

### 开放的chatbot API总结

- cleverbot API: [https://www.cleverbot.com/api/](https://www.cleverbot.com/api/)
  - [https://github.com/plasticuproject/cleverbotfree](https://github.com/plasticuproject/cleverbotfree)
- xiaobing: [https://www.msxiaobing.com/](https://www.msxiaobing.com/)
- mitsuku: [https://www.pandorabots.com/mitsuku/](https://www.pandorabots.com/mitsuku/)
  - [https://github.com/hanwenzhu/mitsuku-api](https://github.com/hanwenzhu/mitsuku-api)

### 主要贡献

- 模型架构：Evolved Transformer
  - 模型输入：多轮对话（最多7轮）
  - 模型输出：回复
  - 最佳模型：2.6B参数，10.2PPL，8K BPE subword vocabulary, 训练数据40B words
- 评测指标
  - PPL
  - SSA（Sensibleness and Specificity Average）用来评估
    - whether make sense
    - whether specific
  - 人工评测使用static（1477个多轮对话）和interactive（想说啥就说啥）两种数据集，发现SSA和PPL在这两个数据集上高度相关
  - 模型在评测集的表现：
    - 0.72的SSA
    - 经过filtering mechanism 和 tuned decoding后有0.79的SSA，相比于人提供的0.86SSA的回复已经很接近了
- 方法的局限性
  - 评测数据集的局限性，不能解决所有领域的问题

## 对话机器人的评价

### 人工进行评测时的参考标准

- Sensibleness
  - common sense
  - logical coherence
  - consistency
  - 人工评测时对于可打的标签：confusing, illogical, out of context, factually wrong, make sense
  - 缺陷：对于安全的回答，如I don’t know，无法区分
- Specificity
  - A: I love tennis.   B: That’s nice 应该被标记为not specific，如果 B：Me too, I can’t get enough of Roger Federer!则被标记为specific
  - 已经被标记为not sensible的直接标记为not specific
- SSA
  - 可以使用Sensibleness和Specificity标记在所有responses的比例来作为参考标准
  - 使用SSA将Sensibleness和Specificity的比例进行了结合

### 可进行对比的几个开源chatbot框架

- 基于RNN：[https://github.com/lukalabs/cakechat](https://github.com/lukalabs/cakechat)
- 基于Transformer: [https://github.com/microsoft/DialoGPT](https://github.com/microsoft/DialoGPT)
  - 762M参数的模型效果更好一些
  - dialogpt没有公开其解码和MMI-reranking的过程，gpt2bot实现了解码：[https://github.com/polakowo/gpt2bot](https://github.com/polakowo/gpt2bot)
  - 附加一个中文的基于DialoGPT开发的闲聊模型
    - [https://github.com/yangjianxin1/GPT2-chitchat](https://github.com/yangjianxin1/GPT2-chitchat)
    - [https://blog.csdn.net/kingsonyoung/article/details/103803067](https://blog.csdn.net/kingsonyoung/article/details/103803067)

### 构建静态评测集

- 从单轮开始：[http://ai.stanford.edu/~quocle/QAresults.pdf](http://ai.stanford.edu/~quocle/QAresults.pdf)
- 增加一些个性化问题，如：Do you like cats?
  - A: Do you like movies?; B: Yeah. I like sci-fi mostly; A: Really? Which is your favorite?期待I love Back to the Future这样的回答，对于I don’t like movies这样的回复应标记为not sensible

### 进行动态评测

- 机器人以Hi开始，评测人员自由与bot对话，并对每一个bot的回复进行评测。每一个对话至少14轮，至多28轮。

## Meena Chatbot

### 训练数据

- 来源于public social media
- 清洗流程
  - 去掉 subword 数目<=2 或 subword 数目 >= 128
  - 去掉 字母比例<0.7
  - 去掉 包含URL
  - 去掉 作者名字bot
  - 去掉 出现100次以上
  - 去掉 跟上文n-gram重复比例过高
  - 去掉 敏感句子
  - 去掉 括号中内容
  - 当一个句子被删除时，则上文全部被删除
- 共清洗出867M的(context, response)对
- 使用sentence piece进行BPE分词，得到8K的BPE vocab
- 最终语料包含341GB的语料(40B word)

### 模型框架

- Evolved Transformer
  - 2.6B parameter
  - 1 ET encoder + 13 ET decoder
- 最大的模型可达到10.2的PPL
- 最大的传统Transformer模型（32层decoder）可达到10.7的PPL
- hidden size: 2560
- attention head: 32
- 共享编码、解码、softmax的embedding
- 编码、解码最长是128

### 训练细节

- 使用Adafactor optimizer，初始学习率0.01，在前10k step保持不变，使用inverse square root of the number of steps进行衰减
- 使用[https://github.com/tensorflow/](https://github.com/tensorflow/)tensor2tensor代码进行训练

### 解码细节

- 为了避免产生乏味的回复，可以使用多种方法进行解码

  - reranking
  - 基于profiles, topics, and styles
  - 强化学习
  - 变分自编吗

- 当PPL足够小时，可以使用sample-and-rank策略进行解码

  - 使用temperature T随机产生N个独立的候选

    - $p_{i}=\frac{\exp \left(z_{i} / T\right)}{\sum_{j} \exp \left(z_{j} / T\right)}$

    - T=1产生不经过修正的分布

    - T越大，越容易产生不常见的词，如相关的实体名词，但可能产生错误的词

    - T越小，越容易产生常见的词，如冠词或介词，虽然安全但不specific

    - 解释1

      ```
      温度是神经网络的超参数，用于在应用softmax之前通过缩放对数来控制预测的随机性。 例如，在TensorFlow的LSTM中，温度代表在计算softmax之前将logit除以多少。
      
      当温度为1时，我们直接在logits（较早层的未缩放输出）上计算softmax，并使用温度为0.6的模型在logits/0.6上计算softmax，从而得出较大的值。 在更大的值上执行softmax可使LSTM 更加自信 （需要较少的输入来激活输出层），但在其样本中也更加保守 （从不太可能的候选样本中进行抽样的可能性较小）。 使用较高的温度会在各个类上产生较软的概率分布，并使RNN更容易被样本“激发”，从而导致更多的多样性和更多的错误 。
      
      softmax函数通过确保网络输出在每个时间步长都在零到一之间，基于其指数值对候选网络的每次迭代进行归一化。
      
      因此，温度增加了对低概率候选者的敏感性。
      ```

    - 解释2

      ```
      当T很大时，即趋于正无穷时，所有的激活值对应的激活概率趋近于相同（激活概率差异性较小）；而当T很低时，即趋于0时，不同的激活值对应的激活概率差异也就越大。
      ```

- 发现使用beam-search解码会产生重复且无趣的回复，使用sample-and-rank产生的回复会丰富一些
- 使用N=20，T=0.88
- response score的计算：logP/T，P是response的likelihood，T是token的个数
- 解码时增加detect cross turn repetitions
  - 当两个turn的n-gram重复超过一定比例时，则从候选中删除
- 增加一个分类层，用来过滤掉敏感回复

## 结论

### SSA和PPL是相关的

- 基本呈线性关系

![图片](<http://q503tsu73.bkt.clouddn.com/paper_ssa.png?e=1580825793&token=05Ii263bPN3Z-CT3JPRaRfWi5sXIj8pwX6V1bN2j:XYKxV0-Vy99TsjbuB7EorIolYVk=&attname=>)

### 效果的比较

- 小冰：呈现出个性化的回复，但有时也会无意义，且经常回复得太平常。小冰另一个特点就是具有同情心，可以在以后的评价指标中考虑这一点。小冰有near-human-level engagingness但not very close to human-level humanness，因此在我们的评测指标上SSA不高。
- mitsuku：56%SSA（72%sensibility 40%specifity）, 网站上的对话并不是它参加图灵测试的版本
- DialoGPT：48%SSA（57%sensibility 49%specifity）
- CleverBot：在interactive评测表现比static上稍微好一些（56% interactive SSA，44% static SSA）。发现cleverbot更擅长将话题引入到它更擅长的领域中，缺少personality
- Meena：base（72% SSA），full（79% SSA）

