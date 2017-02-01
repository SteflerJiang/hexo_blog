---
title: git多账户配置
date: 2016-12-16 14:07:43
categories: 杂
tags: git
---
# 背景介绍
在公司工作，内部有一个git仓库，然后自己平时也喜欢在github上分享自己的代码，所以需要在同一台机器上有两个账户。之前配置好了公司账户的环境，自然不能去clone自己github上的代码了，所以需要搞多账号。
<!--more-->

# 删除全局配置
很多人按照网上的教程都会配置全局，`git config --global`做这样的操作，如下:
``` shell
git config --global user.name "name"
git confit --global user.email "email"
```
如果你的机器上只有一个git账户是绝对没有问题的，多账号就不同了，不同账户只用不同的用户名和邮箱。

取消方法如下：
``` shell
git config --global --unset user.name
git config --global --unset user.email
```

# ssh配置
安装好git之后，右击打开git bash，在这里可以输入一些常用的linux命令。

``` shell
ssh-keygen -C "email1-work" -t rsa
```
生成的公钥和私钥的名字分别为id_rsa.pub, id_rsa，将这个公钥添加到公司的git账号中去
``` shell
ssh-keygen -C "email2-github" -t rsa
```
这里不要一直点回车键，在选择保存方式的时候存另外一个名字，叫id_rsa_github.pub, id_rsa_github，将这个公钥添加到github账号中去。

当然要让ssh都识别这两个秘钥，因此，需要
``` shell
ssh-agent bash
ssh-add ~/.ssh/id_rsa_github
```
这样你机器上的ssh就可以区分两个秘钥了。

下面要做的事情，就是配置ssh。进入ssh目录，windows用户通常在C:\Users\username\.ssh的目录下
``` shell
cd ~/.ssh
```
这里新建一个config文件，配置如下
```
# 该文件用于配置私钥对应的服务器
# Default github user(first@mail.com)
Host company #给公司的host起一个别名
HostName xxxx.com #公司git的地址
User user1
IdentityFile ~/.ssh/id_rsa

# second user(second@mail.com)
# 建一个github别名，新建的帐号使用这个别名做克隆和更新
Host github
HostName github.com
User steflerjiang
IdentityFile ~/.ssh/id_rsa_github
```

有了这样的配置以后，直接clone吧
不过要注意的就是clone的地址需要改一下，打个比方，我博客目前仓库的地址是
``` shell
git@github.com:SteflerJiang/steflerjiang.github.io.git
```
这里需要改成
``` shell
git@github:SteflerJiang/steflerjiang.github.io.git
```
因为我在config里面已经吧`github.com`给设置了一个别名叫`github`。
意思就是在这个地址下，使用`id_rsa_github`这个秘钥来验证。

同样的，在clone了之后需要进入该目录，设置git的name和eamil
``` shell
cd steflerjiang.github.io.git
git config user.name "steflerjiang"
git config user.email "steflerjiang@xxx.com"
```
