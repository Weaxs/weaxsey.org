---
title: "混合专家模型 (MoE) 笔记"
showSummary: true
summary: "梳理 MoE 模型概念和架构，简述了 GShard、Switch Transformer、DeepSeek-MoE、LLaMA-MoE 模型中的特点。"
date: 2024-03-06
tags: ["算法原理"]
---

{{< katex >}}

## 引言

本文主要想梳理一下 MoE 模型相关的概念，并阅读整理部分开源 MoE 模型的论文，简要地描述整体架构等。

## 概念

关于MoE 模型详解的部分主要参考了这篇文章 <i><u>[混合专家模型 (MoE) 详解](https://huggingface.co/blog/zh/moe)</i></u>。

### Transformer 和 MoE

先回顾一下 ***Transformer*** 架构

- ***Transformer*** 完整结构由若干个 ***Block*** 组成
- 每个 ***Block*** 中包含了编码器 *Encoder* 和解码器 *Decoder*
- *Encoder* 和 *Decoder* 都包含4个部分：注意力层 *Attention*、位置感知前馈层 *FNN*、残差连接 *Add* 和层归一化 *Norm*

![Transformer 架构.png](img%2FTransformer%20%E6%9E%B6%E6%9E%84.png)

混合专家模型 (**MoE**) 是一种基于 **Transformer** 架构的模型，主要是由两个关键部分组成：

- 稀疏 MoE 层：代替了 **Transformer** 架构中的前馈层 *FNN*。**MoE** 层包含了多个专家，每个专家可以是一个独立的神经网络，通常是前馈网络 *FNN*，但也可以是更复杂的网络结构，甚至可以是 MoE 层本身。
- 门控网络或路由：这部分决定 token 被发送到哪个专家。一个令牌可以被发送到多个专家，路由器由学习的参数组成，并与网络的其他部分一起进行预训练。

![Switch Transfomer.png](img%2FSwitch%20Transfomer.png)

### 稀疏性

在传统的 Transformer 稠密模型中，所有参数都会对所有输入数据进行处理；相比而言，MoE 稀疏模型支持仅对整个系统的某些特定部分执行计算，即可以根据输入的特定特征或需求，只运行和调用部分参数，这被称为**条件计算**。

22年的一篇论文，指出了 Transformer 模型中 FFNs 也存在稀疏性激活问题。换句话说，对于单个输出，只有 FFNs 中一小部分神经元被激活(<i><u>引用 [MoEfication: Transformer Feed-forward Layers are Mixtures of Experts](https://arxiv.org/abs/2110.01786)”</i></u>)。为了验证这个结论，作者将 Transformer 中的  FNNs 层分割成多个专家，称作 **MoEfication**，比较重要的主要是两点：

- 专家分割：将 FFNs 分割成多个功能区作为专家，这里主要提供了两种方法
    - 参数聚类分割：使用平衡 K-Means (Balanced K-Means) 算法对神经元向量做聚类
    - 共激活图分割：构建共激活图 (Co-activation Graph)
- 专家选择：这部分主要决定如何选择专家，这里没有使用典型的 MoE 门控网络，只是浅谈一下
    - Groundtruth Selection：用贪婪算法计算每个专家的得分，然后选得分最高的专家
    - Similarity Selection：用余弦距离做近似计算，选出最近似的
    - MLP Selection (推荐)：训练多层感知器 (MLP)，预测每个专家中激活神经元的总和并将其作为得分

![MoEfication.png](img%2FMoEfication.png)

### 门控网络

但这里还需要注意一点的是，需要对专家进行负载均衡，以防止不均匀分配和资源利用效率不高的问题。这可以通过一个可学习的门控网络 (G) 来决定要发送给哪些专家 (E)：

$$
y=\sum^n_{i=1}G(x)_iE_i(x)
$$

典型的可学习的门控网络 (G) 通常是带有 \\(softmax\\) 函数的网络（返回每个输出分类的结果和概率），但默认的 \\(softmax\\) 函数是返回所有结果，即返回所有专家和概率。在这种情况下可以再引入一些可调整的噪声，然后保留k个值，公式如下。 (<i><u>引用自 [Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer](https://arxiv.org/abs/1701.06538)</i></u>)

$$
H(x)\_i =(xW_g)\_i + StandardNormal()\*SoftPlus((xW_{noise})\_i)
$$

$$
G(x)=Softmax(KeepTopK(H(x), k))
$$

最后还需要注意的是，在训练 MoE 模型的过程中，因为受欢迎的专家训练的更快，所以门控网络往往更倾向于主要激活相同的几个专家，使得这种情况自我加强。为了缓解这个问题，就需要引入一个**辅助损失**，从而确保所有专家接收大致相等数量的训练样本。在 `transformers` 库中，可以通过 `aux_loss` 参数来控制辅助损失。

{{< alert "circle-info">}}
稀疏混合专家模型 (MoE) 更适用于有多台机器且要求高吞吐量的场景；相反，显存较少且吞吐量低的场景更适合用稠密Transfomer模型
{{< /alert>}}

## GShard：Top-2 门控

谷歌使用 MoE 架构将模型的参数量扩展到了 6000亿。 (<i><u>引用 [GShard: Scaling Giant Models with Conditional Computation and Automatic Sharding](https://arxiv.org/abs/2006.16668)</i></u>)

GShard 将编码器和解码器中的每个前馈网络层(FNN)替换为一个 Top-2 门控网络的 MoE 层。简而言之，token 进入 MoE 层之后，通过门控网络选出 Top-2 的专家进行处理，每个专家都是 Transformer 架构的稠密模型。

![GShard.png](img%2FGShard.png)

GShard 还引入了一下关键变化：

- 随机路由：Top-2选择中，第一个专家是排名最高的，第二个专家是根据其权重比例随机选择的
- 专家容量：设定阈值来定义专家能处理多少个 token。如果两个专家的容量都达到上限，令牌就会溢出，并通过残差链接传递到下一层，或在某些情况下被完全丢弃。所以 MoE 模型的专家容量决定了能处理的token长度。

## **Switch Transformers：单专家和专家容量**

HuggingFace 上开源的 Switch Transformer 是模型是一个基于 [google/flan-t5-large](https://huggingface.co/google/flan-t5-large) 模型改造的 MoE ，拥有2048个专家、1.6万亿参数的 MoE，专家容量64。Switch Transformer 中的每个专家是一个标准的 FFN，所以总的参数量是标准 Transformer 模型的 2048 倍。

 (<i><u>引用自 [Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity](https://arxiv.org/abs/2101.03961), 开源于 [google/switch-c-2048](https://huggingface.co/google/switch-c-2048)</u></i>)

![Switch Transfomer.png](img%2FSwitch%20Transfomer.png)

与 GShard 想法不同，Switch Transformers 对经典的 topK 门控进行了简化，采用了单专家策略，即一个token令牌只由一个专家执行，这种方法可以 ① 减少门控网络的计算负担 ② 每个专家的批量大小至少可以减半 ③ 降低通信成本 ④ 保持模型质量。

Switch Transformers也对专家容量进行了研究，建议容量将批次中的令牌数量均匀分配到各个专家，具体公式如下（注：Switch Transformers 在低容量因子 (例如 1 至 1.25) 下表现出色）

$$
Expert\space{} Capacity = (\frac{tokens \space{}per\space{}batch}{number \space{}of\space{}experts})\times capacity\space{}factor
$$

以开源模型为例，专家容量是64，专家数量是2048，假设容量因子是1，那么每个批次中的令牌数量是 131072。

## DeepSeek-MoE：共享专家

这部分主要看一下国产开源的 MoE 模型，在 huggingface 上开源的模型包含164亿参数，共 64个路由专家、2个共享专家，每个令牌会匹配6个专家。(<i><u>引用 [DeepSeekMoE: Towards Ultimate Expert Specialization in Mixture-of-Experts Language Models](https://arxiv.org/abs/2401.06066), 模型 [deepseek-ai/deepseek-moe-16b-base](https://huggingface.co/deepseek-ai/deepseek-moe-16b-base), 微调代码 https://github.com/deepseek-ai/DeepSeek-MoE</i></u>)

![DeepSeek-MoE.png](img%2FDeepSeek-MoE.png)

DeepSeek-MoE 引入了“**细粒度/垂类专家**”和“**共享专家**”的概念。

“**细粒度/垂类专家**” 是通过细粒度专家切分 (Fine-Grained Expert Segmentation) 将一个 FFN 切分成 \\(m\\) 份。尽管每个专家的参数量小了，但是能够提高专家的专业水平。

“**共享专家**”是掌握更加泛化或公共知识的专家，从而减少每个细粒度专家中的知识冗余，共享专家的数量是固定的且总是处于被激活的状态。

## LLaMA-MoE：轻量化和可配化

LLaMA-MoE-v1 是基于 LLaMA2 的一个 MoE 模型。类似 **MoEfication**，LLaMA-MoE-v1 将 LLaMA2 模型中的前馈网络层 FFNs 分割为包含多个专家的 MoE，这导致LLaMA-MoE-v1中一个专家的参数比其他 MoE 模型要更小些。 (<i><u> 引用 [LLaMA-MoE: Building Mixture-of-Experts from LLaMA with Continual Pre-training](https://github.com/pjlab-sys4nlp/llama-moe/blob/main/docs/LLaMA_MoE.pdf), 模型 [llama-moe](https://huggingface.co/llama-moe), 代码库 https://github.com/pjlab-sys4nlp/llama-moe </i></u>)

对于分割方法，这里提供了随机(Random)、聚类(Clustering)、共激活图(Co-activation Graph)和梯度(Gradient)等等。

对于专家路由即门控网络部分， LLaMA-MoE-v1 实现了经典的 TopK 噪声门控，也实现了 Switch Transformer 中提出的单专家门控。

![LLaMA-MoE.png](img%2FLLaMA-MoE.png)

## 参考

[混合专家模型 (MoE) 详解](https://huggingface.co/blog/zh/moe)

[Learning Factored Representations in a Deep Mixture of Experts](https://arxiv.org/abs/1312.4314)

[Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer](https://arxiv.org/abs/1701.06538)

[GShard: Scaling Giant Models with Conditional Computation and Automatic Sharding](https://arxiv.org/abs/2006.16668)

[Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity](https://arxiv.org/abs/2101.03961)

[DeepSeekMoE: Towards Ultimate Expert Specialization in Mixture-of-Experts Language Models](https://arxiv.org/abs/2401.06066)