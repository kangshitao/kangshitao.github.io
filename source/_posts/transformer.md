---
title: 自注意力和Transformer
excerpt: 自注意力机制和Transformer详解
mathjax: true
categories:
  - 深度学习
tags:
  - 深度学习
  - Attention
  - transformer
keywords:
  - 自注意力
  - self attention
  - transformer
date: 2020-11-15 11:31:47
---



>参考链接：
>
>[1] 邱锡鹏：[神经网络与深度学习](https://nndl.github.io/)
>
>[2] Jay Alammar：[Illustrated Transformer](http://jalammar.github.io/illustrated-transformer/)
>
>[3] [深度学习-图解Transformer(变形金刚)](https://zhuanlan.zhihu.com/p/105493618)
>
>[4] [详解Transformer](https://zhuanlan.zhihu.com/p/48508221)

# 自注意力

在讲述Transformer之前，首先介绍Self-Attention模型。

传统的RNN虽然理论上可以建立输入信息的长距离依赖关系，但是由于信息传递的容量和梯度消失的问题，实际上只能建立短距离（局部）依赖关系。为了建立输入序列之间的长距离依赖关系，并利用注意力机制来“动态”地生成不同连接的权重，诞生了**自注意力模型(Self-Attention Model)**

自注意力模型采用查询-键-值(Query-Key-Value,QKV)模式，计算过程如下图，红色字母表示维度：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/transformer_1.png'
         </img>
</div>

计算过程：

- 输入序列$\boldsymbol{X}=[\boldsymbol{x}_1,\ldots,\boldsymbol{x}_N]\in \mathbb{R}^{D_x \times N}$

- 生成三个向量序列：
  $$
  \begin{align}
  \boldsymbol{Q}=\boldsymbol{W}_q \boldsymbol{X} \in \mathbb{R}^{D_k \times N}  \tag{1}\\
  \boldsymbol{K}=\boldsymbol{W}_k \boldsymbol{X} \in \mathbb{R}^{D_k \times N}  \tag{2}\\
  \boldsymbol{V}=\boldsymbol{W}_v \boldsymbol{X} \in \mathbb{R}^{D_v \times N}  \tag{3}
  \end{align}
  $$
  其中，$\boldsymbol{W}_q \in \mathbb{R}^{D_k \times D_x}$，$\boldsymbol{W}_k \in \mathbb{R}^{D_k \times D_x}$，$\boldsymbol{W}_v \in \mathbb{R}^{D_v \times D_x}$分别为线性映射的参数矩阵，$\boldsymbol{Q}=[\boldsymbol{q}_1,\ldots,\boldsymbol{q}_N]$，$\boldsymbol{K}=[\boldsymbol{k}_1,\ldots,\boldsymbol{k}_N]$，$\boldsymbol{V}=[\boldsymbol{v}_1,\ldots,\boldsymbol{v}_N]$分别是由**查询向量**、**键向量**和**值向量**构成的矩阵。

  >注意：Self-Attention中通常使用点积来计算注意力打分，这里的查询向量Q和键向量K的维度是相同的。

  

- 最终的输出序列是$\boldsymbol{H}=[\boldsymbol{h}_1,\ldots,\boldsymbol{h}_N]\in \mathbb{R}^{D_v \times N}$，其中的输出向量$\boldsymbol{h}_n$使用键值对计算方法计算：
  $$
  \begin{align}
  \boldsymbol{h}_n &= \mbox{att}((\boldsymbol{K},\boldsymbol{V}),\boldsymbol{q}_n)\\
  &=\sum_{j=1}^N \alpha_{nj} \boldsymbol{v}_j \\
  &=\sum_{j=1}^N \mbox{softmax}(s(\boldsymbol{k}_j,\boldsymbol{q}_n)) \boldsymbol{v}_j \tag{4}
  \end{align}
  $$
  其中$n,j\in[1,N]$为输出和输入向量序列的位置，$\alpha_{nj}$表示第$n$个输出关注到第$j$个输入的权重。

- 如果使用**缩放点积**来作为打分函数的话，输出向量序列可以简写为：
  $$
  \boldsymbol{H}=\mbox{softmax}(\frac{\boldsymbol{Q} \boldsymbol{K}^T}{\sqrt{D_k}}) \boldsymbol{V} \tag{5}
  $$

>公式(5)的写法中，softmax函数**按行**进行归一化，即每一行进行归一化，归一化方法根据公式的写法决定。

自注意力模型可以作为神经网络的一层来使用，既可以用来替换卷积层和循环层，也可以和它们一起交替使用。

由于自注意力模型的权重$\alpha_{ij}$只依赖于$q_i$和$k_j$的相关性，忽略了输入信息的位置信息，因此单独使用时，一般需要加入位置编码信息进行修正，Transformer中就是这么做的。

# 多头自注意力

**多头自注意力(Multi-Head Self-Attention)**在多个不同的投影空间中捕捉不同的交互信息。假设在M个投影空间中分别应用自注意力模型，则：
$$
\begin{align}
\mbox{MultiHead}(\boldsymbol{H}) &= \boldsymbol{W}_o[\mbox{head}_1;\ldots;\mbox{head}_M]  \tag{6}\\
\mbox{head}_m &= \mbox{self-att}(\boldsymbol{Q}_m,\boldsymbol{K}_m,\boldsymbol{V}_m) \tag{7}\\
\forall m\in {1,\ldots,M},
\boldsymbol{Q}_m &=\boldsymbol{W}_q^m \boldsymbol{H},
\boldsymbol{Q}_m =\boldsymbol{W}_k^m \boldsymbol{H},
\boldsymbol{Q}_m =\boldsymbol{W}_v^m \boldsymbol{H} \tag{8}
\end{align}
$$
其中，$\boldsymbol{W}_o \in \mathbb{R}^{D_v \times Md_v}$，$\boldsymbol{W}_q^m \in \mathbb{R}^{D_k \times D_h}$，$\boldsymbol{W}_k^m \in \mathbb{R}^{D_k \times D_h}$，$\boldsymbol{W}_v^m \in \mathbb{R}^{D_v \times D_h}$为投影矩阵，$m\in\{1,\ldots,M\}$.

也就是说，多头自注意力是将多个不同参数的自注意力的结果进行拼接，如图：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/transformer_2.png'>
    </img>
</div>

# Transformer

Transformer是2017年Google团队提出的一个基于Mult-Head Attention的Seq2Seq模型，整个网络结构由**编码器**和**解码器**两部分组成，原文链接[Attention Is All you Need](https://arxiv.org/abs/1706.03762).

Transformer的主要结构如下：

<div align='center'>
    <img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/transformer_3.png"
         </img>
</div>

上图是来自原文的结构图，左边是N个编码器，右边是N个解码器，Transformer中的N为6，下图是Jay Alammar博客中的简化图：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/transformer_4.png'>
    </img>
</div>

## 编码器

Transformer中的编码器部分一共有6个相同的编码器层组成，下面以单个层为例。如下图，每个编码器都有两个子层，即**多头自注意力层**(Multi-Head Attention)层和**逐位置的前馈神经网络**(Position-wise Feed-Forward Network)。在每个子层后面都有**残差连接**（图中的虚线）和**层归一化**（LayerNorm）操作，二者合起来称为**Add&Norm**操作。

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/transformer_5.png'>
</div>

编码器的输入为序列$\boldsymbol{x}_{1:T}$，输出为$\boldsymbol{H}^{enc}=[\boldsymbol{h}_1^{enc},\ldots,\boldsymbol{h}_T^{env}]$，然后用两个矩阵将$\boldsymbol{H}^{enc}$映射到$\boldsymbol{K}^{enc}$和$\boldsymbol{V}^{enc}$作为键值对供解码器使用：
$$
\begin{align}
\boldsymbol{K}^{enc}=\boldsymbol{W}'_k \boldsymbol{H}^{enc} \tag{9}\\
\boldsymbol{V}^{enc}=\boldsymbol{W}'_v \boldsymbol{H}^{enc} \tag{10}
\end{align}
$$
编码器中的关键步骤：

1. **位置编码**

   由于自注意力模型忽略了序列$\boldsymbol{x}_{1:T}$中每个$\boldsymbol{x}_t$的位置信息（即使句子结构混乱也能得到类似的结果），因此需要在初始的输入序列中加入位置编码来进行修正，对于输入序列$\boldsymbol{x}_{1:T}\in \mathbb{R}^{D\times T}$，令：
   $$
   \boldsymbol{H}^{(0)} = [\boldsymbol{e}_{x_1}+\boldsymbol{p}_1,\ldots,\boldsymbol{e}_{x_T}+\boldsymbol{p}_T] \tag{11}
   $$
   其中$\boldsymbol{e}_{x_t} \in \mathbb{R}^D$是词$x_t$的嵌入向量表示，$\boldsymbol{p}_t \in \mathbb{R}^D$是位置 $t$ 的向量表示，即**位置编码**。位置编码通过下面的公式预定义：
   $$
   \begin{align}
   \boldsymbol{p}_{t,2i} &=\sin (\frac{t}{10000^{\frac{2i}{D}}}) \\
   \boldsymbol{p}_{t,2i+1} &=\cos (\frac{t}{10000^{\frac{2i}{D}}}) \tag{12}
   \end{align}
   $$
   其中$\boldsymbol{p}_{t,2i}$表示第$t$个位置的编码向量的第$2i$维，$D$是编码向量的维度。

    >Transformer里的位置编码是没有梯度的，不参与训练，每个位置上的值是固定的。

   

2. **层归一化和残差连接(Add&Norm)**

   每个子层后都跟着Add&Norm层，包括**残差连接**和**层归一化**操作：
   $$
   \mbox{LayerNorm}(\boldsymbol{X}+\mbox{Sublayer}(\boldsymbol{X}))  \tag{13}
   $$
   其中$\boldsymbol{X}$是来自上个子层的输出，$\mbox{LayerNorm}$表示层归一化操作，$\mbox{Sublayer}$表示上个子层的函数，即$\mbox{Multi-Head}$或者$\mbox{FNN}$.

   > Transoformer中的子层和Add&Norm层之间有dropout操作。

3. **逐位置的前馈神经网络(Position-wise-Feed-Forward Network)**

   逐位置的前馈神经网络是一个简单的两层网络，对于序列的每个位置上的向量$\boldsymbol{x}\in\boldsymbol{X}^{(l)}$执行以下操作：
   $$
   \mbox{FFN}(\boldsymbol{x})=\boldsymbol{W}_2 \mbox{ReLu}(\boldsymbol{W}_1\boldsymbol{x}+\boldsymbol{b}_1)+\boldsymbol{b}_2  \tag{14}
   $$
   

## 解码器

解码器部分同样由6个相同的层组成，每层都采用自回归的方式生成目标序列，接收来自编码器的输出$\boldsymbol{K}^{enc}$和$\boldsymbol{V}^{enc}$，以及上一层解码器的输出（用来生成查询向量$\boldsymbol{Q}$）。同样以单层为例，每层包括三个子层：**掩蔽自注意力层**(Masked Self-Attention)、**Encoder-Decoder注意力层**、**逐位置的前馈神经网络**，同样，每个子层后面都有**Add&Norm**操作，具体结构如图：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/transformer_6.png'>
	</img>
</div>

解码器中的关键步骤：

1. **掩蔽自注意力** (Masked Self-Attention)

   第$t$步时，使用掩蔽自注意力模块对已经生成的前缀序列$\boldsymbol{y}_{0:(t-1)}$进行编码，得到$\boldsymbol{H}^{enc}=[\boldsymbol{h}_1^{enc},\ldots,\boldsymbol{h}_t^{env}]$，使$t$之后的序列不可见。

2. **Encoder-Decoder注意力**

   将$\boldsymbol{h}_t^{dec}$进行线性映射得到$\boldsymbol{q}_t^{dec}$，将$\boldsymbol{q}_t^{dec}$作为查询向量，通过键值对注意力机制，来从输入($\boldsymbol{K}^{enc}$，$\boldsymbol{V}^{enc}$)中选取有用的信息。

3. **逐位置的前馈神经网络**

   同编码器的$\mbox{FNN}$网络，用来综合得到所有的信息。

与编码器一致，解码器也包括层归一化、残差连接的操作，其计算方式也相同，这里不再具体介绍。

解码器的解码过程可以参考下图：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/transformer_7.gif
'>
</img>
</div>


然后重复以上步骤，直到句子结束：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/transformer_8.gif
'></img>
</div>


解码器初始的输入要有**起始符**来激活解码器，这里用`<s>`表示(上图中没体现)，起始符是通过**右移**(shifted right)操作事先给定好的。结束符用p表示。将图中的例子用语言描述：

```
Step 1
起始输入：起始符<s>+p
中间输入：K、V
最终输出：I

Step 2
起始输入：起始符<s>+'I'+p
中间输入：K、V
最终输出：I am

Step 3
起始输入：起始符<s>+'I'+'am'+p
中间输入：K、V
最终输出：I am a

Step 4
起始输入：起始符<s>+'I'+'am'+'a'+p
中间输入：K、V
最终输出：I am a student

Step 5
起始输入：起始符<s>+'I'+'am'+'a'+'student'+p
中间输入：K、V
最终输出：I am a student <end of sentence>
```

---

补充说明：


1. 关于**掩蔽自注意力**：

   在训练过程中，为了能够一次性完成以上步骤以提高效率，将**右移的目标序列**(Right-Shifted Output) $\boldsymbol{y}_{0:(T-1)}$作为解码器输入， 即在第$t$个位置的输入为$y_{t-1}$，通过**掩码(Mask)**来阻止每个位置选择后面的输入信息，只能看到$t$之前的解码结果，这种方法就称为**掩蔽自注意力**(Masked Self-Attention)。例如，一个3*3的矩阵，矩阵的每一行对应于解码的每一步：
   $$
   \mbox{Mask}:
   \left[
   \begin{array}
   {cccc}
   1&0&0&0\\
   1&1&0&0\\
   1&1&1&0\\
   1&1&1&1\\
   \end{array}
   \right]
   $$
   而在测试/预测过程中，解码过程是一步一步计算的。每一步中解码器输出的矩阵取最后一行用来预测下一个词。

2. 关于**右移(Shifted Right)**操作

   右移操作是为了在编码器的初始输入中添加起始符`<s>`,例如，正常的输出序列位置关系：

   ```
   0:'I'
   1:'am'
   2:'a'
   3:'student'
   ```

   通过右移操作后的输入序列：

   ```
   0:<s>
   1:'I'
   2:'am'
   3:'a'
   4:'student'
   ```

## 总结

参考链接[[4]](https://zhuanlan.zhihu.com/p/48508221)的内容，说明了Transformer的优缺点：

优点：

- Transformer设计足够有创新，其抛弃了在NLP中最根本的RNN或者CNN并且取得了非常不错的效果。
- Transformer的设计最大的带来性能提升的关键是将任意两个单词的距离是1，这对解决NLP中棘手的长期依赖问题非常有效。
- Transformer不仅可以应用在NLP的机器翻译领域，还可以应用在其他非NLP领域。
- Transformer算法并行性很好，符合目前硬件(主要指GPU)环境。

缺点：

- 粗暴的抛弃RNN和CNN使模型丧失了捕捉局部特征的能力，RNN + CNN + Transformer的结合可能会带来更好的效果。
- Transformer失去的位置信息在NLP中非常重要，而论文中在特征向量中加入Position Embedding只是一个权宜之计，并没有改变Transformer结构上的固有缺陷。



