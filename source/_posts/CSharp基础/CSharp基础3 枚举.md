---
title: CSharp基础3 枚举
tags: [CSharp]
categories: Program
date: 2020-02-19 23:51:08 
---

概述，底层类型，位标志

<!-- more -->

## 枚举

### 1.1 概述

- 枚举是值类型，只有一种类型的成员：命名的整数数值常量。
- 每个枚举类型都有一个底层整数类型，默认int。
- 每个枚举成员都会被赋予一个底层类型的常量值
- 默认情况下，第一个成员的赋值为0，后面一次+1.
- 枚举成员的名字不可以重复，但是值可以。
- 枚举只有单一的成员类型。
- 不能对成员使用修饰符。隐式的具有和枚举相同的可访问性。
- 比较不同的枚举类型编译会报错，哪怕值和结构完全一样。
- 

### 1.2 底层类型和显示值

可以使枚举使用除int以外的**整数类型**：

```csharp
enum Enumeg：ulong
{}
```

枚举可以显式的赋值常量，但如果不初始化，编译器会隐式的赋值。规则如下：

```mermaid
graph LR;
    id1[第一个成员有初始化吗]--是-->id2(把它设置为初始化值);
    id1--否-->id3(把它设置为0);
    id3-->id4(下一个成员有初始化吗);
    id4--是-->id2
    id4--否-->id5(给它赋一个比前一个多1的值)
    id2-->id4;
```

### 1.3 位标志

单个枚举变量可以表示多种状态。书中称之为标志字，flagword。

使用的方式：

1. 确认需要多少个位标志，选择足够多的无符号正数类型保存。
2. 确认每个位代表什么，声明一个选好的整数类型用位表示，如0001。
3. 用按位或添加一个枚举状态 0001 or 0010 = 0011
4. 使用按位与，或hasflag解开一个状态。

例子：

```csharp
[Flags]
enum CardDeckSettings:uint
{
    one = 0x01，//位0
    two = 0x02，//位1 
    tree = 0x04，//位2
}
```

使用位标志，枚举中任何成员的值都是2的n次方。标志字加一个状态用位或，减去一个状态用位与。

在.net4.8之后，可以使用HasFlag()方法来确定某个枚举变量是否包含一个状态：

```csharp
bool enumsome = ops.HasFlag(CardDeckSettings.One); //ops,标志字（枚举变量）

//或者这样
CardDeckSettings testflags = CardDeckSettings.one|CardDeckSettings.two;
bool haveenum =  ops.HasFlag(testflags);

//也可以这样,判断ops标志字是否包含one枚举。
bool haveenum =  (ops & CardDeckSettings.one) == CardDeckSettings.one;

//原理：
0|....|0|0|0|1 //one
0|....|0|0|1|0 //two
0|....|0|1|0|0 //tree
0|....|0|1|0|1 //ops（包含tree和one）
//位与检查
0|....|0|0|0|1 //one
0|....|0|1|0|1 //ops
//位与
0|....|0|0|0|1 // ops&one
```

### 1.4 Flags特性

添加了flags特性会加入一些方便的特性，它会告诉编译器，对象浏览器以及其他查看代码的工具，该枚举的成员不仅可以当作单独的值，还可以按位进行组合。其次，允许枚举的tosrting方法为位标志的值提供更多的格式化信息。这样tostring会取匹配枚举的成员然后返回枚举成员的字符串名称。

例如：

```csharp
enum CardDeckSettings:uint
{
    one = 0x01，//位0
    two = 0x02，//位1 
    tree = 0x04，//位2
}
class Program
{
    static void Main()
    {
        CardDeckSettings ops;
        ops = CardDeckSettings.one;
        Console.WriteLine(ops.ToString());
        
        ops = CardDeckSettings.one | CardDeckSettings.tree;
        Console.WriteLine(ops.ToString());
    }
}

//输出结果：
//1.one
//2.5
```

可以看到，如果没加flags的ops含有多个枚举成员，则输出只会显示值，因为tostring方法没有找到哪个枚举成员是3的值。

如果加了flags就是这样的：

```csharp
//输出结果：
//1.one
//2.one,tree
```







