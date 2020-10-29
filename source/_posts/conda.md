---
title: conda常用命令
mathjax: true
categories:
  - 教程
tags:
  - conda
date: 2020-10-28 21:53:39
---



> conda是一个开源的软件包管理系统和环境管理系统，用于安装多个版本的软件包及其依赖关系，并在它们之间轻松切换。
>
> [Anaconda](https://www.anaconda.com/)是一个开源的Python发行版本，包含了conda、python等180多个科学包及其依赖项。
>
> [Miniconda](https://docs.conda.io/en/latest/miniconda.html)是最小的conda安装环境，只包含最基本的内容——python和conda，以及相关的必须依赖项。

---
> **参考文档：官方[conda命令参考](https://docs.conda.io/projects/conda/en/latest/commands.html)**

# Conda 指令

## 查看指令

```
conda -V 或 conda --version	# 查看conda版本
```

```
conda env list 或 conda info --envs  # 查看存在哪些虚拟环境
```

## 更新conda

```
conda update conda  # 检查并更新当前conda
```

## 创建虚拟环境

```
conda create -n [env_name] python=x.x	# 创建python版本号为x.x的虚拟环境
```

## 激活/切换虚拟环境

```
Windows： activate [env_name]
Linux: source activate [env_name]
```

## 关闭/退出虚拟环境

```
conda deactivate  #　退出当前虚拟环境
activate [env_name]  # 切换到其他环境，也相当于关闭当前环境
```

## 删除虚拟环境

```
conda remove -n [env_name] --all
```

## 设置镜像

```
conda config --add channels [mirroe_url]   # 添加镜像地址
conda config --set show_channel_urls yes   # 设置搜索时显示通道地址
conda config --get channels                # 查看已经配置的channels
conda config --remove-key channels  	   # 恢复为默认地址(删除所有其他镜像url)
```

---

# Conda的package管理命令

## 查看已安装的包

```
conda list	# 查看当前环境已安装的所有包
conda list -n [env_name]	# 查看指定环境已安装的所有包
```

## 安装包

```
conda install [package_name]	# 当前环境下安装包
conda install -n [env_name] [package_name]	# 指定环境先安装包
# 还可以使用-c参数，指定通过某个channel安装
```

## 更新包

```
conda update [package_name]		# 更新当前环境下的包
conda update -n [env_name] [package_name]	# 更新指定环境下的包
```

## 删除包

```
conda remove [package_name]		# 删除当前环境下的包
conda remove -n [env_name] [package_name	# 删除指定环境下的包
```

## 查找包的信息

```
conda search [package_name]	# 查找指定包的信息
```

