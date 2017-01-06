---
title: Java 异常处理
date: 2016-12-22 11:32:07
categories: Java
tags: [Java]
---
最近在学习Java的时候发现对异常处理机制理解还不是太深刻，再网上摘抄整理一些内容。

# Java异常处理
异常是程序中的一些错误，但并不是所有的错误都是异常，并且错误有时候是可以避免的。

比如说，你的代码少了一个分号，那么运行出来结果是提示错误java.lang.Error；如果你用System.out.println(11/0)，那么是因为你用了0做除数，会抛出java.lang.ArithmeticException的异常。
<!-- more -->

异常发生的原因有很多，通常包含以下几大类：
- 用户输入了非法的数据
- 要打开的文件不存在
- 网络通信时连接终端，或者JVM内存溢出
这些异常有的时候是因为用户错误引起的，有的是程序错误引起的，还有其他一些是物理错误引起的。

要理解Java异常处理是如何工作的，需要掌握以下三种类型的异常：
- **检查性异常**：最具代表的检查性异常是用户错误或问题引起的异常，这是程序员无法预见的。例如要打开一个不存在的文件时，一个异常就发生了，这些异常在编译时不能被简单地忽略。
- **运行时异常**：运行时异常是可能被程序员避免的异常。与检查性异常相反，运行时异常可以在编译时被忽略。
- **错误**：错误不是异常，而是脱离程序员控制的问题。错误在代码中通常被忽略。例如当栈溢出时，一个错误就发生了，它们在编译时也检查不到。

# Exception类的层次
所有的异常类是congestionjava.lang.Exception类继承的子类。Exception类是Throwable类的子类。

Java程序通常不捕获错误。错误一般发生在严重故障时，它们在Java程序处理的范畴之外。Error用来指示运行时环境发生的错误。例如JVM内存溢出，一般的程序不会从错误中恢复。

异常类有两个主要的子类：IOExceition和RuntimeException类。
![Java异常类继承关系](/images/Java Exception.png "Java异常类继承关系")
在Java内置类中，有大部分常用检查性和非检查性异常。

# Java内置异常类
Java语言定义了一些异常类在java.lang标准包中。标准包运行时异常类的子类是最常见的异常类。由于java.lang包是默认加载到所有的Java程序的，所以大部分从运行时异常类继承而来的异常都可以直接使用。

Java根据各个类库也定义了一些其他的异常，下面的表中列出了Java的非检查性异常。

|异常|描述|
|-|-|
|ArithmeticException|当出现异常的运算条件时，抛出此异常，例如除0的时候|
|ArrayIndexOutOfBoundsException|用非法索引访问数组时抛出的异常。如果索引为负或者大于等于数组大小，则该索引为非法索引。|
|ArrayStoreException|试图将错误类型的对象存储到一个对象数组时抛出的异常|
|CLassCastException|当试图将对象强制转换为不是实例的子类时，抛出该异常|
|IllegalArgumentException|抛出的异常表明向方法传递了一个不合法或不正确的参数|
|IllegalMonitorStateException|抛出的异常表明某一线程已经试图等待对象的监视器，或者试图通过其他正在等待对象的监视器而本身没有制定监视器的线程|
|IllegalStateException|在非法或不适当的时间调用方法时产生的信号。换句话说，即Java环境或Java应用程序没有出去请求操作所要求的适当状态下。|
|IllegalThreadStateException|线程没有处于请求操作所要求的适当状态时抛出的异常|
|IndexOutOfBoundsException|指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出|
|NegativeArraySizeException|如果应用程序试图创建大小为负的数组，则抛出该异常|
|NullPointerException|当程序驶入在需要对象的地方使用Null时，抛出该异常|
|NumberFormatException|当程序试图将字符串转换成一种数值类型，但该字符串不能转换为适当格式时，抛出该异常|
|SecurityException|由安全管理器抛出的异常，指示存在安全侵犯|
|StringIndexOutOfBoundsException|此异常由String方法抛出，指示索引为负，或者超出字符串的大小|
|UnSupportedOperationException|当不支持请求的操作时，抛出该异常|

下面的表列出了Java定义在java.lang包中的检查性异常类。

|异常|描述|
|-|-|
|ClassNotFoundException|程序试图加载类时，找不到相应的类，抛出该异常|
|CloneNotSupportedException|当调用Object类中的clone方法克隆对象，但该对象的类无法实现Clonneable接口时，抛出该异常|
|IllegalAccessExceoption|拒绝访问一个类的时候，抛出该异常|
|InstantiationException|当试图使用Class类中的newInstance方法创建一个类的实例，而指定的类对象因为是一个接口或是一个抽象类而无法实例化时，抛出该异常|
|InterrupedException|一个线程被另一个线程终端，抛出该异常|
|NoSuchFieldException|请求的变量不存在|
|NoSuchMethonException|请求的方法不存在|

# 异常方法
下面的列表是Throwable类的主要方法：

|序号|方法及说明|
|-|-|
|1|`public String getMessage()`返回关于发生的异常的详细信息。这个消息在|
|2|`public Throwable getCause()`返回一个THrowable对象代表异常原因|
|3|`public String toString()`使用getMessage()的结果返回类的串级名称|
|4|`public void printStackTrace()`打印toString()结果和栈层次到System.err，即错误输出流|
|5|`puvlic StackTraceElement[] getStackTrace()`返回一个包含堆栈层次的数组，下标为0的元素代表栈顶，最好一个元素代表方法调用堆栈的栈底|
|6|`public Throwable fillInStackTrace()`用当前的调用栈层次填充Throwable对象栈层次，添加到栈层次任何先前信息中|

# 捕获异常
使用try和catch关键词可以捕获异常，try/catch代码块放在异常可能发生的地方。使用try/catch的语法如下：
```java
try {
    // 程序代码
} catch (Exception e) {
    // catch 块
}
```

# 多重捕获块
```java
try{
   // 程序代码
}catch(异常类型1 异常的变量名1){
  // 程序代码
}catch(异常类型2 异常的变量名2){
  // 程序代码
}catch(异常类型2 异常的变量名2){
  // 程序代码
}
```
上面的代码段包含了 3 个 catch块。
可以在 try 语句后面添加任意数量的 catch 块。
如果保护代码中发生异常，异常被抛给第一个 catch 块。
如果抛出异常的数据类型与 ExceptionType1 匹配，它在这里就会被捕获。
如果不匹配，它会被传递给第二个 catch 块。
如此，直到异常被捕获或者通过所有的 catch 块。

# throw/throws关键字
如果一个方法没有捕获一个检查性异常，那么该方法必须使用throws关键字来声明。throws关键字放在方法签名的尾部，也可以使用throw关键字抛出一个异常，无论它是新实例化的还是刚捕获到的。

下面方法的声明抛出一个 RemoteException 异常：
```java
import java.io.*;
public class className
{
     void deposit(double amount) throws RemoteException
    {
        // Method implementation
        throw new RemoteException();
    }
    //Remainder of class definition
}
```

# finally关键字
finally关键字用来创建在try代码块后面执行的代码块。无论是否发生异常，finally代码块中的代码总会被执行。在finally代码块中，可以运行清理类型等收尾善后性质的语句。语法如下：
```java
try{
    // 程序代码
}catch(异常类型1 异常的变量名1){
    // 程序代码
}catch(异常类型2 异常的变量名2){
     程序代码
}finally{
    // 程序代码
}
```
注意事项：
- catch不能独立于try存在
- 在try/catch后面添加finally块并非强制性要求
- try代码后不能既没有catch块又没有finally块
- try，catch，finally块之间不能添加任何代码

# 声明自定义异常
编写自定义异常类需要注意：
- 所有异常类都必须是Throwable的子类
- 如果希望写一个检查性异常类，则需要继承Exception类
- 如果希望写一个运行时异常类，则需要继承RuntimeException类
一个实例
```java
import java.io.*;

public class InsufficientFundsException extends Exception
{
    private double amount;
    public InsufficientFundsException(double amount)
    {
        this.amount = amount;
    }
    public double getAmount()
    {
        return amount;
    }
}
```
为了展示如何使用我们自定义的异常类，在下面的 CheckingAccount 类中包含一个withdraw() 方法抛出一个 InsufficientFundsException 异常。
```java
import java.io.*;
 
public class CheckingAccount
{
   private double balance;
   private int number;
   public CheckingAccount(int number)
   {
      this.number = number;
   }
   public void deposit(double amount)
   {
      balance += amount;
   }
   public void withdraw(double amount) throws
                              InsufficientFundsException
   {
      if(amount <= balance)
      {
         balance -= amount;
      }
      else
      {
         double needs = amount - balance;
         throw new InsufficientFundsException(needs);
      }
   }
   public double getBalance()
   {
      return balance;
   }
   public int getNumber()
   {
      return number;
   }
}
```
下面的 BankDemo 程序示范了如何调用 CheckingAccount 类的 deposit() 和 withdraw() 方法。
```java
public class BankDemo
{
   public static void main(String [] args)
   {
      CheckingAccount c = new CheckingAccount(101);
      System.out.println("Depositing $500...");
      c.deposit(500.00);
      try
      {
         System.out.println("\nWithdrawing $100...");
         c.withdraw(100.00);
         System.out.println("\nWithdrawing $600...");
         c.withdraw(600.00);
      }catch(InsufficientFundsException e)
      {
         System.out.println("Sorry, but you are short $"
                                  + e.getAmount());
         e.printStackTrace();
      }
    }
}
```


# 通用异常
在Java中定义了两种类型的异常和错误
- JVM异常：由JVM排除的异常或错误，例如NullPointerException类，ArrayIndexOutOfBoundsException类，ClassCastException类
- 程序级异常：由程序或者API程序抛出的异常。例如 IllegalArgumentException 类，IllegalStateException 类。
