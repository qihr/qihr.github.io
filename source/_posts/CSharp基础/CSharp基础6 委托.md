---
title: CSharp基础6 委托
tags: [CSharp]
categories: Program
date: 2020-02-19 23:52:36 
---

声明,创建,调用委托

<!-- more -->

## 1.1 概述

可以认为委托是持有一个或多个方法的对象。通过执行委托，执行它所持有的方法。其实委托就相当于类型安全的，面向对象的c++函数指针。也可以把委托看作是包含有序方法列表的对象，这些方法具有相同的签名和返回类型。

使用前，一般会创建一个委托类型。（注意一开始创建的是委托类型，不是委托对象。）

使用方法：

1. .声明一个委托类型，委托的声明和方法声明类似，但是没有实现块。
2. 使用委托类型声明一个委托变量。
3. 创建委托类型的对象，赋值给委托变量。
4. 为委托变量增加方法，方法的签名和返回类型和委托类型声明的一致。
5. 调用委托。

需要注意的点：

- 方法的列表成为调用列表。
- 需要保存的方法可以是任何类或者结构的，只要返回类型和签名（包括ref和out修饰符）一致。
- 列表中的方法也可以是静态的。

## 1.2 声明,创建,调用委托

声明委托对象：因为委托是类型的声明，所以他不需要在类内部声明。

```csharp
//关键字  返回类型 委托类型名 签名
delegate void    MyDel   (int x)
```

创建委托对象：

```csharp
delvar = new MyDel{someobj.MyMethod}; //创建委托并保存引用，传入对象的成员方法
dvar = new MyDel{class.Method}; //创建委托并保存引用，传入类的静态成员

//等价于
delvar = someobj.MyMethod;
dvar = class.Method;
```

使用委托对象：

- 通过赋值可以改变包含在委托变量中的引用，旧的委托对象会被垃圾回收。

- 委托是恒定的，委托对象被创建后就不能被改变。在使用+=时，实际上是创建了新的委托（里面是原委托的方法列表和+=号右边的方法）然后再复制给+=左边。-=用于删除方法，和+=一样，新的委托是旧的委托的副本删除时会从方法列表最后开始搜索，移除第一个与方法匹配的实例。

- 试图删除委托中不存在的方法没有效果。

- 调用空委托会抛出异常。可以和null做比较然后抛出。

- 如果一个方法在方法列表里多次，当委托被调用时，每有一个该方法就会调用一次。

- 调用列表最后一个方法的返回值就是委托调用返回的值，其他方法的值会被忽略。

- 如果委托有引用参数，在调用委托列表的下一个方法时，参数的新值（不是初始值）会传给下一个方法：

  ```csharp
  delegate void MyDel(ref int x);
  class Myclass
  {
      public void Add2(ref int x){x += 2;}
      public void Add3(ref int x){x += 3;}
      static void Main()
      {
          Myclass mc = new Myclass();
          MyDel mdel = mc.Add2;
          mdel += mc.Add3;
          mdel += mc.Add2;
          int x= 5;
          //输出：12
          mdel(ref x);
      }
  }
  ```



## 1.3 匿名方法

匿名方法允许不创建具名方法。以下地方可以使用匿名方法：

- 声明委托变量时作为初始化表达式。
- 组合委托时赋值语句的右边。
- 为委托增加事件时赋值语句的右边。

使用方法：

  ```csharp
  //关键字  参数列表 语句块
  delegate (parameters) {implementationcaode}
  
  //例子：
  delegate int OhterDel(int InParam)
  static void Main()
  {
      OhterDel del = delegate(int x)
      {
          return x + 20;
      }
  }
  
  ```

  匿名方法的参数必须在这几个方面和委托匹配：

  - 参数数量
  - 参数类型及位置
  - 修饰符

  可以省略或使圆括号为空来简化匿名方法，但必须是：委托的参数列表不包含任何out参数，且匿名方法不使用任何参数：

  ```csharp
  delegate int OhterDel(int InParam)
  static void Main()
  {
      OhterDel del = delegate
      {
          Method();
      }
  }
  
  ```

  如果委托中使用了params参数，匿名参数的列表将忽略params关键字：

  ```csharp
  delegate int OhterDel(int InParam,params int[] Y)
  static void Main()
  {
      OhterDel del = delegate(int x,int[] Y) //省略
      {
          Method();
      }
  }
  ```

  匿名方法的变量和参数作用域：

   

  被限制在实现方法的主体之内。

  与委托的具名方法不同，**匿名方法可以访问外围作用域的局部变量和环境。**

  外围作用域的变量叫外部变量，匿名方法使用外部变量称为方法捕获。

  ### 1.3.1 匿名方法的变量和参数作用域

  - 变量和参数作用域被限制在实现方法的主体之内。
  - 与委托的具名方法不同，匿名方法可以访问外围作用域的局部变量和环境。
  - 外围作用域的变量叫外部变量，匿名方法使用外部变量称为方法捕获。

  示例：

  ```csharp
  static void Main()
  {
      //一个块，x只在块内有效
      {
      	int x =5;
      	OhterDel del = delegate()
      	{
          	Console.WriteLine(x);//x可以在匿名方法作用域之内使用。
      	}
      }
      
      //只要捕获方法还是委托的一部分，即使外部变量离开了作用域，也一直有效：
      del();
  }
  ```

## 1.4 Lambda表达式

如果先引入了lambda方法，那就不会有匿名方法。匿名方法2.0引入的，lambda是3.0引入的。

lambda运算符也读作 goes to =>。

编译器可以省略一些东西：

  - 带有类型的参数列表称为显示类型。

  - 省略类型的参数列表被称为隐式类型。

  - 如果只有一个隐式类型参数（这个和隐式类型不一样的概念，意思是隐式类型中只有一个参数），可以省略圆括号。

lambda表达式要点：

    1. lambda表达式的参数列表中的参数数量类型位置必须和委托完全匹配。
    2. 表达式的参数列表中的参数不一定需要包含类型（隐式类型）。除非有ref和out，此时必须标注类型（显示类型。）
    3. 如果只有一个参数，且是隐式类型。圆括号可以省略否则必须有。
    4. 如果没有参数必须使用圆括号。

  例子：

  ```csharp
  Mydel del =  delegate(int x)   {return x + 1;};  //匿名方法
  Mydel del =         (int x) => {return x + 1;};  //Lambda表达式
  Mydel del = 		   	(x) =>  {return x + 1;};  //Lambda表达式
  Mydel del = 		   	  x =>  {return x + 1;};  //Lambda表达式
  Mydel del = 		   	  x =>   x + 1;  			//Lambda表达式
  ```

  







