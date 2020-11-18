---
title: 论文阅读-Star Graph Neural Networks for Session-based Recommendation
excerpt: '2020,CIKM,使用星型GNN的基于会话的推荐系统'
mathjax: true
categories: 论文笔记
tags:
  - SBRS
  - paper
  - GNN
  - Attention
  - 深度学习
keywords: 会话推荐,SGNN-HN,GNN,star graph neural networks,session
date: 2020-11-05 20:19:11
---



>2020年10月份[CIKM](https://www.cikm2020.org/accepted-papers/accepted-research-papers/)会议的一篇论文，主要内容是提出了带有Highway Network的Star-GNN模型，简称为SGNN-HN模型，原文[链接](https://dl.acm.org/doi/abs/10.1145/3340531.3412014)

# 摘要

现有基于GNN的模型，有两个缺陷：

1. 一般的GNN模型只考虑了相邻item的转换信息，忽略了来自不相邻item的高阶转换信息。
2. 一般的GNN模型，如果想要捕获高阶转换信息，必须要增加层数，这样就面临严重的过拟合问题

为了解决上述两个问题，作者提出了Star Graph Neural Networks with Highway Networks (SGNN-HN)模型：

1. 对于问题1，使用Star GNN(SGNN)学习当前session中不相邻item的复杂转换信息。
2. 对于问题2，使用Highway Networks(HN)自适应地从item表示中选择嵌入信息，缓解过拟合问题。

# 介绍

基于session的推荐系统是根据当前正在进行的session生成推荐，SBRS没有用户的历史交互信息，只有当前session中的item信息，并且是很有限的。使用RNN考虑session的序列信息(如GRU4Rec)，或者使用注意力机制考虑主要目的(如NARM)，都不能完全考虑到item的复杂转换信息。后来的GNN模型(如SRGNN、TAGNN等)，虽然利用GNN的优势，考虑了item间的复杂转换信息，但是面临上述两个问题，即高阶转换信息和过拟合问题。

因此，作者提出了SGNN-HN模型，**首先**在图中加入star节点，构建SGNN，建立当前session复杂item转换信息的模型，解决长距离信息传播问题。**然后**使用HN，在SGNN前后动态选择item表示，解决过拟合问题。**最后**使用注意力机制将item表示融合，得到session表示，用于推荐。

# 相关工作

- 一般推荐模型：CF、MF、NCF、Item-KNN等
- 序列推荐模型：MC、FPMC、GRU4Rec、NARM、KNN、CSRM
- 注意力模型：STAMP、Co-Attention模型
- GNN模型：SRGNN、FGNN

# 本文模型

本文模型结构图如下：

![1](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/paper-SGNN-HN_1.png)

## 问题定义

$ V=\{v_1,v_2,\dots,v_{|V|}\} $ 表示所有session中的item集合，其中$|V|$ 表示item的个数。

$S=\{v_1,v_2,\dots,v_t,\dots,v_n\}$ 是给定的当前session，包含n个item。

模型的目标是预测$v_{n+1}$ ，具体来说，对于每个session，输出$\widehat{y}=\{\widehat{y}_1,\widehat{y}_2,\dots,\widehat{y}_{|V|}\}$ ，表示所有item的预测分数，最后取分数最高的K个作为推荐item。

## 构建星型图

对于每个session $S=\{v_1,v_2,\dots,v_t,\dots,v_n\}$,构建为一个星型图，表示为$G_s=\{\mathcal{V}_s,{\Large{\varepsilon}}_s\}$。

其中$\mathcal{V_s}=\{\{x_1,x_2,\dots,x_m\},x_s\}$表示图中的所有节点。前半部分$\{x_1,x_2,\ldots,x_m\}$表示**satellite**节点，$x_s$表示**star**节点。注意这里的$m\leqslant n$，因为session中可能会有重复出现的item。

${\Large\varepsilon}_s$是图中的边集合，包括**satellite连接**(图2中实线)和**star连接**(图2中虚线)两种有向边：

- **Satellite connections**：satellite连接用来传递session中的相邻item间的信息。论文中使用GGNN为例，实现相邻节点之间的信息传播，并且按照satellite边构建输入和输出矩阵，例如对于session $S=\{x_2,x_3,x_5,x_4,x_5,x_7\}$，构建的输入输出矩阵如图3：

  <div align='center'>
      <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/paper-SGNN-HN_2.png' style='width:50%; height:50%'/>
  </div>

  
- **Star connections**：受[Star-Transformer](https://arxiv.org/abs/1902.09113)模型启发，在图中添加Star节点，构建星型图。star节点和satellite节点之间的边就是Star连接，如图2，Star连接是双向边，分别代表两种信息传递方向，更新两种节点。一方面，以star节点作为中间节点，非相邻的item之间能够以two-hop的方式进行信息传播，来更新satellite节点。另一方面，另一个方向的边能够用来考虑所有satellite节点的信息，生成准确的star节点表示。

本文模型使用门控网络控制分别从邻居节点和star节点获取信息量的多少。

## 学习item嵌入

学习satellite节点和star节点表示

### 初始化

- satellite节点：使用item的embedding

$$
h^0=\{x_1,x_2,\ldots,x_m\} \tag{1}
$$



- star节点：对satellite节点进行平均池化，得到star节点初始状态

$$
x_s^0 = \frac 1{m}\sum\limits^{m}_{i=1} {x_i} \tag{2}
$$

### 更新节点

1. **Satellite节点更新**

   对于每个satellite节点，邻居节点信息包括**相邻节点信息**和**star节点信息**，分别对应来自直接相连节点和没有直接相连节点的信息。

   - 首先，考虑相邻节点信息，使用GGNN网络更新节点。对于图的第$l$层每个satellite节点$x_i$，使用出度和入度矩阵获得传播的信息：
     $$
     \begin{align}
     a_i^l = Concat(\mathbf{A}_i^I([\mathbf{x}_1^{l-1},\mathbf{x}_2^{l-1},\ldots,\mathbf{x}_m^{l-1}]^{\mathbf{T}}\mathbf{W}^I+\mathbf{b}^I),\\
     \mathbf{A}_i^O([\mathbf{x}_1^{l-1},\mathbf{x}_2^{l-1},\ldots,\mathbf{x}_m^{l-1}]^{\mathbf{T}}\mathbf{W}^O+\mathbf{b}^O)) \tag{3}
     \end{align}
     $$
     其中，$\mathbf{A}_i^I,\mathbf{A}_i^O \in \mathbb{R}^{1\times m}$，分别为入度和出度矩阵的第$i$行，$\mathbf{W}^I,\mathbf{W}^O \in \mathbb{R}^{d \times d}$是参数矩阵，$\mathbf{b}^I,\mathbf{b}^O \in \mathbb{R}^d$是偏置向量。最终得到的$a_i^l \in \mathbb{R}^{1\times 2d}$表示节点$x_i$的传播信息，然后继续使用GGNN网络计算：
     $$
     \begin{align}
     \mathbf{z}_i^l=\sigma(\mathbf{W}_z\mathbf{a}_i^l+\mathbf{U}_z\mathbf{h}_i^{l-1}),\\
     \mathbf{r}_i^l=\sigma(\mathbf{W}_r\mathbf{a}_i^l+\mathbf{U}_r\mathbf{h}_i^{l-1}),\\
     \mathbf{\tilde{h}}_i^l=\tanh(\mathbf{W}_h\mathbf{a}_i^l+\mathbf{U}_h(\mathbf{r}_i^l\odot \mathbf{h}_i^{l-1}),\\
     \mathbf{\hat{h}}_i^l=(1-\mathbf{z}_i^l)\odot \mathbf{h}_i^{l-1} + \mathbf{z}_i^l \odot \mathbf{\tilde{h}}_l \tag{4}
     \end{align}
     $$
     其中，$\mathbf{W}_z,\mathbf{W}_r,\mathbf{W}_h \in \mathbb{R}^{d\times 2d}$，$\mathbf{U}_z,\mathbf{U}_r,\mathbf{U}_h \in \mathbb{R}^{d\times d}$是参数矩阵，$\sigma$表示*sigmoid*激活函数，$\odot$表示向量的对应元素点乘。使用这种方式，相邻节点的信息能够在星型图中传播。

   - 然后，考虑来自star节点的信息，对于每个satellite节点$x_i$，使用门控机制考虑应该从相邻节点和star节点分别获取多少信息。具体来说，使用自注意力计算$x_i$和star节点$x_s$的相似度$\alpha_i^l$:
     $$
     \alpha_i^l=\frac{(\mathbf{W}_{q1} \mathbf{\hat{h}}_i^l)^\mathbf{T} \mathbf{W}_{k1} \mathbf{x}_s^{l-1}}
     {\sqrt{d}} \tag{5}
     $$
     其中，$\mathbf{W}_{q1},\mathbf{W}_{k1} \in \mathbb{R}^{d\times d}$是参数矩阵，$\mathbf{\hat{h}}_i^l$是$x_i$的表示向量。然后计算$x_i$最终的节点表示：
     $$
     \mathbf{h}_i^l = (1-\alpha_i^l) \mathbf{\hat{h}}_i^l + \alpha_i^l \mathbf{x}_s^{l-1} \tag{6}
     $$

     > 注意，这里的star节点是上一层的，因为这一层的star节点还没有更新，star节点需要在satellite节点更新完之后再更新。

2. **Star节点更新**

   star节点是根据最新的satellite节点表示来更新的，同样使用self-attention，将star节点作为Query向量：
   $$
   \beta = softmax(\frac{\mathbf{K}^ \mathbf{T} \mathbf{q}}{\sqrt{d}})
   = softmax(\frac{(\mathbf{W}_{k2} \mathbf{h}^l)^ \mathbf{T} \mathbf{W}_{q2} \mathbf{x}_s^{l-1}}{\sqrt{d}}) \tag{7}\\
   $$
   $\mathbf{W}_{k2},\mathbf{W}_{q2} \in \mathbb{R}^{d\times d}$是对应的参数矩阵。最后计算得到star节点的向量表示
   $$
   \mathbf{x}_s^l = \beta \mathbf{h}^l \tag{8}
   $$
   其中$\beta \in \mathbb{R}^m$，$\mathbf{h}^l \in \mathbb{R}^{d\times m}$

### Highway Networks

上述节点更新的过程可以多次迭代，即堆叠多层SGNN，第$l$层SGNN可以表示为：
$$
\mathbf{h}^l,\mathbf{x}_x^l = SGNN(\mathbf{h}^{l-1},\mathbf{x}_s^{l-1},\mathbf{A}^I,\mathbf{A}^O) \tag{9}
$$
多层图结构可以传播大量节点间的信息，同样也会带来偏差，导致过拟合，为了处理这个问题，使用highway networks从多层SGNN之前和之后选择性地获取信息。

用$\mathbf{h}^0,\mathbf{h}^L$分别表示多层SGNN之前和多层SGNN之后的satellite节点表示，HN网络用以下公式表示：
$$
\mathbf{h}^f = \mathbf{g} \odot \mathbf{h}^0 +(1-\mathbf{g})\odot \mathbf{h}^L \tag{10}
$$
其中$\mathbf{g} \in \mathbb{R}^{d\times m}$，是由SGNN的输入输出决定的：
$$
\mathbf{g} = \sigma(\mathbf{W}_g[\mathbf{h}^0;\mathbf{h}^L]) \tag{11}
$$
$\mathbf{W}_g \in \mathbb{R}^{d\times2d}$。

HN网络处理之后，得到satellite节点和star节点的最终表示$\mathbf{h}^f$和$\mathbf{x}_s^L$(简写为$\mathbf{x}_s$)​。将以上步骤用算法流程表示：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/paper-SGNN-HN_3.jpg'/>
</div>

##  Session表示和预测

将$\mathbf{h}^f \in \mathbb{R}^{d\times m}$用$\mathbf{u}\in \mathbb{R}^{d\times n}$表示(?没看懂)，然后加入可学习的位置编码$\mathbf{p}\in \mathbb{R}^{d\times n}$，则satellite节点表示变为$\mathbf{u}^p = \mathbf{u}+\mathbf{p}$。

考虑使用用户的全局偏好和最近兴趣来生成session表示，使用最后一个点击item作为最近兴趣，即$\mathbf{z}_r=\mathbf{u}_n^p$，然后计算全局偏好：
$$
\mathbf{z}_g = \sum _{i=1}^n \gamma_i \mathbf{u}_i^p \tag{12}
$$

$$
\gamma_i = \mathbf{W}_0^ \mathbf{T} \sigma(\mathbf{W}_1 \mathbf{u}_i^p+
\mathbf{W}_2 \mathbf{x}_s + \mathbf{W}_3 \mathbf{z}_r + \mathbf{b}) \tag{13}
$$

$\mathbf{W}_0,\mathbf{b} \in \mathbb{R}^d$，$\mathbf{W}_1,\mathbf{W}_2,\mathbf{W}_3 \in \mathbb{R}^{d\times d}$。

然后计算最终的session表示：
$$
\mathbf{z}_h = \mathbf{W}_4[\mathbf{z}_g;\mathbf{z}_r] \tag{14}
$$
考虑到长尾效应，参考[NISER](https://arxiv.org/abs/1909.04276)，对session表示$\mathbf{z}_h$和候选item $\mathbf{v}_i$的嵌入向量进行层标准化处理，即:
$$
\tilde{\mathbf{z}}_h = LayerNorm(\mathbf{z}_h) = \frac{\mathbf{z}_h}{\Vert \mathbf{z}_h \Vert _2} \\
\tilde{\mathbf{v}}_i = LayerNorm(\mathbf{v}_i)= \frac{\mathbf{v}_i}{\Vert \mathbf{v}_i \Vert _2}
$$
然后计算推荐分数：
$$
\tilde{\mathbf{y}_i} = \tilde{\mathbf{z}}_h^{\mathbf{T}}\tilde{\mathbf{v}_i} \tag{15}
$$
最后使用softmax函数进行归一化，根据[论文](https://dl.acm.org/doi/10.1145/3123266.3123359)，由于余弦相似度的值在$[-1,1]$之间，可能导致softmax损失很大，以致模型无法收敛，因此使用缩放因子用来帮助模型更好地收敛：
$$
\hat{\mathbf{y}} = \frac{\exp(\tau \tilde{\mathbf{y}_i})}
{\sum \nolimits_i \tau \hat{\mathbf{y}_i}}, \forall i=1,2,\ldots,|V| \tag{16}
$$
其中$\hat{\mathbf{y}}=(\hat{\mathbf{y}}_1,\hat{\mathbf{y}}_2,\ldots,\hat{\mathbf{y}}_{|V|})$。损失函数使用交叉熵：

$$
L(\hat{\mathbf{y}})=-\sum\limits_{i=1}^{|V|} \mathbf{y}_i \log(\hat{\mathbf{y}}_i)+
(1-\mathbf{y}_i)\log(1-\hat{\mathbf{y}}_i) \tag{17}
$$
$\mathbf{y}_i \in \mathbf{y}$，$\mathbf{y}$是one-hot向量，$\mathbf{y}_i=1$表示第$i$个item是给定session的目标item。

# 实验

实验数据和处理方式同SRGNN、NARM等模型，并且同样使用了数据增强的方法，具体细节见原文。

实验数据：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/paper-SGNN-HN_4.png'/>
</div>

评估指标使用Precision和MRR。

# 实验结果

## 与SOTA模型对比

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/paper-SGNN-HN_5.png'/>
</div>

## SGNN网络的作用

分别用自注意力网络(SAT)和门控图神经网络(GGNN)代替SGNN：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/paper-SGNN-HN_6.png'/>
</div>

## HN网络层的作用

HN网络对于不同数量GNN层的作用：

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/paper-SGNN-HN_7.png'/>
</div>
## Session长度的影响

<div align='center'>
    <img src='https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/paper-SGNN-HN_8.png'/>
</div>

# 结论

本文提出了SGNN-HN模型，一方面解决了一般GNN模型不能考虑高阶信息转换的问题，另一方面解决了GNN模型容易过拟合的问题。