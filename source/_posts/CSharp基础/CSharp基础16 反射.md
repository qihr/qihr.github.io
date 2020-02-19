---
title: CSharp基础16 预处理指令
tags: [CSharp]
categories: Program
date: 2019-12-17 08:55:00



---

 反射相关内容

<!-- more -->

# 1.1概述



# 1.2 元数据和反射

有关程序及其类型的数据被称为元数据`metadata`，特性可以给类型添加元数据。元数据保存在程序的程序集中。

程序运行时，可以查看其他程序集或其本身的元数据。一个运行的程序查看本身的元数据或其他程序的元数据的行为叫做反射。对象浏览器是显示元数据的程序的一个例子。

# 1.3 Type类

BCL中有一个叫做Type的抽象类，被设计用来包含类型的特性，使用这个类的对象可以获取程序使用的类型的信息。

Type是抽象类，因此不能有实例，在运行时，CLR创建从Type派生的类实例，Type包含了类型信息。访问这些实例时，CLR不会返回派生类的引用而是type基类的引用。

注意事项：

- 程序中用到的每一个类型，CLR都会创建一个包含这个类型信息的Type类型的对象。
- 程序中用到的每一个类型都会关联到独立的Type类对象。
- 不管创建的类型有多少个实例，只有一个Type对象会关联到所有这些实例。

## 1.3.1 获取Type对象

Object类型包含一个`GetType()`方法，返回实例的Type对象引用,所以可以在任何类型对象上使用`GetType()`方法。还有一个Getfields方法。

或者使用typeof运算符获取type对象：

```csharp
Type t = typeof(DerivedClass)
```

# 1.4 特性

特性(attribute)允许向程序的程序集增加元数据的语言结构，应用了特性的程序结构叫目标。

- 用来获取和使用元数据的程序叫做特性的消费者，例如对象浏览器。
- .Net预定了很多特性，用户也可以自定义特性。

在源代码中，我们将特性应用于程序结构，编译器获取源代码并且从特性产生元数据，然后把元数据放到程序集中。消费者程序可以把获取特性的元数据以及程序中其他组件的元数据。

编译器同时生成和消费特性。

## 1.4.1 应用特性

特性的目的：告知编译器将程序结构的某组元数据嵌入程序集。使用方法具体看例子：

```csharp
[Serializable]
public class MyClass
{...}

[MyAttribute("Simple class","Version 3.57")]
public class MyClass
{...}
```

## 1.4.2 自定义特性

特性值是某个特殊类型的类。所有特性类都派生自System.Attribute。

### 声明自定义特性

声明一个自定义特性，需要做如下工作：

- 声明一个派生自System.Attribute的类。
- 给类起一个Attribute结尾的名字。

由于特性持有目标的信息，所有特性类的公共成员只能是：字段，属性，构造函数。

### 使用特性的构造函数

和其他类一样，编译器会生成一个隐式公共无参的构造函数。构造函数也可以被重载。构造函数必须以Attribute后缀结尾。

### 指定构造函数

为目标应用特性时，其实是在指定该使用哪个构造函数来创建特性的实例。在应用特性时，构造函数的实参必须是在编译器能确定的常量表达式。

如果构造方法没有参数，可以省略圆括号：

```csharp
[MyAttr]
class someclass
```

### 使用构造函数

只有特想的消费者访问特性时才能调用构造函数，应用一个特性是一条声明语句，不会决定什么时候构造特性类的对象。和普通类的构造函数的，普通类的构造时命令式的，特性时声明式的。区别如下：

- 命令语句：在这里创建一个新的对象。
- 声明语句：这份特性和这个目标关联，如果需要构造特性，使用这个构造函数。

特性的构造函数同样可以使用位置参数和命名参数。

### 限制特性的使用

AttributeUsage特性可以应用到自定义的特性上，用于限制自定义特性只能被特定的结构使用：

```csharp
[AttributeUsage(AttributeTarget.Method| AttributeTarget.Constructor) ]
public sealed class MyAttributeAttribute:System.Attribute
{...}
```

限制特性的多个实例不能应用到同一个目标上：

```csharp
[AttributeUsage(AttributeTarget.Class,Inherited = false, AllwoMultiple = false) ]
public sealed class MyAttributeAttribute:System.Attribute
{...}
```

限制有很多种，用的时候再去查。

### 自定义特性的实践

可以参考如下规则:

- 特性应该表示目标结构的一些状态。
- 如果特性需要某些字段，可以通过包含具有位置参数的构造函数来收集数据，可选字段可以采用命名参数按需初始化。
- 除了属性之外，不要实现公共方法或其他函数成员。
- 为了更安全，把特性声明为sealed。
- 特性中使用AttributeUsage显式指定特性可应用的目标。

```csharp
[AttributeUsage(AttributeTarget.Class,Inherited = false, AllwoMultiple = false) ]
public sealed class MyAttributeAttribute:System.Attribute
{
	public string Des{get;set;}
	public MyAttributeAttribute(string des)
	{
	 Des = des
	}
}
```

### 访问特性

通过Type的两个方法`Isdefined()`和`Getcustomattributesisdefined()`用来检测某个特性是否应用到了某个类上。

**Isdefined**:

- 第一个参数接受需要检查的特性Type对象。
- 第二个参数是bool，指示是否搜索Myclss继承树查找这个特性。

**Getcustomattributesisdefined**：返回应用到结构的特性的数组。返回的时是object数组，必须将它强制转换为相应的特性类型。调用方法后，每个与目标相关联的特性的实例就会被创建。

例子：

```csharp
[AttributeUsage(AttributeTarget.Class,Inherited = false, AllwoMultiple = false) ]
public sealed class MyAttributeAttribute:System.Attribute
{
	public string Des{get;set;}
	public MyAttributeAttribute(string des)
	{
	 Des = des
	}
}

[MyAttribute("hello")]
class Myclass{}

class Program
{
    static void Main()
    {
        Type t = typeof(Myclass);
        object[] Attr = t.Getcustomattributesisdefined(false);
        foreach(Attribute a in Attr)
        {
            MyAttributeAttribute attr = a as MyAttributeAttribute;
            if(null != attr)
            {
                Console.WriteLine(attr.Des);
            }
        }
	}
}

```

