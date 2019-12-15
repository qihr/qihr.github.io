---
title: UI框架笔记
tags: [Unity]
categories: Program
---

s

<!-- more -->

## 一 设计模式

### 1.工厂模式

所有对象的创造，都由同一个对象去创造

简单来说： 向工厂类或者方法传入一个参数，会得到一个new出来的对应对象。



### 2.存储讲解

一些知识点(因为教程中工厂创建类是根据 参数 == "类名" 来区分的)：

```csharp
string tmpa = "hello"; String tmpb = "hello";
```

刚刚记笔记的时候简写了直接赋值常量字符串，这样是不能观察出去别的，必须像下图一样new出来

```csharp
string tmpA = new string (new char[]){'H','L'});
string tmpB = new string (new char[]){'H','L'});
Debug.Log(tempA == tempB);
Debug.Log(tempA.Equals(tempB));
```

（两个都是true）

Equal 对比的是变量的地址是否相等 ==比较的是内容是否一样 除了string类型（string较特殊 字符串是共享的 equal对于string对比的是内容。）

```csharp
//zhan 1001
object tmpC = tempA;
//zhan 1002
object tmpD = tmpB;

Debug.Log("tmpc == tmpD")
Debug.Log(tmpc ==tempD);

Debug.Log(tempD.equals(tempC));
```

答案是 false true

因为虽然是tempd.equal 但是tempd实质上是tempb传过来的，并且equal被重载所以还是在比较string的内容

注意 == 号是可以被重载的：

```csharp
public static bool operator ==(Vecter Ihs ,Vecter rhs)
{
    if(exp)
    {
        return true;
    }else{
        return false;
    }
    
}
```

同时需要重载 != 不然会报错。



#### 参数传递

1. 按值传递值类型

2. 按值传递引用类型

3. 按引用传递值类型

4. 按引用传递引用类型

```csharp
public void Food(int x)
{
    x = 10;
}

//按值传递引用类型
public void Food2(ref int temp)
{
    tempX = 10;
}

public void Food3(Vecter tmpVect1)
{
    tempVect1.xx = 10;
}

public void Food4(Vecter tempVect2)
{
    Vecter tmpVect = new Vecter(10);
    tmpVect2 = tempVect;
}

public void Food5(ref Vecter tmpVect2)
{
    Vecter tmpVect = new Vecter(10);
    
    tmpVect2 = tmpVect;
}
```

需要说一下 ： Food4接受的参数是，参数里的对象在堆上所在的地址。而不是参数在栈上的地址。 自始至终 都是在做值传递。

food5加了ref 所以参数传了栈的值

### 3.策略模式

API传一个参数，得到不同的答案
一般做一些算法处理。
根据不同的参数做不同的处理。
多态的补充： 父类的指针 指向子类 再调用子类的方法。 eg： personnal（子类对象） person = new personstrage(子类构造方法)
也可以是 stragenase（父类对象） sb = new personstrage（子类构造方法）
override的方法 当子类以父类形式调用方法时，调用的时子类的override的方法。

### 4.观察者模式

观察者每隔一段时间去询问对象工作是否完成

缺点： 经常询问，效率比较低。

### 5.代理模式

对象完成工作后通知代理

优点：完成任务后主动通知，不浪费性能

能用代理模式的时候，不要使用观察者模式。

和观察者模式不一样的地方在于，虽然对象在两种模式下都是每一帧在工作（比如播放动画），但是代理模式下，调用者不需要去每帧检查。而观察者模式下，调用者需要每帧检查。



<img src="C:\Users\Sayi1\AppData\Roaming\Typora\typora-user-images\image-20191215161524583.png" alt="image-20191215161524583" style="zoom:50%;" />





在干活的人多的时候，代理模式去向调用层报告要比观察者模式好得多。

观察者模式和代理模式的优缺点。



### 6.单例模式

单例模式存在一些问题：

单例对象之间的交叉问题。代码混乱。

尽量少用单例，要分析场景是否适合。

### 7.门面模式

1. 管理的东西，没有关联的逻辑关系 

2. 但是把这些东西集合，可以形成一个功能。

<img src="C:\Users\Sayi1\AppData\Roaming\Typora\typora-user-images\image-20191215161620196.png" alt="image-20191215161620196" style="zoom:50%;" />

### 8.建造模式

![image-20191215161644256](C:\Users\Sayi1\AppData\Roaming\Typora\typora-user-images\image-20191215161644256.png)



什么是assetbundle：比如说一个图片资源，和一个text文本资源，打成一个文件 test.u3d 这个u3d就是assetbundle。

案例中写了两个类  一个是从内存中读取assetbundle：IABResuse。另一个是从硬盘中：AssetbundleLoad

上层调用->AssetbundleLoad到内存->Assetbundle.LoadAsset

自上而下的 每一个环节都提供不同的功能 分工明确。

也叫责任链模式



### 9.中介者模式

迪米特原则  最少的引用其他类

中介者模式就是这个原则。

一个面试题：

![image-20191215161738292](C:\Users\Sayi1\AppData\Roaming\Typora\typora-user-images\image-20191215161738292.png)



点乘 叉乘

![image-20191215161749804](C:\Users\Sayi1\AppData\Roaming\Typora\typora-user-images\image-20191215161749804.png)



先判断夹角是否小于九十度 小于证明 npc在前方。

然后判断x和y的投影。

![image-20191215161807283](C:\Users\Sayi1\AppData\Roaming\Typora\typora-user-images\image-20191215161807283.png)





## 零 问题

对象为什么都在堆中？

堆 栈 全局变量区 程序存储的三个区域 栈也有地址吗

为什么 tempc == tempd 是false，因为类没有定义==重载吗？















