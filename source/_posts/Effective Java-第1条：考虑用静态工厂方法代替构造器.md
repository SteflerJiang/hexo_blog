---
title: Effective Java-第1条：考虑用静态工厂方法代替构造器
date: 2016-12-17 10:45:09
categories: Java
tags: [Java, 设计模式]
---
静态工厂方法与构造器不同的优势在于
1. 它们有名称
2. 不必在每次调用它们的时候都创建一个新对象
3. 它们可以返回原返回类型的任何子类型的对象
4. 在创建参数化类型实例的时候，它们使得代码变得更加简洁
<!-- more -->

静态工厂方法的主要缺点在于
1. 类如果不含有共有的或者受保护的构造器，就不能被子类化
2. 它们与其他的静态方法实际上没有任何区别


