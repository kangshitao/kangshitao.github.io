---
title: 建立Hexo个人博客
excerpt: 记录本博客的创建和功能完善的过程
date: 2020-10-25 19:15:21
mathjax: true
tags: ['hexo','blog']
categories: 博客
keywords: hexo-blog,个人博客搭建，zhaoo
top: 10
---

> 使用hexo和github.pages搭建博客，主题为[zhaoo](https://github.com/izhaoo/hexo-theme-zhaoo)

# 准备工作

github创建名为username.github.io的repository

安装git

安装Node.js



# 安装Hexo

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



# 更换主题

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



# 关联git库

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



# 新建文章

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

# 数学公式渲染

> hexo默认的Markdown渲染器是hexo-renderer-marked，会先按照Markdown语法解析，然后才是LaTex，所以会有冲突。试了网上各种解决方法，终于遇到一个有效的方法：[hexo无法显示公式的问题-DGZ's Blog](https://www.dazhuanlan.com/2020/03/07/5e633a3b46a81/)

## 重装插件

首先，只保留一个公式渲染器，这里保留kramed渲染器。

```
# 卸载默认的渲染器，其他的公式渲染器也卸载，只保留kramed
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-kramed --save
```

## 修改js文件

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

## 安装mathjax插件

```
# 如果有hexo-math，则卸载掉
npm uninstall hexo-math --save
npm install hexo-renderer-mathjax --save
```

## 修改html文件

打开`blog/node_modules/hexo-renderer-mathjax/mathjax.html，将：`

```
<script src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
```

修改为：

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>
```

## 激活插件

在使用公式的md文件添加属性：

```
mathjax: true
```

## 部署

如果是第一次修改，修改完以后需要重新启动服务：

```
hexo clean
hexo g
hexo s
```

> 弄完才发现，主题作者添加了公式渲染的说明，方法简单快捷，可以[参考](https://www.izhaoo.com/2020/05/05/hexo-theme-zhaoo-doc/#%E6%95%B0%E5%AD%A6%E5%85%AC%E5%BC%8F)

# 图床

众所周知，github是万能的，使用[PigGo](https://picgo.github.io/PicGo-Doc/)和Github搭建个人图床，参考[PigGo官方文档](https://picgo.github.io/PicGo-Doc/zh/guide/#%E5%90%AC%E8%AF%B4%E4%BD%A0%E4%B9%9F%E6%83%B3%E7%94%A8picgo)

## 新建github项目

新建github项目用来存放图片，并生成token：

```
github头像->Settings->Developer settings->Generate new token
将repo选项打勾
```

## 配置PicGo

![1](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/blogcreatprocess_1.png)

然后点击确定，并设置为默认图床。

自定义域名可以按照图中的设置，也可以使用别的自定义域名。第二种域名方式就是使用[CDN](https://en.wikipedia.org/wiki/Content_delivery_network)加速的方法，CDN(Content Delivery Network)是指内容分发网络，也称为内容传送网络。

[jsDelivr](https://www.jsdelivr.com/?docs=gh)是一种免费的CDN加速产品，可以加速Github和NPM的资源，其中使用jsDelivr的方法访问Github资源只需要将链接改为以下格式：

```
https://cdn.jsdelivr.net/gh/username/repository@version/file   
# 其中的version指的是仓库版本，如果没有release版本，也可以使用分支名称
```

> jsDelivr的更多用法可以参考官方文档

使用jsDelivr加速的自定义域名填写内容如下：

![2](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/blogcreatprocess_2.png)

## 上传图片

将图片拖入上传区，上传成功后会自动在剪切板生成图片链接。

## 其他设置

其他参数设置可根据个人喜好进行设置：

![3](https://cdn.jsdelivr.net/gh/kangshitao/BlogPicture@main/img/blogcreatprocess_3.png)

## 图片上传失败问题

由于某些原因，github当作图库不是很稳定，时好时坏也是正常现象。

参考[PicGo踩坑记](https://blog.csdn.net/TalesOV/article/details/104450037?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.channel_param)

- 首先检查设置的仓库名是否正确，仓库名不可包含空格，且不要出现奇怪的符号
- 上传的图片名字不要有奇怪符号
- 间歇性上传失败的话，可以将server的开关状态切换一下

另外，有条件的可以使用全局代理:airplane:

# 其他

主题的其他设置可以参考[zhaoo主题文档](https://www.izhaoo.com/2020/05/05/hexo-theme-zhaoo-doc/)，这里只记录一部分

## 访问量统计

使用[leancloud](https://leancloud.cn/)，这里的统计是每篇文章的访问量，zhaoo主题支持leancloud，只需要按要求填写即可：

```
进入leancloud，创建应用(开发版)
创建Class，ACL权限选择无限制
打开设置-应用keys
将AppID、AppKey、Rest API服务器地址复制到themes/_config.yml
```

对于网站的访问量统计，使用[卜蒜子](http://busuanzi.ibruce.info/)，在`themes\zhaoo\layout\_partial\footer.ejs`中合适的位置加入以下代码，其中的字体显示内容和风格可以自由设置：

```
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"</script>
<span id="busuanzi_container_site_pv">
  总浏览量：<span id="busuanzi_value_site_pv" style="font-size:12px;"></span> 次
</span>
<span id="busuanzi_container_site_uv" >
  总访客数：<span id="busuanzi_value_site_uv" style="font-size:12px;"></span>人
</span>
```

## 评论功能

使用[Valine](https://valine.js.org/)，前提是主题支持：

```
leancloud,创建应用(开发版)
创建Class，配置默认
打开设置-应用keys
将AppID、AppKey复制到themes/_config.yml
```

## 侧边目录

zhaoo没有支持侧边目录，需要自己添加代码，使用hexo官方的toc函数

在`themes\zhaoo\layout\_partial\post\article.ejs`中，最前面加以下代码：

```
<aside class="post-widget">
  <nav id="toc" class="toc-article fixed">
	<strong class="toc-title">文章目录</strong>
	  <%- toc(page.content, {
	    class: 'post-toc',
		list_number: false,
        max_depth: '6',
	    min_depth: '1'}) %>
	</nav>
</aside>
```

然后在`themes\zhaoo\source\css\_partial\post.styl`中，最前面添加以下代码：

```
.post-widget
  float right
  width 15%
  padding-left 10px
  min-hidden 1px
.toc-article.fixed
  top 76px
  bottom 140px
  overflow-y auto
.toc-article
  position fixed
  overflow-x hidden
.toc-title
  padding-left 20px
```

下一步计划，目录跟随页面高亮显示当前位置标题……参考[Hexo折腾笔记](https://unnamed42.github.io/2016-09-10-Hexo%E6%8A%98%E8%85%BE%E7%AC%94%E8%AE%B0.htm)



## SEO优化

>搜索引擎优化（Search Engine Optimization,SEO），是一种通过了解搜索引擎的运作规则来调整网站，以及提高目的网站在有关搜索引擎内排名的方式。

Github.pages屏蔽了百度的爬虫请求，配置较麻烦，这里仅以Google为例，使用[Google Search Console](https://search.google.com/search-console/about)，以下过程参考：[hexo博客搭建(五) SEO优化](https://zhuanlan.zhihu.com/p/35400128)

```
1、首先去google search console验证，有多种方式可选。
2、添加站点地图 sitemap
npm install hexo-generator-sitemap --save  # 安装站点地图自动生成的插件
在_config.yml文件中添加以下代码：
sitemap:
  path: sitemap.xml
3、部署之后可以去google search console测试。
```



其他设置问题，持续更新...

---





# 参考教程

> [hexo官方文档](https://hexo.io/zh-cn/docs/)
>
> [hexo史上最全搭建教程](https://blog.csdn.net/sinat_37781304/article/details/82729029)