---
title: Google Java风格指南
date: 2017-01-19 10:27:32
categories: Java
tags: Java
---

转载一篇Google的Java编程风格指南，自己手打一遍，也算是阅读，顺带添加以下原文和自己的理解。

# 1 简介 Introduction

> This document serves as the complete definition of Google's coding standards for source code in the Java™ Programming Language. A Java source file is described as being in Google Style if and only if it adheres to the rules herein.

> Like other programming style guides, the issues covered span not only aesthetic issues of formatting, but other types of conventions or coding standards as well. However, this document focuses primarily on the hard-and-fast rules that we follow universally, and avoids giving advice that isn't clearly enforceable (whether by human or tool).

<!-- more -->

这份文档是Google Java编程风格规范的完整定义。当且仅当一个Java源文件符合此文档中的规则，我们才认为它符合Google的Java编程风格。

和其他的编程风格指南一样，这里所讨论的不仅仅是编码格式美不美观的问题，同事也讨论一些约定及编码标准。然而这份文档主要侧重于我们所普遍遵循的规则，对于那些不是明确强制要求的，我们尽量避免提供意见。

## 1.1 术语说明 Terminology notes

> In this document, unless otherwise clarified:

> 1. The term class is used inclusively to mean an "ordinary" class, enum class, interface or annotation type (@interface).
> 2. The term member (of a class) is used inclusively to mean a nested class, field, method, or constructor; that is, all top-level contents of a class except initializers and comments.
> 3. The term comment always refers to implementation comments. We do not use the phrase "documentation comments", instead using the common term "Javadoc."

> Other "terminology notes" will appear occasionally throughout the document.

在本文档中，除非另有说明：

1. 术语class可表示普通类，枚举类，接口或者annotation类型(@interface)
2. 术语member(of a class)可表示一个内部类，字段，方法或者构造器，即一个类中出了初始化和注释外的所有顶层内容。
3. 术语comment只用来指代实现的注释(implementation comments)。我们不使用"documentation comments"一词，而是Javadoc。

其他的术语说明会偶尔贯穿整个文档。
<span id="jump">Hello World</span>

## 1.2 指南说明 Guide notes

> Example code in this document is non-normative. That is, while the examples are in Google Style, they may not illustrate the only stylish way to represent the code. Optional formatting choices made in examples should not be enforced as rules.

文档中的示例代码并不作为规范。也就是说，虽然示例代码是遵循Google编程风格的，但并不意味着这是展现这些代码的唯一方式。示例中的格式选择不应该被强制定为规则。

# 2 源文件基础 Source file basics

## 2.1 文件名 File name

> The source file name consists of the case-sensitive name of the top-level class it contains (if which there is [exactly one](#md-anchor)), plus the `.java` extension.

源文件以其最顶层的类名来命名，大小写敏感，文件扩展名为`.java`。

## 2.2 文件编码: UTF-8 File encoding: UTF-8

> Source files are encoded in UTF-8.

源文件格式编码为UTF-8。

## 2.3 特殊字符 Special characters
## 2.3.1 空白字符 Whitespace characters
> Aside from the line terminator sequence, the ASCII horizontal space character (0x20) is the only whitespace character that appears anywhere in a source file. This implies that:

> 1. All other whitespace characters in string and characters literals are escaped.
> 2. Tab characters are not used for indentation.

除了行结束符，ASCII水平空格符(0x20，即空格)是源文件中唯一允许出现的空白字符，这意味着：

1. 所有其他字符串中的空白字符都要进行转义。
2. 制表符不用于缩进。

### 2.3.2 特殊转义序列 Special escape sequences

> For any character that has a special escape sequence (\b, \t, \n, \f, \r, \", \' and \\\\), that sequence is used rather than the corresponding octal (e.g. \012) or Unicode (e.g. \u000a) escape.

对于具有特殊[转义序列](https://zh.wikipedia.org/wiki/%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97)的任何字符(\b, \t, \n, \f, \r, \", \' and \\\\)，我们使用它的转义序列，而不是相应的八进制(比如`\012`)或Unicode(比如`\u000a`)转义。

### 2.3.3 非ASCII字符 Non-ASCII characters

> For the remaining non-ASCII characters, either the actual Unicode character (e.g. ∞) or the equivalent Unicode escape (e.g. \u221e) is used. The choice depends only on which makes the code easier to read and understand, although Unicode escapes outside string literals and comments are strongly discouraged.

>> Tip: In the Unicode escape case, and occasionally even when actual Unicode characters are used, an explanatory comment can be very helpful.
>> Tip: Never make your code less readable simply out of fear that some programs might not handle non-ASCII characters properly. If that should happen, those programs are broken and they must be fixed.


对于剩余的非ASCII字符，是使用实际的Unicode字符(比如∞)，还是使用等价的Unicode转义符(比如\u221e)，取决于哪个能让代码更易于阅读和理解。
> Tip: 在使用Unicode转义符或者一些实际的Unicode字符时，建议做些注释给出解释，这有助于别人阅读和理解。

例如：

```java
String unitAbbrev = "μs"; //                              | 赞，即使没有注释也非常清晰
String unitAbbrev = "\u03bcs"; // "μs"                    | 允许，但没有理由要这样做
String unitAbbrev = "\u03bcs"; // Greek letter mu, "s"    | 允许，但这样做显得笨拙还容易出错
String unitAbbrev = "\u03bcs"; //                         | 很糟，读者根本看不出这是什么
return '\ufeff' + content; // byte order mark             | Good，对于非打印字符，使用转义，并在必要时写上注释
```

> Tip: 永远不要由于害怕某些程序可能无法正确处理非ASCII字符而让你的代码可读性变差。当程序无法正确处理非ASCII字符时，程序肯定是出问题了，需要去修bug。

# 3 源文件结构 Source file structure



> 作者：Hawsteain
> 出处：[原作者链接](http://hawstein.com/posts/google-java-style.html)
