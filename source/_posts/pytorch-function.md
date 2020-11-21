---
title: PyTorch各种维度变换函数总结
excerpt: >-
  记录reshape/view、resize_、transpose/permute、squeeze/unsqueeze、expand/repeat函数使用方法和区别
mathjax: true
categories: PyTorch
tags:
  - PyTorch
  - Python
keywords:
  - pytorch
  - reshape'
  - view
  - transpose
  - permute
  - unsqueeze
  - 维度变换
date: 2020-11-21 22:33:39
---





# 介绍

本文对于PyTorch中的各种维度变换的函数进行总结，包括`reshape()`、`view()`、`resize_()`、`transpose()`、`permute()`、`squeeze()`、`unsqeeze()`、`expand()`、`repeat()`函数的介绍和对比。



# contiguous

区分各个维度转换函数的前提是需要了解**contiguous**。在PyTorch中，contiguous指的是**Tensor底层一维数组的存储顺序和其元素顺序一致**。

Tensor是以一维数组的形式存储的，C/C++使用**行优先**(按行展开)的方式，Python中的Tensor底层实现使用的是C，因此PyThon中的Tensor也是按行展开存储的，如果其**存储顺序**和**按行优先展开的一维数组元素顺序**一致，就说这个Tensor是连续(contiguous)的。

**形式化定义：**

对于任意的$d$维张量$t$，如果满足对于所有的$i$，第$i$维相邻元素间隔=第$i+1$维相邻元素间隔$\times$第$i+1$维长度的乘积，则$t$是连续的：
$$
stride[i]=stride[i+1]\times size[i+1],\forall i=0,\ldots,d-1(i\neq d-1)
$$

- $stride[i]$表示第$i$维相邻元素之间间隔的位数，称为步长，可通过`stride()`方法获得。
- $size[i]$表示固定其他维度时，第$i$维的元素数量，即第$i$维的长度，通过`size()`方法获得。

> Python中的多维张量按照**行优先展开**的方式存储，访问矩阵中下一个元素是通过偏移来实现的，这个偏移量称为**步长(stride)**，比如python中，访问$2\times3$矩阵的同一行中的相邻元素，物理结构需要偏移1个位置，即步长为1，同一列中的两个相邻元素则步长为3。

举例说明：

```python
>>>t = torch.arange(12).reshape(3,4)
>>>t
tensor([[ 0,  1,  2,  3],
        [ 4,  5,  6,  7],
        [ 8,  9, 10, 11]])
>>>t.stride(),t.stride(0),t.stride(1) # 返回t两个维度的步长，第0维的步长，第1维的步长
((4,1),4,1)
# 第0维的步长，表示沿着列的两个相邻元素，比如‘0’和‘4’两个元素的步长为4
>>>t.size(1)
4
# 对于i=0，满足stride[0]=stride[1] * size[1]=1*4=4，那么t是连续的。
```



PyTorch中的有一些操作没有真正地改变tensor的内容，只是改变了索引和元素的对应关系，操作之前和操作之后的数据是共享的。这些操作包括`narrow(),view(),expand(),transpose()`等[<sup>[2]</sup>](#2)。当执行这些函数时，原来语义上相邻、内存里也相邻的元素，可能会出现语义上相邻，但内存上不相邻的情况，就不连续(not contiguous)了。

PyTorch提供了两个关于contiguous的方法：

`is_contiguous()`     : 判断Tensor是否是连续的
`contiguous()`	       : 返回新的Tensor，重新开辟一块内存，并且是连续的

举例说明(参考[[1]](#1))：


```python
>>>t = torch.arange(12).reshape(3,4)
>>>t
tensor([[ 0,  1,  2,  3],
        [ 4,  5,  6,  7],
        [ 8,  9, 10, 11]])
>>>t2 = t.transpose(0,1)
>>>t2
tensor([[ 0,  4,  8],
        [ 1,  5,  9],
        [ 2,  6, 10],
        [ 3,  7, 11]])
>>>t.data_ptr() == t2.data_ptr()  # 返回两个张量的首元素的内存地址
True    	#说明底层数据是同一个一维数组
>>>t.is_contiguous(),t2.is_contiguous()  # t连续，t2不连续
(True, False)
```



可以看到，t和t2共享内存中的数据。如果对t2使用`contiguous()`方法，会开辟新的内存空间：

```python
>>>t3 = t2.contiguous()
>>>t3
tensor([[ 0,  4,  8],
        [ 1,  5,  9],
        [ 2,  6, 10],
        [ 3,  7, 11]])
>>>t3.data_ptr() == t2.data_ptr() # 底层数据不是同一个一维数组
False
>>>t3.is_contiguous()
True
```



关于contiguous的更深入的解释可以参考[[1]](#1).



# view()/reshape()

## view()



tensor.[view()](https://pytorch.org/docs/master/tensors.html?highlight=view#torch.Tensor.view)函数返回一个和tensor共享底层数据，但不同形状的tensor。使用`view()`函数的要求是**tensor必须是contiguous的**。

用法如下：

```python
>>>t
tensor([[ 0,  1,  2,  3],
        [ 4,  5,  6,  7],
        [ 8,  9, 10, 11]])
>>>t2 = t.view(2,6)
>>>t2
tensor([[ 0,  1,  2,  3,  4,  5],
        [ 6,  7,  8,  9, 10, 11]])
>>>t.data_ptr() == t2.data_ptr()	# 二者的底层数据是同一个一维数组
True
```



## reshape()



tensor.[reshape()](https://pytorch.org/docs/master/generated/torch.reshape.html#torch.reshape)类似于`tensor.contigous().view()`操作，如果tensor是连续的，则reshape()操作和view()相同，返回指定形状、共享底层数据的tensor；如果tensor是不连续的，则会开辟新的内存空间，返回指定形状的tensor，底层数据和原来的tensor是独立的，相当于先执行`contigous()`，再执行`view()`。

> 如果不在意底层数据是否使用新的内存，建议使用`reshape()`代替`view()`.



# resize_()

tensor.[resize_()](https://pytorch.org/docs/master/tensors.html?highlight=resize#torch.Tensor.resize_)函数，返回指定形状的tensor，与`reshape()`和`view()`不同的是，`resize_()`可以只截取tensor一部分数据，或者是元素个数大于原tensor也可以，会自动扩展新的位置。

`resize_()`函数对于tensor的连续性无要求，且返回的值是共享的底层数据（同`view()`），也就是说只返回了指定形状的索引，底层数据不变的。



# transpose()/permute()

> `permute()`和`transpose()`还有`t()`是PyTorch中的转置函数，其中`t()`函数只适用于2维矩阵的转置，是这三个函数里面最"弱"的。

## transpose()



tensor.[transpose()](https://pytorch.org/docs/master/generated/torch.transpose.html#torch.transpose)，返回tensor的指定维度的转置，底层数据共享，与`view()/reshape()`不同的是，`transpose()`只能实现维度上的转置，不能任意改变维度大小。

对于维度交换来说，`view()/reshape()`和`transpose()`有很大的区别，一定不要混用！混用了以后虽然不会报错，但是数据是乱的，血坑。

> `reshape()/view()`和`transpose()`的区别在于对于维度改变的方式不同，前者是在存储顺序的基础上对维度进行划分，也就是说将存储的一维数组根据shape大小重新划分，而`transpose()`则是真正意义上的转置，比如二维矩阵的转置。

举个例子：

```python
>>>t
tensor([[ 0,  1,  2,  3],
        [ 4,  5,  6,  7],
        [ 8,  9, 10, 11]])
>>> t.transpose(0,1)	# 交换t的前两个维度，即对t进行转置。
tensor([[ 0,  4,  8],
        [ 1,  5,  9],
        [ 2,  6, 10],
        [ 3,  7, 11]])
>>> a.reshape(4,3)     # 使用reshape()/view()的方法，虽然形状一样，但是数据排列完全不同
tensor([[ 0,  1,  2],
        [ 3,  4,  5],
        [ 6,  7,  8],
        [ 9, 10, 11]])
```



## permute()



tensor.[permute()](https://pytorch.org/docs/master/tensors.html?highlight=permute#torch.Tensor.permute)函数，以view的形式返回矩阵指定维度的转置，和`transpose()`功能相同。

与`transpose()`不同的是，`permute()`同时对多个维度进行转置，且参数是**期望的维度的顺序**，而`transpose()`只能同时对**两个维度**转置，即参数只能是两个，这两个参数没有顺序，只代表了哪两个维度进行转置。

举个例子：

```python
>>> t				# t的形状为(2,3,2)
tensor([[[ 0,  1],
         [ 2,  3],
         [ 4,  5]],

        [[ 6,  7],
         [ 8,  9],
         [10, 11]]])
>>> t.transpose(0,1)   # 使用transpose()将前两个维度进行转置，返回(3,2,2)
tensor([[[ 0,  1],
         [ 6,  7]],

        [[ 2,  3],
         [ 8,  9]],

        [[ 4,  5],
         [10, 11]]])
>>> t.permute(1,0,2)   # 使用permute()按照指定的维度序列对t转置，返回(3,2,2)
tensor([[[ 0,  1],
         [ 6,  7]],

        [[ 2,  3],
         [ 8,  9]],

        [[ 4,  5],
         [10, 11]]])
```



# squeeze()/unsqueeze()

## squeeze()

tensor.[squeeze()](https://pytorch.org/docs/master/generated/torch.squeeze.html?highlight=squeeze#torch.squeeze)返回去除size为1的维度的tensor，默认去除所有size=1的维度，也可以指定去除某一个size=1的维度，并返回去除后的结果。

举个例子：

```python
>>> t.shape 
torch.Size([3, 1, 4, 1])
>>> t.squeeze().shape  # 去除所有size=1的维度
torch.Size([3, 4])
>>> t.squeeze(1).shape  # 去除第1维
torch.Size([3, 4, 1])
>>> t.squeeze(0).shape  #　如果指定的维度size不等于1，则不执行任何操作。
torch.Size([3, 1, 4, 1])
```



## unsqueeze()

tensor.[unsqueeze()](https://pytorch.org/docs/master/generated/torch.unsqueeze.html#torch.unsqueeze)与`squeeze()`相反，是在tensor插入新的维度，插入的维度size=1，用于维度扩展。

举个例子：

```python
>>> t.shape
torch.Size([3, 1, 4, 1])
>>> t.unsqueeze(1).shape   # 在指定的位置上插入新的维度，size=1
torch.Size([3, 1, 1, 4, 1]) 
>>> t.unsqueeze(-1).shape  # 参数为-1时表示在最后一维添加新的维度，size=1
torch.Size([3, 1, 4, 1, 1])
>>> t.unsqueeze(4).shape   # 和dim=-1等价
torch.Size([3, 1, 4, 1, 1])
```



# expand()/repeat()

## expand()

tensor.[expand()](https://pytorch.org/docs/master/tensors.html?highlight=tensor%20expand#torch.Tensor.expand)的功能是扩展tensor中的size为1的维度，且只能扩展size=1的维度。以view的形式返回tensor，即不改变原来的tensor，只是以视图的形式返回数据。

举个例子：

```python
>>> t
tensor([[[0, 1, 2],
         [3, 4, 5]]])
>>> t.shape
torch.Size([1, 2, 3])
>>> t.expand(3,2,3)  # 将第0维扩展为3，可见其将第0维复制了3次
tensor([[[0, 1, 2],
         [3, 4, 5]],

        [[0, 1, 2],
         [3, 4, 5]],

        [[0, 1, 2],
         [3, 4, 5]]])
>>> t.expand(3,-1,-1) # dim=-1表示固定这个维度，效果是一样的，这样写更方便
tensor([[[0, 1, 2],
         [3, 4, 5]],

        [[0, 1, 2],
         [3, 4, 5]],

        [[0, 1, 2],
         [3, 4, 5]]])
>>> t.expand(3,2,3).storage()    # expand不扩展新的内存空间
 0
 1
 2
 3
 4
 5
[torch.LongStorage of size 6]
```



## repeat()

tensor.[repeat()](https://pytorch.org/docs/master/tensors.html?highlight=repeat#torch.Tensor.repeat)用于维度复制，可以将size为任意大小的维度复制为n倍，和`expand()`不同的是，`repeat()`会分配新的存储空间，是真正的复制数据。

举个例子：

```python
>>> t
tensor([[0, 1, 2],
        [3, 4, 5]])
>>> t.shape
torch.Size([2, 3])
>>> t.repeat(2,3)  # 将两个维度分别复制2、3倍
tensor([[0, 1, 2, 0, 1, 2, 0, 1, 2],
        [3, 4, 5, 3, 4, 5, 3, 4, 5],
        [0, 1, 2, 0, 1, 2, 0, 1, 2],
        [3, 4, 5, 3, 4, 5, 3, 4, 5]])
>>> t.repeat(2,3).storage()   # repeat()是真正的复制，会分配新的空间
 0
 1
 2
 0
 1
 2
 0
 1
 2
 3
 4
 5
 ......
 3
 4
 5
[torch.LongStorage of size 36]
```

>如果维度size=1的时候，`repeat()`和`expand()`的作用是一样的，但是`expand()`不会分配新的内存，所以优先使用`expand()`函数。



# 总结

1. `view()/reshape()`两个函数用于**将tensor变换为任意形状**，本质是将所有的元素**重新分配**。
2. `t()/transpose()/permute()`用于**维度的转置**，转置和`reshape()`操作是有区别的，注意区分。
3. `squeeze()/unsqueeze()`用于**压缩/扩展维度**，仅在维度的**个数**上去除/添加，且去除/添加的维度size=1。
4. `expand()/repeat()`用于**数据的复制**，对一个或多个维度上的数据进行复制。
5. 以上提到的函数仅有两种会分配新的内存空间：`reshape()`操作处理非连续的tensor时，返回tensor的copy数据会分配新的内存；`repeat()`操作会分配新的内存空间。其余的操作都是返回的视图，底层数据是共享的，仅在索引上重新分配。



# Reference



<span id='1'>1. </span>[PyTorch中的contiguous](https://zhuanlan.zhihu.com/p/64551412)

<span id='2'>2. </span>[stackoverflow-pytorch-contiguous](https://stackoverflow.com/questions/48915810/pytorch-contiguous#)

<span id='3'>3. </span>[PyTorch官方文档](https://pytorch.org/docs/master/index.html)