---
title: 使用Hexo和Github Page快速搭建自己的博客
date: 2016-12-16 14:47:20
categories: 杂
tags: [git, hexo]
---
偶然看到了[Hexo](https://hexo.io/zh-cn/)，和免费的Github Page，对于程序员来说有一个属于自己的网站和分享技术的地方，真是装逼起飞的开始啊。

废话不多说，直接上干货，对了我这个是针对于windows用户的。

# 预装环境
- [Node.js](http://nodejs.org/)
- [Git](http://git-scm.com/)

# 安装Hexo
直接下载安装吧，没什么好说的，windows没那么麻烦的。有了nodejs之后，
``` shell
npm install -g hexo-cli
```
这样就安装好啦

# 建站
安装完Hexo后，执行以下命令，Heox将会在制定文件夹中新建所需要的文件。
``` shell
hexo init blog
cd blog
npm install
```
新建完成后，指定文件夹的目录如下：
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
## _config.yml
它的是最主要的配置文件，我们先不动

# Github建仓库
去github上新建一个仓库，仓库名为`steflerjiang.github.io`

# 上传至仓库
在blog目录下配置`_config.yml`文件，其中的`deploy`项配置为
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github:SteflerJiang/steflerjiang.github.io.git
  branch: master
```

然后执行
```
hexo g
hexo -d
```
这里如果deploy报错的话，`ERROR Deployer not found: git`，运行`npm install hexo-deployer-git --save`估计是依赖没有装上去。
等十几分钟，打开`https://steflerjiang.github.io/`就可以看到你自己的blog啦。

Hexo的其他问题，可以直接看它的官方网站。[Hexo](https://hexo.io/zh-cn/)

# 主题主题主题
本来想用默认的主题随便用用，结果不小心google了一下，发现了[next](http://theme-next.iissnan.com/)，完全不能自拔，太漂亮了

