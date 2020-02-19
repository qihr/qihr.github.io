---
title: CSharp基础14 异常
tags: [CSharp]
categories: Program
date: 2019-12-17 08:55:00


---

 异常相关内容

<!-- more -->

# 1.1概述

系统捕获错误，并抛出一个异常，如果程序没有提供处理该异常的代码，系统会挂起这个程序。

# 1.2 Try语句

try语句由三个部分组成：

- try块：避免出现异常被保护的语句。
- catch语句：处理try块抛出的异常。
- finally块：无论是否有异常都会执行的代码。

catch子句可以有多个。注意finally和catch的顺序，异常一旦被捕获，就不会挂起或者有报错信息。

# 1.3 异常类

BCL中定义了许多表示异常类型的异常类，所有异常类都派生自`System.exception`类。

# 1.4 catch语句

catch语句处理异常，有三种不同级别的处理：

- catch { Statements } ：捕捉任何出现的异常。
- catch(ExceptionType) { Statements }：捕捉匹配任何该名称类型的异常。
- catch(ExceptionType someobj) { Statements }：捕捉匹配任何该名称类型的异常，并接受一个异常变量，用于访问该对象的信息。

带对象的特点catch子句提供异常的最多信息，匹配该类或派生自它的类的异常。还会给出一个异常实例，一个对CLR创建的异常对象的引用。可以在catch内访问异常变量。

catch字句段可以包含多个catch子句，异常发生时，系统按顺序搜索子句列表，第一个匹配该异常对象类型的子句会被执行。因此，catch子句排序有两个重要的原则：

- 最明确的异常类型第一，直至最普通的类型。
- 如果有一个一般的catch语句，必须是最后一个。

# 1.5 Finally语句

finally始终会执行。无论是否有异常抛出。可选，可以不写。

# 1.6 为异常寻找处理程序

如果异常在一个没有try语句保护的块中产生，系统会按顺序搜索调用栈，进一步寻找匹配的处理代码。

# 1.7 抛出异常

throw可以显示抛出一个异常：

```csharp
try
{
    ArgumentNullException myEx = new ArgumentNullExcpetion("arg");
    throw myEx;
}
catch(ArgumentNullException e)
{
    Console.WriteLine("Message:",e.Message);
    //还可以将异常在catch中抛出，让其他异常处理程序处理。
    //重新抛出异常，没有附加参数。
    throw;
}
```















