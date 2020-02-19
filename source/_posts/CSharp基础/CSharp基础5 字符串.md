---
title: CSharp基础5 字符串
tags: [CSharp]
categories: Program
date: 2020-02-19 23:52:19 
---

 String，StringBuilder

<!-- more -->

# 1.1概述



# 1.2 String

字符串是unicode字符串数组，是不可变的，不能被修改。所以对于一个string来说，任何改变都会分配一个新的恒定字符串。



# 1.3 StringBuilder

StringBuilder可以动态有效的产生字符串,避免创建许多副本。和string不同，StringBulider对象会被确实的修改。

- StringBuilder类是BCL的成员，位于System.Text命名空间中。
- StringBuilder对象是Unicode字符的可变数组。

# 1.4 把字符串解析为数据值

解析允许接受表示值的字符串，并把它转换为实际值。所有预定义的简单类型都有一个Parse方法，用于解析。

```csharp
double d = double.Parse("25.973");
```

Parse方法的缺点是如果string不能成功转换目标类型，会抛出一个异常。Tryparse方法可以避免这个问题。

```csharp
bool succes = int.TryParse(stringFirst,out intFirst)
```

