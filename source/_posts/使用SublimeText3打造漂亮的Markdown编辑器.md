---
title: 使用SublimeText3打造漂亮的Markdown编辑器
date: 2016-12-16 15:13:15
categories: 杂
tags: [sublime, markdown]
---
因为之前做的是手游开发python + lua，所以编辑器一直用的是sublime，然后为了写markdown特地去安装了一个markdown编辑器，后来发现原来sublime也可以变成一个漂亮的Markdown编辑器。

# 预装环境
[Sublime Text](http://www.sublimetext.com/)它的各种便利我就不多说了，安装完sublime后的第一件事情就是给它安装一个`package control`。

在sublime中使用快捷键ctrl+`或者是打开菜单中的View > Show Console，输入以下python代码
``` python
import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
稍等一会，package control就安装完成了。

快捷键`ctrl+shift+p`，输入`install package`点击回车，就可以安装想要的任何插件啦。

# MarkdownEditing
MarkdownEditing是Markdown写作者必备的插件，它可以不仅可以高亮显示Markdown语法还支持很多编程语言的语法高亮显示。

# OmniMarkupPreviewer
OmniMarkupPreviewer用来预览markdown 编辑的效果，同样支持渲染代码高亮的样式。

安装完之后，重启sublime即可，点击`control+alt+o`就可以一边写markdown，一边预览自己的成果啦。

如果OmniMarkupPreviewer报404错，记得把OmniMarkupPreviewer中的user setting中添加如下设置即可。
```
{
    "renderer_options-MarkdownRenderer": {
        "extensions": ["tables", "fenced_code", "codehilite"]
    }
}
```
