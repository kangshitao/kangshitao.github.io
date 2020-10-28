---
title: 使用Hexo搭建博客过程
date: 2020-10-25 19:15:21
mathjax: true
tags: ['hexo','blog']
categories: 教程
---

> 使用hexo和github.pages搭建博客

## 准备工作

github创建名为username.github.io的repository

安装git

安装Node.js



## 安装Hexo

新建文件夹blog，存放博客的所有内容

```
# 安装hexo
npm install hexo-cli -g
# 确认是否安装成功
hexo -v
# init
hexo init
npm install
```

生成的几个文件：

- node_modules：依赖包
- public：存放生成的页面
- scaffolds：生成文章的一些模版
- source：用来存放文章
- themes：主题
- _config.yml: 配置文件

```
hexo server
```

然后在浏览器输入localhost:4000即可看到生成的博客。



## 更换主题

hexo默认主题是landscape，可以自由更换主题：

```
#下载主题到themes/zhaoo路径
git clone git@github.com:izhaoo/hexo-theme-zhaoo.git themes/zhaoo
```

根据需求创建tags、about、categories页面：

```
hexo new page tags
hexo new page about
hexo new page categories
```

以上代码会在`source`的对应文件夹下生成index.md文件，可根据需求配置页面:

```
# 以tags的index为例，将layout设置为tags布局，不同与一般的布局。
layout: tags
```

> layout(布局)有三种：post(文章)、draft(草稿)、page(页面)，默认是post

然后根据主题配置说明，配置主题的配置文件`themes/zhaoo/_config.yml`



##　关联git库

```
# 安装要用的插件
npm install hexo-deployer-git --save
```

修改`blog/_config.yml`配置文件：

```
deploy：
  type: git
  repo: git@github.com:username/username.github.io.git
  branch: master
```

上述代码表示部署到指定的库中，并且默认部署到分支master。

部署的机制是将本地生成的`.deploy_git`文件夹里的文件，上传到远程库，并不包括配置文件和主题文件。

> 为了方便多终端工作，建议每次写完文章，除了deploy操作以外，将源代码也push到远程仓库保存。新建一个分支，将源文件push到这个分支里面，这样在另一台机器上只需要clone远程库，安装必要的文件就可以了。



## 新建文章

```
# 创建新的文件，默认保存在source/_post路径下
hexo new 'new blog'
# run server
hexo server
# 清除缓存
hexo clean
# 生成静态网页
hexo generate/hexo g
# 将网页部署到远程站点
hexo deploy/hexo d
```

在`scaffolds`文件夹中，有三种默认布局新建时的模版，可以自定义新建文件时的格式。

tags和categories属性的值可以用列表，表示多个标签/类别。

## 数学公式渲染

> hexo默认的Markdown渲染器是hexo-renderer-marked，会先按照Markdown语法解析，然后才是LaTex，所以会有冲突。试了网上各种解决方法，终于遇到一个有效的方法：[hexo无法显示公式的问题-DGZ's Blog](https://www.dazhuanlan.com/2020/03/07/5e633a3b46a81/)

### 1、重装插件

首先，只保留一个公式渲染器，这里保留kramed渲染器。

```
# 卸载默认的渲染器，其他的公式渲染器也卸载，只保留kramed
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-kramed --save
```

### 2、修改js文件

打开`bolg/node_modules/hexo-renderer-kramed/lib/renderer.js`，将：

```
//change inline math rule
function formatText(text) {
    // Fit kramed's rule: $$ + 1 + $$
    return text.replace(/`$(.*?)$`/g, '$$$$$1$$$$');
}
```

修改为：

```
// Change inline math rule
function formatText(text) {
    return text;
}
```

### 3、安装mathjax插件

```
# 如果有hexo-math，则卸载掉
npm uninstall hexo-math --save
npm install hexo-renderer-mathjax --save
```

### 4、修改html文件

打开`blog/node_modules/hexo-renderer-mathjax/mathjax.html，将：`

```
<script src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
```

修改为：

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>
```

### 5、激活插件

在使用公式的md文件添加属性：

```
mathjax: true
```

### 6、部署

如果是第一次修改，修改完以后需要重新启动服务：

```
hexo clean
hexo g
hexo s
```

## 图床

众所周知，github是万能的，使用[PigGo](https://picgo.github.io/PicGo-Doc/)和Github搭建个人图床，参考[PigGo官方文档](https://picgo.github.io/PicGo-Doc/zh/guide/#%E5%90%AC%E8%AF%B4%E4%BD%A0%E4%B9%9F%E6%83%B3%E7%94%A8picgo)

### 1、新建github项目

新建github项目用来存放图片，并生成token：

```
github头像->Settings->Developer settings->Generate new token
将repo选项打勾
```

### 2、配置PicGo

![](https://raw.githubusercontent.com/kangshitao/BlogPicture/main/img/blogcreatprocess_1.png)

然后点击确定，并设置为默认图床

### 3、上传图片

将图片拖入上传区，上传成功后会自动在剪切板生成图片链接。

### 4、其他设置

其他参数设置可根据个人喜好进行设置：

![](https://raw.githubusercontent.com/kangshitao/BlogPicture/main/img/blogcreatprocess_2.png)

### 5、图片上传失败问题

由于某些原因，github当作图库不是很稳定，时好时坏也是正常现象。

参考[PicGo踩坑记](https://blog.csdn.net/TalesOV/article/details/104450037?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.channel_param)

- 首先检查设置的仓库名是否正确，仓库名不可包含空格，且不要出现奇怪的符号
- 上传的图片名字不要有奇怪符号
- 间歇性上传失败的话，可以将server的开关状态切换一下

另外，有条件的可以使用全局代理:airplane:

## 其他

关于博客的其他设置问题，以后再更新...

---





## 参考教程：

> [hexo官方文档](https://hexo.io/zh-cn/docs/)
>
> [hexo史上最全搭建教程](https://blog.csdn.net/sinat_37781304/article/details/82729029)