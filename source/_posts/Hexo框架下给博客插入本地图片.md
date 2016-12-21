---
title: Hexo框架下给博客插入本地图片
date: 2016-12-20 19:16:18
categories: 杂
tags:
---
终于能给博客插图了，以前一直以为要把图片单独上传到七牛云这些线上服务器才行，hexo可以直接用本地的服务器了。
<!--more-->

## hexo中图片路径
要加入图片，现在source下建立一个与博文_posts同级别的文件夹images，将图片放入，此时的图片路径为：

`/hexo/source/images/myImage.png`

在博客中插入，要使用**相对路径**，应写为`/images/myImage.png`

## markdown插入图片语法
markdown中插入图片语法如下：

`![图片注释](/images/myImg.png "图片标题")`

这种方法无法指定图片的尺寸与对齐方式。

## 另一种插入方法：HTML
使用嵌入HTML来插入图片，可以更好地控制图片的显示方式。
```html
<img src="/images/myImage.png" width=50% height=50% align=center/>
```

插入一个喜欢的dota图片吧
***
用markdown语法插入 `![dota插图](/images/four heroes.jpg "Dota")`
![dota插图](/images/four heroes.jpg "Dota")
***
用html语法插入 `<img src="/images/four heroes.jpg" width=50% height=50% align=center/>`
<img src="/images/four heroes.jpg" width=50% height=50% align=center/>
