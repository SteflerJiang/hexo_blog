---
title: Java8的lambda和默认方法
date: 2017-03-20 11:48:10
categories: Java
tags: [Java8, lambda, interface]
---

# Labmda
Lambda的基本语法是

`(parameters) -> expression`

或者

`(parameters) -> {expression;}`


这里讨论一种异常情况。假设你用与Runnable同样的签名声明了一个函数接口，我们称之为Task。

```Java
interface Task {
    void exexute();
}

public static void doSomething(Runnable r) {r.run();}
public static void doSomething(Task t) {t.execute();}
```
如果我们用以前的匿名类是没问题的:
```Java
doSomething(new Task() {
    void execute() {
        System.out.println("Danger danger!!");
    }
})
```
但是将这种匿名类转换为Lambda表达式时，就导致了一种晦涩的方法调用，因为Runnable和Task都是合法的目标类型：

```Java
doSomething(() -> System.out.println("Danger danger!!"));
```

这时候编译器会报一个编译的异常，`Ambigulous method call`，这时候需要对Task尝试使用显示的类型转换来解决这种模棱两可的情况:

```Java
doSomething((Task)() -> System.out.println("Danger danger!!"));
```

# 接口默认方法
## 概念
Java8引入了一种新的机制。Java8中的接口现在支持在声明方法的同事提供实现。通过两种方式可以完成这种操作。
1. Java8允许在接口内声明静态方法。
2. Java8引入了一个新功能，叫*默认方法*。

在新的Collection接口中的stream和List接口中的sort都是这样的。定义如下：

```Java
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}

@SuppressWarnings({"unchecked", "rawtypes"})
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();
    Arrays.sort(a, (Comparator) c);
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```

默认方法设计的目的是为了解决**向接口添加方法**。

## Java8中的抽象类和抽象接口
那么既然接口也能实现方法，抽象类也能实现方法，区别如下：
1. 一个类只能继承一个抽象类，但是一个类可以实现多个接口。
2. 一个抽象类可以通过实例变量(字段)保存一个通用状态，而接口是不能有实例变量的。

## 解决冲突的规则
先看下面的场景
```Java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B {
    default void hello() {
        System.out.println("Hello from B");
    }
}

public class C implements B, A {
    public static void main(String[] args) {
        new C().hello();
    }
}
```

###解决问题的三条规则
如果一个类使用相同的函数前面从多个地方继承了方法，通过三条规则可以进行判断。
1. 类中的方法优先级最高。类或父类中声明的方法的优先级高于任何声明为默认方法的优先级。
2. 如果无法依据第一条进行判断，那么子接口的优先级更高：函数前面相同时，优先选择拥有最具体实现的默认方法的接口，即如果B继承了A，那么B就比A更加具体。
3. 最好，如果还是无法判断，继承了多个接口的类必须通过显示覆盖和调用期望的方法，显示地选择使用哪一个默认方法的实现。

Java8引入了一种新的语法`X.super.m()`，其中X是你希望调用的m方法所在的父接口。
