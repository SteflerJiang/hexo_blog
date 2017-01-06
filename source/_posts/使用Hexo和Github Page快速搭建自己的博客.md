---
title: 使用Hexo和Github Page快速搭建自己的博客
date: 2016-12-16 14:47:20
categories: 杂
tags: [git, hexo]
---
偶然看到了[Hexo](https://hexo.io/zh-cn/)，和免费的Github Page，对于程序员来说有一个属于自己的网站和分享技术的地方，真是装逼起飞的开始啊。

废话不多说，直接上干货，对了我这个是针对于windows用户的。
<!--more-->
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

看到一个很详细的教程在这里[Hexo搭建GitHub博客（三）- NexT主题配置使用](https://zhiho.github.io/2015/09/29/hexo-next/)

# sitemap
sitemap的意思就是站点地图，意味着你要告诉搜索引擎你网站的哪些网址是可以被收录的。

我一共给两个搜索引擎添加了sitemap，分别是google和百度。废话不多说，对于hexo 3.x版本
```
cd your-site
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
```
执行完之后，打开你根目录下的`package.json`文件，会发现dependencies里面多了两行
```
"hexo-generator-baidu-sitemap": "^0.1.2",
"hexo-generator-sitemap": "^1.1.2"
```
打开你的**站点配置文件**，在最下面加入如下配置：
```
<!-- 不要用网上的path配置，会导致deploy失败 -->
baidusitemap:
  path: baidusitemap.xml

sitemap:
  path: sitemap.xml
```

这就代表安装成功了。
然后执行
```
hexo g
hexo d
```
重新生成和部署。

## 百度
打开并登录[百度站长平台](http://zhanzhang.baidu.com/linksubmit/url)，在链接提交中提交你自己的博客地址，比如我自己的就是`https://steflerjiang.github.io/`，提交完成之后有个验证过程，意思就是证明这个网站归你所有，我选择用html验证，很简单，在根目录下的`index.html`的***head***标签中加一行代码验证一下就行，加完之后重新执行`hexo d`，等个1分钟就可以验证了。

然后去[sitemap提交](http://zhanzhang.baidu.com/linksubmit/index?tab=sitemap)选择sitemap，填写自己的sitemap地址，我的地址如下`https://steflerjiang.github.io/baidusitemap.xml`，输入验证码提交即可。

## Google
google和百度基本的流程差不多，打开[Google Search Console](https://www.google.com/webmasters/tools/home?hl=zh-CN)，点击**添加属性**，选择网站并提交自己的网站地址，也用alternate methods中的html tag方法来完成验证，跟百度的一样。

然后在左侧选择，抓取->站点地图，提交自己的sitemap地址，比如我的就是`https://steflerjiang.github.io/sitemap.xml`，等一两天左右就行了。
