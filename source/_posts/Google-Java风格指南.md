---
title: Google Java风格指南
date: 2017-01-19 10:27:32
categories: Java
tags: Java
---

转载一篇Google的Java编程风格指南，自己手打一遍，也算是阅读，顺带添加以下原文和自己的理解。

[参考文章](http://www.hawstein.com/posts/google-java-style.html)

[原文](https://google.github.io/styleguide/javaguide.html)

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
### 2.3.1 空白字符 Whitespace characters
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

> A source file consists of, in order:
> 
1. License or copyright information, if present
2. Package statement
3. Import statements
4. Exactly one top-level class
5. Exactly one blank line separates each section that is present.

一个源文件应该按顺序包含一下几块：

1. 许可证或者版权信息
2. package语句
3. import语句
4. 一个顶级类
5. 每个section之间用一个空行分隔

## 3.1 许可证或者版权信息 License or copyright information, if present
> If license or copyright information belongs in a file, it belongs here.

如果一个文件包含许可证或者版权信息，那么它应该被放在文件最前面

## 3.2 Package语句 Package statement
> The package statement is not line-wrapped. The column limit (Section 4.4, Column limit: 100) does not apply to package statements.

package语句不需要换行。列限制(4.4节， 列长度限制：100)并不适用于package语句。(即package语句写在一行里)

## 3.3 Import语句 Import statement
### 3.3.1 import不要使用通配符 No wildcard imports
> Wildcard imports, static or otherwise, are not used.

即，不要出现类似这样的import语句：`import java.util.*`;

### 3.3.2 不要换行 No line-wrapping
> Import statements are not line-wrapped. The column limit (Section 4.4, Column limit: 100) does not apply to import statements.

import语句不换行，列限制(4.4节)并不适用于import语句。(每个import语句独立成行)

### 3.3.3 顺序和间隔 Ordering and spacing
> Imports are ordered as follows:
> 
1. All static imports in a single block.
2. All non-static imports in a single block.
If there are both static and non-static imports, a single blank line separates the two blocks. There are no other blank lines between import statements.
>
Within each block the imported names appear in ASCII sort order. (Note: this is not the same as the import statements being in ASCII sort order, since '.' sorts before ';'.)

import语句的顺序如下：

1. 每个静态导入为一块
2. 所有非静态导入为一块

如果既有静态导入又有非静态导入，那么两个块之间用一个空行隔开。每个块之间的导入不需要其他空行。

每个块之间import语句按照ASCII顺序排列。(注意：)

### 3.3.4 类的非静态导入 No static import for classes
> Static import is not used for static nested classes. They are imported with normal imports.

静态导入不适于用静态内部类。把它们看做正常的导入就行了。

## 3.4 类声明 Class declaration
### 3.4.1 只有一个顶级类声明 Exactly one top-level class declaration
> Each top-level class resides in a source file of its own.

每个顶级类都在一个与它同名的源文件中(当然，还包含`.java`后缀)。

### 3.4.2 类成员顺序 Ordering of class contents
>The order you choose for the members and initializers of your class can have a great effect on learnability. However, there's no single correct recipe for how to do it; different classes may order their contents in different ways.

> What is important is that each class uses some logical order, which its maintainer could explain if asked. For example, new methods are not just habitually added to the end of the class, as that would yield "chronological by date added" ordering, which is not a logical ordering.

类的成员顺序对易学性有很大的影响，但这也不存在唯一的通用法则。不同的类对成员的排序可能是不同的。

最重要的一点，每个类应该以某种逻辑去排序它的成员，维护者应该要能解释这种排序逻辑。比如， 新的方法不能总是习惯性地添加到类的结尾，因为这样就是按时间顺序而非某种逻辑来排序的。

#### 3.4.2.1 重载：永不分离 Overloads: never split
> When a class has multiple constructors, or multiple methods with the same name, these appear sequentially, with no other code in between (not even private members).

当一个类有多个构造器时，或者有多个同名方法，它们应该连续出现，中间不允许有其他代码，即使是私有成员对象也不行。

# 4 格式 Formatting
> Terminology Note: block-like construct refers to the body of a class, method or constructor. Note that, by Section 4.8.3.1 on array initializers, any array initializer may optionally be treated as if it were a block-like construct.

术语说明：块状结构(block-like construct)指一个类，方法或者构造函数的主题。需要注意的是，在节4.8.3.1中数组初始化的初始值也可被选择性地视为块状结构。

## 4.1 大括号 Braces
### 4.1.1 使用大括号(即使是可选的) Braces are used where optional
> Braces are used with if, else, for, do and while statements, even when the body is empty or contains only a single statement.

大括号与`if, else, for, do, while`语句一起使用，即使只有一条语句(或是空)，也应该把大括号写上。

### 4.1.2 非空块：K & R风格  Nonempty blocks: K & R style
>Braces follow the Kernighan and Ritchie style ("Egyptian brackets") for nonempty blocks and block-like constructs:

>1. No line break before the opening brace.
2. Line break after the opening brace.
3. Line break before the closing brace.
4. Line break after the closing brace, only if that brace terminates a statement or terminates the body of a method, constructor, or named class. For example, there is no line break after the brace if it is followed by else or a comma.

对于非空块和块状结构，大括号应遵循Kernighan和Ritchie风格([Egyptian brackets](https://blog.codinghorror.com/new-programming-jargon/)):

Examples:
```java
return () -> {
  while (condition()) {
    method();
  }
};

return new MyClass() {
  @Override public void method() {
    if (condition()) {
      try {
        something();
      } catch (ProblemException e) {
        recover();
      }
    } else if (otherCondition()) {
      somethingElse();
    } else {
      lastThing();
    }
  }
};
```
> A few exceptions for enum classes are given in Section 4.8.1, Enum classes.

节4.8.1给出了enum类的一些例外。

### 4.1.3 空块：可以用简洁版本 Empty blocks: may be concise

>An empty block or block-like construct may be in K & R style (as described in Section 4.1.2). Alternatively, it may be closed immediately after it is opened, with no characters or line break in between ({}), unless it is part of a multi-block statement (one that directly contains multiple blocks: if/else or try/catch/finally).

一个空的块状结构里什么也不包含，大括号可以简洁地写成`{}`，不需要换行。例外：如果它是一个多块语句的一部分(`if/else` 或 `try/catch/finally`) ，即使大括号内没内容，右大括号也要换行。

Examples:
```
  // This is acceptable
  void doNothing() {}

  // This is equally acceptable
  void doNothingElse() {
  }
```

```
  // This is not acceptable: No concise empty blocks in a multi-block statement
  try {
    doSomething();
  } catch (Exception e) {}
```


## 4.2 块缩进：2个空格 Block indentation: +2 spaces

> Each time a new block or block-like construct is opened, the indent increases by two spaces. When the block ends, the indent returns to the previous indent level. The indent level applies to both code and comments throughout the block. (See the example in Section 4.1.2, Nonempty blocks: K & R Style.)

每次新加一个块或者块状结构，缩进需要增加2个空格(。。。我自己写的都是4个空格)。当块结束时，缩进返回先前的级别。缩进级别同时使用于代码和注释。

## 4.3 一行一个语句 One statement per line

> Each statement is followed by a line break.

每一个语句都应该独占一行。

## 4.4 列限制：100 Column limit: 100

> Java code has a column limit of 100 characters. Except as noted below, any line that would exceed this limit must be line-wrapped, as explained in Section 4.5, Line-wrapping.

>Exceptions:

>1. Lines where obeying the column limit is not possible (for example, a long URL in Javadoc, or a long JSNI method reference).
2. package and import statements (see Sections 3.2 Package statement and 3.3 Import statements).
3. Command lines in a comment that may be cut-and-pasted into a shell.

Java代码一行最多100个字符。除了以下限制以外，任何超过列数限制的行必须重新换行

以下例外：
1. 不可能满足列限制的行(例如，Javadoc中的一个长URL，或是一个长的JSNI方法参考)。
2. `package`和`import`语句(见3.2节和3.3节)。
3. 注释中那些可能被剪切并粘贴到shell中的命令行。

## 4.5 自动换行 Line-wrapping

>Terminology Note: When code that might otherwise legally occupy a single line is divided into multiple lines, this activity is called line-wrapping.

>There is no comprehensive, deterministic formula showing exactly how to line-wrap in every situation. Very often there are several valid ways to line-wrap the same piece of code.

>>Note: While the typical reason for line-wrapping is to avoid overflowing the column limit, even code that would in fact fit within the column limit may be line-wrapped at the author's discretion.

>>Tip: Extracting a method or local variable may solve the problem without the need to line-wrap.

术语说明：一般情况下，一行长代码为了避免超出列限制而被分为多行，我们称之为自动换行(line-wrapping)。

我们并没有全面，确定性的准则来决定在每一种情况下如何自动换行。很多时候，对于同一行代码会有好几种有效的自动换行方式。

> Note: 有时候，一行代码即使在列限制以内，为了避免超出列限制，程序员也会考虑自动换行。

> Tip: 提取方法或局部变量可以在不换行的情况下解决代码过长的问题(是合理缩短命名长度吧)

### 4.5.1 从哪里断开 Where to break

> The prime directive of line-wrapping is: prefer to break at a higher syntactic level. Also:

> 1. When a line is broken at a non-assignment operator the break comes before the symbol. (Note that this is not the same practice used in Google style for other languages, such as C++ and JavaScript.)
    - This also applies to the following "operator-like" symbols:
        - the dot separator (.)
        - the two colons of a method reference (::)
        - an ampersand in a type bound (<T extends Foo & Bar>)
        - a pipe in a catch block (catch (FooException | BarException e)).
2. When a line is broken at an assignment operator the break typically comes after the symbol, but either way is acceptable.
    - This also applies to the "assignment-operator-like" colon in an enhanced for ("foreach") statement.
3. A method or constructor name stays attached to the open parenthesis (() that follows it.
4. A comma (,) stays attached to the token that precedes it.
5. A line is never broken adjacent to the arrow in a lambda, except that a break may come immediately after the arrow if the body of the lambda consists of a single unbraced expression. Examples:

```
MyLambda<String, Long, Object> lambda =
    (String label, Long value, Object obj) -> {
        ...
    };
Predicate<String> predicate = str ->
    longExpressionInvolving(str);
```

> Note: The primary goal for line wrapping is to have clear code, not necessarily code that fits in the smallest number of lines.

自动换行的基本准则是：更倾向于在更高的语法级别处断开。

1. 如果在`非赋值运算符`处断开，那么在该符号**前**断开。(注意: 这一点与Google其他语言的编程风格不同，如c++或者js)。
    - 这也适用于一下的`类运算符号`：
        - 点分隔号 (.)
        - 方法引用中的双冒号 (::)
        - 类型界限中的& (<T extends Foo & Bar>)
        - catch语句中的管道符号 (catch (FooException | BarException e))
2. 如果在`赋值运算符处`断开，通常的做法是在该符号后断开(比如=，它与前面的内容留在同一行)。
    - 这条规则也适用于foreach语句中的分号。
3. 方法名或构造函数名与左括号留在同一行。
4. 逗号(,)与其前面的内容留在同一行。
5. 在lambda语句中不要在箭头附近换行，除非，箭头后面只有一行没有被大括号括起来的语句，

注意: 自动换行最主要的目的是为了让代码更整洁，而不是为了让代码行数最少。

### 4.5.2 自动换行时至少缩进4+空格 Indent continuation lines at least +4 spaces

> When line-wrapping, each line after the first (each continuation line) is indented at least +4 from the original line.
> 
> When there are multiple continuation lines, indentation may be varied beyond +4 as desired. In general, two continuation lines use the same indentation level if and only if they begin with syntactically parallel elements.
> 
> Section 4.6.3 on Horizontal alignment addresses the discouraged practice of using a variable number of spaces to align certain tokens with previous lines.

自动换行时，第一行后的每一行至少比第一行多缩进4个空格。

当存在连续自动换行时，缩进可能会多缩进不只4个空格(语法元素存在多级时)。一般而言，两个连续行使用相同的缩进当且仅当它们开始于同级语法元素。

第4.6.3水平对齐一节中指出，不鼓励使用可变数目的空格来对齐前面行的符号。

## 4.6 Whitespace 空格









> 作者：Hawsteain
> 出处：[原作者链接](http://hawstein.com/posts/google-java-style.html)
