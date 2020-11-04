---
title: Markdown简单教程
excerpt: Markdown基本语法介绍
date: 2020-10-25 16:19:14
mathjax: true
tags: Markdown
categories: 教程
keywords: Markdown,markdown教程,typora
---



_本文作为 Markdown基础语法教程，同时作为个人笔记 &练习。使用 Typora工具，语法可能略有不同，以实际平台为准_

# 标题

Markdown标题有两种格式

## 使用 = 和 - 标记一级和二级标签

语法如下：

```
一级标题
=====

二级标题
-----
```

效果如下：

![1](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/markdown_1.png)

## 使用#号标记

语法如下：

```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

> 注意：#号后面要加一个空格

# 段落

换行：使用shift+enter键换行。

换段：Markdown的段落要求前后保留一个空行，Typora编辑器直接回车即可，或者先换行再加一个空行，表示更换段落。

> 两个或以上的空格+shift+enter，实现段内强制换行（行尾有↓箭头）。

# 字体格式

字体格式：

```
*斜体文本*
_斜体文本_
**粗体文本**
__粗体文本__
***粗斜体文本***
___粗斜体文本___
```

显示效果如下：
*斜体文本*
_斜体文本_
**粗体文本**
__粗体文本__
***粗斜体文本***
___粗斜体文本___

#  列表

Markdown支持无序列表和有序列表。

## 无序列表：

无序列表有三种写法，`+` 或`-`或`*`后加空格即可：

```
+ 第一项
* 第一项
- 第一项
```

显示效果：

![2](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/markdown_2.png)

## 有序列表

有序列表使用数字和`.`再加上空格即可：

```
1. 第一项
2. 第二项
3. 第三项
```

显示效果：

![3](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/markdown_3.png)

## 列表嵌套

列表嵌套需要在子列表选项前加四个空格：

```
1. 第一项：
    - 第一项的第一个元素
    - 第一项的第二个元素
2. 第二项：
    - 第二项的第一个元素
    - 第二项的第二个元素
```

显示效果：

![4](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/markdown_4.png)
> Typora中，子列表选项前不加空格也可以。

# 插入区块和分隔线

## 区块

Markdown的区块引用是在段落开头使用`>` 号和空格：

```
> 区块引用
```

显示效果：

![5](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/markdown_5.png)

另外，区块引用也可以像列表一样嵌套，一个`>` 表示第一层，两个`>` 表示第二层，以此类推：

```
> 第一层
>> 第二层
>>> 第三层
```

区块中也可以使用列表，同样，列表中也可以使用区块。

## 分隔线

在一个空行中，使用三个或以上的`*`、`+`、`-`、`_`来创建分隔线：

```
***
++++
-----
______
```

# 插入代码

## 句中代码

如果是句中的代码片段或函数，只需要一对反引号`` `即可 (位于Tab键上面):

```
`printf()`函数
```

显示效果：

`pringf()`函数

## 代码块

代码块需要两个以上反引号，可以指定语言，以下为python语言的代码块：

```
​```
def main():
	print('hello world')
​```
```


显示效果：

```python
def main():
	print('hello world')
```

> 还可以使用缩进式方法，使用四个空格或者一个制表符(Tab键)，一段代码会持续到没有缩进的那一行

#  插入链接和图片

## 插入链接

插入链接有两种方法：

```
[链接名称](链接地址)
```

或者：

```
<链接地址>
```

例如：
方法一：[github](https://github.com/)
方法二：<https://github.com/>

## 插入图片

插入图片的方法与插入链接的方法区别在于，插入图片的语法前加`!`号：

```
![属性名称](图片地址)
```

还可以使用<img>标签，用来指定图片大小：

```
<img src="图片地址" width="80%"
```

> Markdown需要以链接的形式插入图片，如果md文件发布到网站，则链接可能会失效，所以建议使用图床保存图片，这样无论md文件在哪里，只要链接不失效，就可以看到图片。

## 高级链接

可以通过变量来设置一个链接，在文档末尾为变量赋值：

```
网址链接[链接名称][1]
插入图片![][2]
[1]:https://github.com/
[2]:https://www.google.com.hk/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png
```

# 插入表格

Markdown制作表格使用`|` 来分隔不同的单元格，使用`-`分隔表头和其他行，代码如下：

```
| 表头 | 表头|
| --- | ---|
|单元格|单元格|
|单元格|单元格|
```

效果如下：

| 表头   | 表头   |
| ------ | ------ |
| 单元格 | 单元格 |
| 单元格 | 单元格 |

**对齐方式** ：

- `-:`设置内容和标题栏居右对齐
- `:-`设置内容和标题栏居左对齐
- `:-:`设置内容和标题栏居中对齐

代码如下：

```
|左对齐|右对齐|居中对齐|
|：---|---：|：----：|
|单元格|单元格|单元格|
|单元格|单元格|单元格|
```

效果如下：

| 左对齐 | 右对齐 | 居中对齐 |
| :----- | -----: | :------: |
| 单元格 | 单元格 |  单元格  |
| 单元格 | 单元格 |  单元格  |

# HTML元素

Markdown支持部分html标签，比如：`<kbd>`、`<b>`、`<i>`、`<em>`、`<sup>`、`<sub>`、`<br>`，以及`<fond>`等html标签：

```
使用 <kbd>Ctrl</kbd>+<kbd>c</kbd> 复制
<font color="red" size=5>我是红色字体,大小为</font>
```

效果如下：

使用 <kbd>Ctrl</kbd>+<kbd>c</kbd> 复制
<font color="red" size=5>
我是红色字体,大小为5</font>

# 其他

## 转义

使用`\`符号转移特殊字符：

```
**文本加粗**
\*\*正常显示星号\*\*
```

效果如下：

**文本加粗**
\*\*正常显示星号\*\*

## 插入公式

使用两个美元符号`$$`将TeX或LaTeX格式的数字公式包裹，即可插入公式：

> 由于渲染器的问题，两个花括号中间必须要有个空格

```
$
\sqrt{ {x^2}+{y^2} }+v_1+v_2
$
```

效果如下：

$\sqrt{ {x^2}+{y^2} }+v_1+v_2$

## 插入流程图

Typora插入横向流程图：

```
​```mermaid
graph LR
A[方形] -->B(圆角)
    B --> C{条件a}
    C -->|a=1| D[结果1]
    C -->|a=2| E[结果2]
    F[横向流程图]
​```
```


效果如下：

![6](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/markdown_6.png)

> 更多类型的流程图、顺序图等可以参考：[typora画流程图、时序图(顺序图)、甘特图](https://jingyan.baidu.com/article/48b558e3035d9a7f38c09aeb.html)



---



# 参考链接


>* [最完整的Markdown教程-简书](https://www.jianshu.com/p/335db5716248)
>* [Markdown教程|菜鸟教程](https://www.runoob.com/markdown/md-tutorial.html)

