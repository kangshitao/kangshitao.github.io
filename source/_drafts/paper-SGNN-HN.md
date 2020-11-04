---
title: 论文阅读-Star Graph Neural Networks for Session-based Recommendation
excerpt: 2020,CIKM,使用星型GNN的基于会话的推荐系统
categories: 论文笔记
tags: ['SBRS','paper','GNN']
keywords: 会话推荐,SGNN-HN,GNN,star graph neural networks,session
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
  <img src="https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/paper-SGNN-HN_2.png" alt="2" style="zoom:50%;" />

- **Star connections**：受[Star-Transformer](https://arxiv.org/abs/1902.09113)模型启发，在图中添加Star节点，构建星型图。star节点和satellite节点之间的边就是Star连接，如图2，Star连接是双向边，分别代表两种信息传递方向，更新两种节点。一方面，以star节点作为中间节点，非相邻的item之间能够以two-hop的方式进行信息传播，来更新satellite节点。另一方面，另一个方向的边能够用来考虑所有satellite节点的信息，生成准确的star节点表示。

不同于一般的GNN模型，本文的模型使用门控网络控制分别从邻居节点和star节点获取信息量的多少。

## 学习item embedding