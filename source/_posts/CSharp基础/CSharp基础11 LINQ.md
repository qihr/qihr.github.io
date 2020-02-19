---
title: CSharp基础11 LINQ
tags: [CSharp]
categories: Program
date: 2019-12-17 08:55:00


---

LINQ基础

<!-- more -->

# 1.1概述

LINQ发音为Link 语言集成查询的缩写（language interated query）

LINQ是.net框架的扩展，它允许我们以使用SQL查询数据库的方式查询数据集合。

LINQ可以从数据库，程序对象的集合以及XML文档中查询数据。

# 1.2 LINQ提供程序

每一种数据源类型，在其背后一定有根据该数据源类型实现LINQ查询的代码模块。这些模块叫做LINQ提供程序（provider）。

关于LINQ提供程序：

- 微软为常见的数据源类型提供了LINQ提供程序。
- 可以使用任何支持LINQ的语言来查询有LINQ提供程序的数据源类型。
- 第三方也会提供各种数据源类型的提供程序。

## 1.2.1 匿名类型

无名类类型的特性叫做匿名类型，常用于LINQ查询的结果之中。

匿名类型的变量使用和初始化对象一样的表达式（new关键字，类名。或构造方法以及对象初始化语句。），但没有类名和构造函数，且必须使用var声明的对象来赋值。

```csharp
var stu = new{name = "123",age = 1};
```

注意：

1. 匿名类型只能和局部变量配合使用，不能用于类成员。
2. 匿名类型没有名字，必须使用var关键字作为变量类型。
3. 不能设置匿名类型对象的属性，编译器为匿名对象创建的属性都是只读的。

除了对象初始化语句的赋值形式，匿名类型的对象初始化语句还可以使用简单标识符和成员访问表达式，这两种形式叫做投影初始化语句。如：

```csharp
string something = "";
//name 赋值形式，someclass.obj成员访问，something标识符
var stu = new{name = "123",someclass.obj，something};
Console.Write(stu.name,stu.obj,stu.somthing);
//等同于：
var stu = new{name = "123", obj = someclass.obj，something = something};


```

# 1.3 方法语法和查询语法

LINQ查询有两种形式的语法：查询语法和方法语法。在一个查询中可以组合这两种形式。

- 方法语法： 标准方法调用，使用一组标准查询运算符。是命令式的，指明了查询方法的调用的顺序。方法语法使用了lambda表达式。
- 查询语法：类似于SQL，使用查询表达式形式书写。是声明式的，查询描述了你需要返回的数据。但没有指明如何执行这个查询。 

官方推荐查询语法。但有些运算符必须使用方法语法书写。

```csharp
static void Main()
{
    int[] numbers = {1,2,3,4,5,6,};
    
    //查询语法
    var numsQuery = from n in numbers where n < 20  select n;
    
    //方法语法
    var numsMethod = numbers.where(x=>x < 20);
    
    //结合
    int numscount = (from n in numbers where n<20 select n).Count();
}
```

# 1.4  查询变量

LINQ查询可以返回两种类型的结果：可以是一个可枚举的一组数据，满足查询参数的项列表。也可以是一个叫做标量的单一值。它满足查询条件的结果的某种摘要形式。比如说`Count()`返回的就是标量。

```csharp
static void Main()
{
    int[] numbers = {1,2,3,4,5,6,};
    
    IEnumerable<int> lownums = from n in numbers where n < 20  select n;
    
    int numscount = (from n in numbers where n<20 select n).Count();
}
```

在执行上面的代码后，lowNums查询变量不会包含查询结果。编译器会创建能够执行这个查询的代码。

查询变量numcount包含的是真实的整数值，但只能通过真实运行查询后获得。

上图两个查询变量有很大的区别，主要是查询执行的时间：

-  如果查询表达式返回的是枚举（可枚举类型），查询会在处理枚举时才执行。
- 如果枚举被处理多次，查询就会执行多次。
- 如果在进行被查询的数据遍历后，查询执行之前数据有改动，则查询会使用新的数据。
- 如果查询表达式返回标量，查询立即执行，并会把结果保存在查询变量之中。

个人理解：使用时查询是因为，一旦被查询的集合有任何改变，那之前查询到的结果就会出现问题，甚至是异常。

# 1.5 查询表达式的结构

方法语法的结构：

- 子句必须按照一定顺序出现。
- from子句和select group子句是必须的。
- 其他子句可选。
- LINQ查询表达式中，select子句在表达式最后，c#中这么做的原因之一是让visual studio智能感应能在我们输入代码时给我们更多选项。
- 可以有任意多的from let where子句。

*这些东西组合的结构叫查询体。

## from子句

from指定作为数据源使用的数据集合。

from语句和foreach不同点如下：

- foreach语句命令式的指定了要从第一个到最后一个按顺序地访问集合中的项，而from子句声明式地规定规定每个项都要被访问，但是没有规定顺序。
- foreach语句遇到代码就执行主体，from语句什么也不执行，之创建可以查询的后台代码对象，只有程序访问查询变量语句时，才会执行查询。

## join子句

使用联结来结合两个或多个集合中的数据。联结操作接受两个集合然后创建一个临时的对象集合，每个对象包含原始集合对象中的所有字段。

```sql
join some in collection on field1 equal field2
```

例子：

```c#
var query = from s in students
			join c in studentsInCourses on s.StID equals c.StID
```

## let子句

let子句接受一个表达式的运算符，并且把它赋值给一个需要在其他运算中使用的标识符。

```c#
let a = Expression
```

例子：

```c#
static void Main()
{
    var groupA = new[]{3,4,5,6};
    var groupB = new[]{7,8,9,10};

    var someInts = from a in groupA from b in groupB
                   let sum = a + b
                   where sum == 12
                   select new {a,b,sum};

    foreach (var a in someInts)
    {
        Console.WriteLine(a);
    }

}
```

## Orderby语句

orderby子句的默认排序是升序。 可以使用asending和descending关键字显示的设置元素的排序为升序或降序。

例子：

```c#
var query = from s in students
			orderby s.age
    		select student
```

## Select Group语句

group by子句可选，用来指定选择的项如何被分组。

```c#
select Exp group collection by filed
```



## 查询的匿名类型

查询结果可以由原始集合的项，项和某些字段或匿名类型组成。

```c#
static void Main()
{
    var groupA = new[]
    {
        new {filed = "1",filed2 = "2"},
        new {filed = "1",filed2 = "2"}
    };

    var query = from s in groupA
                select new {s.filed};

    foreach (var query in query)
    {
        Console.WriteLine(a.filed);
    }
}
```

# 1.6标准查询运算符

标准查询运算符由一系列的API方法组成，能查询任何.net数组或集合。需要注意的点：

- 被查询的集合对象叫序列，必须实现Ienumerable接口。
- 标准查询运算符使用方法语法。
- 一些运算符返回Ienumerable对象，而其他的一些运算符返回标量。返回标量的运算符立即执行，并返回一个值。
- 有很多操作都是一个谓词作为参数，谓词是一个方法，以对象作为参数根据对象是否满足某个条件而返回true和false。

例子：

```c#
int[] numbers = new int[]{2,4,6};
int total = numbers.Sum();
int howmayn = numbers.Count();
```

## 1.6.1 标准查询运算符的签名

System.LINQ.enumerable类声明了标准查询运算符方法，他们也是扩展了Ienumerable<T> 泛型类的扩展方法。

直接调用扩展方法和将其作为扩展进行调用的不同如下面所示：

```c#
//注意下面的使用方法是完全等价的
var count = Enumerable.Count(array);
var count2 = array.Count;
```



# 1.7 查询表达式和标准运算符

标准查询运算符是进行查询的一组方法。

可以使用带有标准查询运算符的方法语法来编写查询表达式：

```c#
static void Main()
{
    var numbers = new int[] {2,3,4,5,6,7};

    int howmany = (from n in numbers 
                    where n < 7 
                    select n).Count();
    Console.WriteLine("Count:{0}",howMant);
}
```



## 1.7.1 将委托作为参数

泛型委托用于给运算符提供用户自定义的代码：

(count运算符被重载了，且有两种形式，第二种是带泛型委托的参数。）

第一种：

```c#
public static int Count<T>(this IEnumerable<T> source);
```

第二种：

```c#
public static int Count<T>(this IEnumerable<T> source,Func<T,bool> predicate);
```

如果传入的方法语句只有一条，比较简单，或者并不会在程序的其他地方调用，可以传入lambda表达式。











