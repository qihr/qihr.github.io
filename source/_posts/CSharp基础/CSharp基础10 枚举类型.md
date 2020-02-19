---
title: CSharp基础10 枚举类型
tags: [CSharp]
categories: Program
date: 2019-12-17 08:55:00

---

Ienumerator接口

<!-- more -->

# 1.1 概述

数组能使用foreach语句是因为，数组提供了一个叫做枚举器的对象。枚举器实现了Getenumerator方法，可以依次返回请求的数组中的元素，枚举器知道项的次序并跟踪他在序列中的位置。

获取一个枚举器的方法：调用对象的Getenumerator方法。

foreach要和可枚举类型一起使用，只要遍历对象是可枚举类型，它就会执行以下行为：

- 调用Getenumerator获取对象枚举器。
- 从枚举器中请求每一项并且把它作为迭代变量，代码可以读取变量但不可以改变。

# 1.2 Ienumerator接口

实现了ienumerator接口的枚举器包含了3个函数成员：`current()`, `movenext()` ,`reset()`。

- Current 返回序列中当前位置项的属性，它是只读属性，它返回object类型的引用，所以可以返回任何类型。
- Movenext把枚举器位置前进到集合中下一项的方法，它也返回布尔值，指示新的位置是有效的位置还是超过了序列的尾部。
- Reset是把位置重置为原始状态的方法。

编写foreach循环时，编译器会生成与下面类似的代码：

```csharp
static void Main()
{
	int[] array = {10,20,12,31};
	IEnumerator ie = array.GetEnumerator();
	
	while( ie.MoveNext())
	{	
		int i = (int) ie.Current;
		Console.WriteLine("{0}",i);
	}
}
```



# 1.3  Ienumerable 接口

和上面说的ienumerator是不一样的接口。

可枚举类是指实现了ienumerable接口的类，只有一个成员：Getenumerator方法，它返回对象的枚举器（实现了ienumerator接口）。

一个例子：

1.先实现一个枚举器，实现ienumerator接口：

```csharp
using System.Collection;

class ColorEnumerator:IEnumerator
{
    string[] colors;
    int position = -1;

    public ColorEnumerator(string[] thecolors)
    {
        colors = new string[thecolors.Length];
        for (int i = 0; i < thoColors.Length; i++)
        {
            colors[i] = thecolors[i];
        }
    }

    public object Current
    {
        get{
            if (position == -1)
            {
                throw new InvalidOperationException();
            }
            if (position >= colors.Length)
            {
                throw new InvalidOperationException();
            }
            return  colors[position];
        }
    }

    public bool MoveNext()
    {
        if (position < colors.Length - 1)
        {
            position ++ ;
            return true;
        }else
        {
            return false;
        }
    }

    public void Reset()
    {
        position = -1;
    }
}
```

2.类实现Ienumerable 接口：

```csharp
class  Spectrum:IEnumerable
{
    string[] Colors = {"1","2","3"};
    public IEnumerable GerEnumerator()
    {
        return new ColorEnumerator(Colors);
    }
}

class  Program
{
    static void Main()
    {
        Spectrum spectrum = new Spectrum();
        foreach (string color in spectrum)
        {
            Console.WriteLine(color);
        }
    }
}
//控制台输出：1，2，3

```

#  1.4 泛型枚举接口

大多数情况下应该使用泛型版本的枚举接口：

```csharp
Ienumerable<T>
Ienumerator<T> 
```

泛型和非泛型枚举接口的差异：

对于非泛型接口的形式：

- IEnumerable接口的GetEnumerator方法返回实现Ienumerator枚举器的示例；
- 实现了IEnumerable的类实现了Current属性，它返回object的引用，然后我们必须把它转换为实际类型的对象。

对于泛型接口形式：

- IEnumerable<T>接口的GetEnumerator方法返回实现IEnumator<T>的枚举器类的实例。
- 实现IEnumerator<T>的类实现了Current属性，他返回实际类型的对象，而不是object基类的引用。

为什么推荐使用泛型枚举接口：

- 非泛型接口的实现是不安全的，它们返回object类型的引用，然后必须转化为实际类型。
- 泛型接口的枚举器是类型安全的，它返回实际类型的引用，如果要创建自己的可枚举类，应该实现这些泛型接口。

# 1.5 迭代器

可以把手动写的，替换为迭代器生成的可枚举类型和枚举器。

## 1.5.1 迭代器块

迭代器块是有一个或多个yield语句的代码块。

 以下3种类型的代码块中的任意一种都可以是迭代器块：

1. 方法主体
2. 访问器主体
3. 运算符主体

两个版本的迭代器块：

```csharp
//版本1
public IEnumerator<string> BlackAndWhite()
{
    yield return "Black";
    yield return "gray";
    yield return "white";
}

//版本2
public IEnumerator<string> BlackAndWhite()
{
    string[] colors = {"Black","gray","white"};

    for (int i = 0; i < colors.Length; i++)
    {
        yield return thoColors[i];
    }
}
```



## 1.5.2 yield关键字

> 使用 `yield return` 语句可一次返回一个元素。
>
> 可通过使用 [foreach](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/foreach-in) 语句或 LINQ 查询来使用从迭代器方法返回的序列。 `foreach` 循环的每次迭代都会调用迭代器方法。 迭代器方法运行到 `yield return` 语句时，会返回一个 `expression`，并保留当前在代码中的位置。 下次调用迭代器函数时，将从该位置重新开始执行。
>
> 可以使用 `yield break` 语句来终止迭代。

## 1.5.3 使用迭代器创建枚举器

```csharp
class Myclass
{
    public IEnumerable<string> GerEnumerator()
    {
        return BlackAndWhite();
    }
	
	public IEnumerator<string> BlackAndWhite()
	{
    	yield return "Black";
    	yield return "gray";
    	yield return "white";
	}	
}

class  Program
{
    static void Main()
    {
        Myclass myclass = new Myclass();
        foreach (string color in myclass)
        {
            Console.WriteLine(color);
        }
    }
}
```

可以看到，比手动实现IEnumerator要方便一些。实际上编译器在`BlackAndWhite()`方法内部用一个嵌套类自动实现了枚举器,并且还生成了一个方法`BlackAndWhite()`。

## 1.5.4使用迭代器创建可枚举类

 这个例子中，`BlackAndWhite()`方法实现了一个可枚举类，在类中`GetEnumerator()`返回了这个枚举类。

此时编译器生成的类既是`IEnumerable`又是`IEnumerator`的，并且还生成了一个方法`BlackAndWhite()`。

```csharp
class Myclass
{
    public IEnumerator GetEnumerator()
    {
        IEnumerable<string> enumable = BlackAndWhite();
        return enumable.GetEnumerator();
    }
	
	public IEnumerable<string> BlackAndWhite()
	{
    	yield return "Black";
    	yield return "gray";
    	yield return "white";
	}	
}

class  Program
{
    static void Main()
    {
        Myclass myclass = new Myclass();
        
        //使用类对象迭代
        foreach (string color in myclass)
        {
            Console.WriteLine(color);
        }
        
        //使用类的枚举器方法。
        foreach (string color in myclass.BlackAndWhite())
        {
            Console.WriteLine(color);
        }
    }
}
```

## 1.5.5 常见迭代器模式

先说一下前两节的不同

1.5.3：`BlackAndWhite()`生成了一个`Enumerator`枚举器。

1.5.4：`BlackAndWhite()`生成了一个`Enumerable`枚举类，而枚举类中包含了`Enumerator`枚举器，类中还包含一个`GetEnumerator()`方法。

所以说1.5.4这种方法要更全面一些，实际上，这两节正是迭代器的常用法：

1. 枚举器的迭代器模式：

   ```csharp
   class Myclass
   {
       public IEnumerable<string> GerEnumerator()
       {
           return IteratorMethod();
       }
   	
   	public IEnumerator<string> IteratorMethod()
   	{
       	yield return ....;
   	}	
   }
   static void Main()
   {
      foreach (string color in myclass)
      {
         Console.WriteLine(color);
      }
   }
   ```

2. 可枚举类型的迭代器模式

   ```csharp
   class Myclass
   {
       public IEnumerable<string> GerEnumerator()
       {
           return IteratorMethod().GerEnumerator();
       }
   	
   	public IEnumerable<string> IteratorMethod()
   	{
       	yield return ....;
   	}	
   }
   static void Main()
   {
      foreach (string color in myclass)
      {
         Console.WriteLine(color);
      }
   }
   ```

   