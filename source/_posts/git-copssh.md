---
layout: post
title: Windows下搭建基于SSH的git代码仓库
date: 2016-08-07 22:35
tags: 	
	git
comments: true
keywords: git, copssh,windows,Blog
description: 内心一点、坚持一点、总会有收获
toc: true
---

前几天因开发需要，出现代码管理问题，考虑到github上托管私有代码要收费等问题
于是就动手开始研究搭建公司内网的git仓库，已经搭建好了，现在就和大家分享下我的搭建过程以及心得体会

<!-- more -->

### 所需软件及下载和安装

[git(客户端和服务端都需要安装)](https://git-for-windows.github.io/)
[copssh(安装在服务端)](https://www.itefix.net/copssh)
至于git的安装不作详细的赘述了,可自行百度，其中要说的主要就是要记得在系统环境变量中作配置如（图1）

![图1](/assets/blogImg/git-copssh/git-global-config.png)

安装完git和配置好之后接着就着重说下copssh的安装和配置

#### copssh安装

下载好了copssh的安装包后直接双击安装后就弹出如（图2）的界面

![图2](/assets/blogImg/git-copssh/copssh_install_1.png)

选好要按装的目录后就配置CopSSH默认的帐号密码如（图3）

（这一步会在你的电脑新增一个管理员帐号，这些参数就是管理员的用户名和密码，而且以后基本上是不会用到的）

![图3](/assets/blogImg/git-copssh/copssh_install_2.png)

a、copssh安装完成后在开始面板中打开copssh

![图4](/assets/blogImg/git-copssh/copssh_config.png)

b、新增用户

![图5](/assets/blogImg/git-copssh/copssh_config_1.png)

c、选择用户(可以是当前系统用户，本例就选着刚安装copssh时创建的用户吧)

![图6](/assets/blogImg/git-copssh/copssh_config_2.png)

d、接着就是默认好了

![图7](/assets/blogImg/git-copssh/copssh_config_3.png)
e、公钥配置：公钥存放地点在copssh的安装路径下如：D:\copssh\ICW\home\git\.ssh\，新建一个文件叫authorized_keys，用任意一个文件编辑器打开，把公钥拷贝进去即可。拷贝进去的公钥就会显示出来（图8）
![图8](/assets/blogImg/git-copssh/copssh_config_key.png)

还有另外一种添加公钥的方式是点击add按钮产生密钥(此出省略几张图)
![图9](/assets/blogImg/git-copssh/copssh_config_key_1.png)
然后你会发现在你的copssh的用户目录中发现产生一个密钥
![图10](/assets/blogImg/git-copssh/copssh_config_key_3.png)
此时打开.ssh目录下的authorized_keys文件你会发现多了一个公钥
![图11](/assets/blogImg/git-copssh/copssh_config_key_2.png)
在服务端产生了公钥之后接着就是在客户端配置私钥：
1、把服务端的私钥拷贝到客户端
2、客户端中，使用puttygen.exe加载私钥产生对应的公钥并保存

在配置完所有的信息后就可以用git base输入
`
ssh git@192.168.0.0
`
git是copssh中配置的用户名，192.168.0.0是IP地址

在登录成功后，若发现git命令行不起作用可做如下图操作
![图12](/assets/blogImg/git-copssh/copssh_config_git.png)
1、打开copssh安装目录下的etc文件夹中的profile文件
2、添加  
`
:/cygdrive/d/Git/bin:/cygdrive/d/Git/mingw64/libexec/git-core
`
(注意开始的分号)到变量PATH





