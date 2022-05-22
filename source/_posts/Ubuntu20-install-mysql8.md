---
title: Ubuntu20安装MySQL8
excerpt: Win11的WSL安装MySQL8
mathjax: true
date: 2022-05-22 21:40:18
tags: [Ubuntu,MySQL]
categories: 数据库
keywords: MySQL,Ubuntu,WSL
---





## 一、版本信息

系统：`Ubuntu 20.04.4`

```bash
root@LAPTOP-4LK64UFH:/etc# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.4 LTS
Release:        20.04
Codename:       focal
```

MySQL：`8.0.29`

```bash
root@LAPTOP-4LK64UFH:/# mysql --version
mysql  Ver 8.0.29-0ubuntu0.20.04.3 for Linux on x86_64 ((Ubuntu))
```



## 二、安装步骤

### 1、卸载旧版本

安装之前确保旧版本已经卸载干净：

```bash
# 卸载mysql-common
sudo apt-get remove mysql-common
# 卸载mysql-server
sudo apt-get autoremove --purge mysql-server-5.0
# 查看MySQL剩余的包，然后卸载
dpkg --list|grep mysql
# 清除残留数据
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P 
# 删除MySQL数据文件
sudo rm /var/lib/mysql/ -R
# 删除MySQL配置文件
sudo rm /etc/mysql/ -R
```



### 2、安装

更新软件库：

```bash
sudo apt update
```

安装MySQL服务：

```bash
sudo apt install mysql-server
# 这里会自动安装mysql-client
```

安装依赖：

```bash
sudo apt install libmysqlclient-dev
```

安装完后，检查状态：

```bash
sudo netstat -tap | grep mysql
```



常用命令：

```bash
# 启动MySQL服务
sudo service mysql start
# 停止MySQL服务
sudo service mysql stop
# 重启MySQL服务
sudo service mysql restart
```





# 三、使用MySQL

MySQL8版本中，用户密码字段为`authentication_string`，并且新增了`caching_sha2_password`加密插件。

### 1、设置用户密码

用户表保存在`mysql`库的`user`表中:

```mysql
ALTER mysql.user 'userName'@'hostName' IDENTIFIED WITH caching_sha2_password BY 'userName';
# 这里用caching_sha2_password，则plugin字段也应该是caching_sha2_password

# 修改完后刷新
flush privileges;
```

使用这个指令前，要确保`authentication_string`字段为空，否则会执行失败。



### 2、远程连接MySQL服务

服务器上安装好MySQL后，客户端连接服务器的数据库，需要检查下面几项内容：

* 服务器防火墙关闭。
* 服务器ssh服务开启。
* 服务端端口打开：使用`netstat -an | grep 3306`指令查看3306端口情况，如果端口前面的地址是`127.0.0.1`，需要将`etc/mysql/mysql.conf.d/mysqld.cnf`中的`bind-address = 127.0.0.1`注释掉，确保其他地址客户端可以连接。



# 四、问题排查



**问题一**：修改完密码登录，提示拒绝登录

**错误信息**：ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)

**原因**：可能是缓存密码的加密方式和当前用的加密插件不一致，**确保设置密码时的加密插件和用户的`plugin`字段是相同的加密插件**。

> 如果第三方客户端不支持caching_sha2_password，可以改成旧的mysql_native_password 加密方式。



**问题二**：无法连接MySQL服务器

**错误信息**：ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (13)

**原因**：1、检查目录是否有指定文件。2、可能是当前用户没有此文件的权限，添加权限即可。

