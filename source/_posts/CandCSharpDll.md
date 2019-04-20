---
title: Unity下C#调用纯C动态链接库
tags: [CSharp, Unity, C,Android]
categories: Program
---
链接库可以解决在项目中需要调用其他语言代码（如C/C++）的需求。库中保存了其他程序需要调用的方法。

注：C++下导出链接库和使用方法和C有区别

<!-- more -->

# 一 概述

## 1.1 动态与静态链接库

链接库分**动态链接库**（Dynamic-link library，简写DLL）以及**静态链接库**（Statically-linked library）。

### 区别

**动态链接库：**在程序编译时，动态库中的函数代码不复制到可执行文件中，待程序需要调用库中函数时再进行加载。

**静态链接库：程序编译时使用到的静态库会被打包到可执行文件中。**

**优势**：

动态库只有在需要时才会加载，所以比静态库要节省内存，可扩展性强，同时减少了开发中的耦合度。

而静态库由于存在于执行文件中，所以代码的装载，执行速度要比动态库快。

**劣势：**

动态库的版本一旦出现异常，冲突，会引发DLL HELL问题。并且，动态库虽然节省内存，但调用方法的开销大，需要多次的间接访问才能调用到方法。

静态库生成的可执行文件体积较大，且代码无法共享。

## 1.2 不同系统下的链接库格式

**Windows**：.dll（动态库），.lib（静态库）

**Linux（以及基于Linux的Android）**：.so(动态库，share object)，.a（静态库）

**Mac：**.a、.framework（静态库），.tbd，.dylib，.framework（动态库）

*注：在Mac（iOS）系统下包含动态库的应用程序**无法在Apple Store通过审核上架**。详情：IOS静态库和动态库*

## 1.3 P/Invoke

> P/Invoke又名平台调用，是.NET CLR提供的，为了使开发者从托管代码(如题主的C#)调用动态连接库中的非托管代码（通常是C）而提供的一种服务。
>
> 在受控代码与非受控代码进行交互时会产生一个事务（transition） ，这通常发生在使用平台调用服务（Platform Invocation Services），即P/Invoke

简单来说，就是P/Invoke为C#提供了调用其他语言代码的服务。

## 1.4 MinGW和Cygwin

在生成Android使用的链接库时，需要使用Linux的编译工具，若想在Windows环境下使用Linux的编译工具（gcc/g++），则需要一个平台进行转换，也就是MinGW或Cygwin。

MinGW（ Minimalistic GNU for Windows），可被视为是Windows版本下的GCC，把代码中的LInux方式调用替换为对应的Windows调用方式。其中Msys子项目提供了一些模拟Linux的Shell和基础工具。

Cygwin 是在Windows平台上运行的unix模拟环境，Cygwin更像一个平台，模拟了Linux的接口，提供了运行在它上面的程序使用。

> - MinGW生成的程序，究其本质**调用的是Kernel.32导出的标准Windows系统API**，在windows下Mingw的编译性能会高一些，编译速度也会快一些。
> - Cygwin更像一个平台，它**相对完整地模拟了LInux**，提供了一个接近2M的Cygwin1.dll的文件作为目标库，来模拟Linux系统的接口，但是相对来说编译的速度就要慢一些。如果想要在Windows上开发可以运行在LInux上的程序，应该选用Cygwin。

资料:https://www.cnblogs.com/zoe-mine/p/7056369.html

MinGW和Cygwin的安装请自行查看资料

# 二 Windows下的链接库调用

## 2.0 准备环境

VS2015，[Dependency Walker](http://www.dependencywalker.com/)（用于查看DLL中是否有写好的方法）

## 2.1 DLL项目

### 2.1.1建立DLL项目

[![Image 0011554868057.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011554868057.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011554868057.png)

如果没有Win32，则需要下载模板

[![Image 0011554869322.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011554869322.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011554869322.png)

### 2.1.2文件介绍

新建项目完成后，会有如下几个文件：

[![Image 0011554869544.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011554869544.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011554869544.png)

- （1）stdafx.h：用于包含标准系统包含的头文件，用户需要的其他标准头文件也写在这里。

- （2）stdafx.cpp：和stdafx.h对应，用于对stdafx.h进行预编译处理所谓头文件预编译，把一个工程(Project)中使用的一些MFC标准头文件预先编译，以后该工程编译时，不再编译这部分头文件，仅仅使用预编译的结果。这样可以加快编译速度，节省时间,编译结果文件是projectname.pch,stdafx.cpp可以在C++项目中加快编译速度。如果不使用MFC可以将其删除。

- （3） targetver.h：定义dll最高可以使用的windows版本

- （4） **dllmain.cpp：**dll的程序入口点。

- [![Image 0011555668421.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555668421.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555668421.png)

- | DLL_PROCESS_ATTACH | 进程被调用，DLL被连接到当前进程并被初始化   |
  | ------------------ | ------------------------------------------- |
  | DLL_THREAD_ATTACH  | 当前进程创建一个新线程，DLL在新线程内被调用 |
  | DLL_PROCESS_DETACH | 调用DLL的进程被终止，DLL被卸载              |
  | DLL_THREAD_DETACH  | 调用DLL的线程被终止，DLL被卸载              |

### 2.1.3 将DLL项目修改为C

此时项目需要做一些操作，才能修改为纯C的DLL库。

（1）.将预编译头修改为：创建**(/YC)**

[![Image 0011554870532.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011554870532.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011554870532.png)

（2）.修改为：编译为C代码**（/TC）**[![Image 0011554870620.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011554870620.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011554870620.png)

（3）.删除stdafx.cpp，将其他cpp文件后缀修改为.c。修改完如图（Ccallback，CFunction，Cstruct是我写好的.c，可忽略）：

[![Image 0011555056920.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555056920.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555056920.png)

### 2.1.4 编写C代码

（1）编写方法，命名为CFunction.c

```c
// CFunction.cpp : 定义 DLL 应用程序的导出函数。
//

#include "stdafx.h"

__declspec(dllexport) int NoArgReturnSth()
{
  printf("NoArgReturnSth()调用:\n");
  return 99999;
}

__declspec(dllexport) int Sum(int a , int b)
{
  printf("Sum(int a , int b)调用:\n");
  return a + b;
}

__declspec(dllexport) int SetNumZero(int* a)
{
  printf("int SetNumZero(int *a)调用：\n");
  printf("int SetNumZero(int *a)调用,int a的内存位置：%d\n",a);
  *a = 0;
  return 1;
}

 __declspec(dllexport) int sendMessage(char* msg)
{
   printf("int sendMessage(char* msg)调用：\n");
   int i = 0;
   for(;*(msg + i) != NULL;i++ )
   { 
    *(msg + i) = *(msg + i) - 32;
   }
   return 1;
}
```

（2）编写方法，命名为CStruct.c

```c
#include "stdafx.h"

typedef struct
{
  int osVersion;
  int majorVersion;
  int minorVersion;
  int buildNum;
  int platFormId;
  char szVersion[128];
}OSINFO;

//（传递结构体指针）
__declspec(dllexport) int SetVersionPtr(OSINFO *info)
{
  char * str = "Hello DLL";
  info->osVersion = 100;
  printf("测试内容%d\n", info->majorVersion);
  strcpy(info->szVersion, str);
  return 1;
}
```

（3）编写方法，命名为Ccallback.c

```c
#include "stdafx.h"


typedef int(*pfCallBack)(int a, char b[]);

pfCallBack CallBackfun = NULL;

char ch[100];


__declspec(dllexport) void SetCB(int(*pfCallBack)(int a, char b[]))
{
  CallBackfun = pfCallBack;
}

 __declspec(dllexport) void callTheCB()
{
  int a = 4;
  memset(ch, '\0', 100);
  strcpy(ch, "aabbcc");
  CallBackfun(a, ch);
}
```

注：printf方法需要在stdafx.h文件中添加#include "stdio.h"。

### 2.1.5 导出DLL

（1）编译，编译成功后在项目的Debug目录下会有如下几个文件，（我的项目名字叫CFunctionw，所以生成的叫CFunction）：

[![Image 0011555058072.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555058072.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555058072.png)

（2）检查DLL是否存在需调用的方法，下载[Dependency Walker](http://www.dependencywalker.com/)，安装后打开CFunction.dll：

[![Image 0011555058152.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555058152.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555058152.png)

可以看到有写好的方法，至此，DLL编写完成。

## 2.2 C#调用DLL库（简单的控制台Demo）

------

（1）新建一个C#控制台项目。

（2）在Main方法中调用DLL：

```c++
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using System.Runtime.InteropServices;

namespace ConsoleApplication1
{
    [StructLayout(LayoutKind.Sequential)]
    public struct OSINFO
    {
        public int osVersion;
        public int majorVersion;
        public int minorVersion;
        public int buildNum;
        public int platFormId;
        [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 128)]
        public string szVersion;
    }

    class Program
    {
        //定义一个委托，其返回类型和形参与方法体的返回类型形参一致
        //一定要加上这句，要不然C#中的回调函数只要被调用一次，程序就异常退出了
        [UnmanagedFunctionPointer(CallingConvention.Cdecl)]  

        public delegate int callBackHandler(int a, [MarshalAs(UnmanagedType.LPStr)]StringBuilder b);

        internal static void Main(string[] args)
        {
            int a = 1;
            int b = 2;
            Console.WriteLine("------------------------------");
            Console.WriteLine("DLL func execute:{0}", NoArgReturnSth());
            Console.WriteLine("------------------------------");
            Console.WriteLine("DLL func execute:{0}", SUM(a,b));
            Console.WriteLine("------------------------------");
            Console.WriteLine("修改前的a:{0}", a);
            Console.WriteLine("DLL func execute:{0}", SetNumZero(ref a));
            Console.WriteLine("修改后的a:{0}", a);
            Console.WriteLine("------------------------------");
            StringBuilder buf = new StringBuilder(2048);
            buf.Append("helloworld");
            Console.WriteLine("传入字符串:{0}", buf);
            sendMessage(buf);
            Console.WriteLine("传出字符串:{0}", buf.ToString());
            Console.WriteLine("------------------------------");
            initStruct();
            Console.WriteLine("------------------------------");
            callBackHandler fun = new callBackHandler(localFun);
            SetCB(fun);
            callTheCB();
            Console.WriteLine("------------------------------");
            Console.Read();
        }
        //普通无参方法
        [DllImport("CFunction.dll")]
        private static extern int NoArgReturnSth();

        //有参int方法
        [DllImport("CFunction.dll", EntryPoint = "Sum", CallingConvention = CallingConvention.Cdecl)]
        public static extern  int SUM(int a, int b);

        //传入int指针方法
        [DllImport("CFunction.dll", EntryPoint = "SetNumZero", CallingConvention = CallingConvention.Cdecl)]
        public static extern int SetNumZero(ref int a);

        //传入字符串指针方法
        [DllImport("CFunction.dll", CallingConvention = CallingConvention.Cdecl)]
        public static extern int sendMessage(StringBuilder msg);
        //传入结构体方法
        [DllImport("CFunction.dll", EntryPoint = "SetVersionPtr", CallingConvention = CallingConvention.Cdecl)]
        public static extern int SetVersionPtr(ref OSINFO info);
        //结构体初始化，并调用DLL方法。
        public static void initStruct()
        {
            IntPtr pv = Marshal.AllocHGlobal(Marshal.SizeOf(typeof(OSINFO)));
            for (int i = 0; i < 5; i++)
            {
                Marshal.WriteInt32(pv, i * Marshal.SizeOf(typeof(Int32)), ((Int32)(i + 1)));
            }

            OSINFO entries = (OSINFO)Marshal.PtrToStructure(pv, typeof(OSINFO));
            entries.majorVersion = 999;
            if (SetVersionPtr(ref entries) == 1)
            {
                Console.WriteLine("--osVersion:{0}", entries.osVersion);
                Console.WriteLine("--Major:{0}", entries.majorVersion);
                Console.WriteLine("--Minor:{0}", entries.minorVersion);
                Console.WriteLine("--BuildNum:{0}", entries.buildNum);
                Console.WriteLine("--szVersion:{0}", entries.szVersion);
            }
            Marshal.FreeHGlobal(pv);
        }

       //传入委托方法
        [DllImport("CFunction.dll", CallingConvention = CallingConvention.Cdecl)]
        public static extern void SetCB(callBackHandler fun1);
        //回调方法
        [DllImport("CFunction.dll", CallingConvention = CallingConvention.Cdecl)]
        public static extern void callTheCB();
         //本地委托方法
        public static int localFun(int a, [MarshalAs(UnmanagedType.LPStr)] StringBuilder b)
        {
            Console.WriteLine("回调:{0}", b.ToString());
            return 0;
        }

    }
}
```

**CallingConvention = CallingConvention.Cdecl**

首先引入System.Runtime.InteropServices，然后导入DLL，格式为：

```csharp
[DllImport("dllname.dll"),EntryPoint = "functionname",CallingConvention = CallingConvention.Cdecl,]
private static extern int functionname2();
```

[DllImport("CFunction.dll")]用于导入DLL。EntryPoint 、CallingConvention 是两个较常用的属性，EntryPoint 用于指定DLL中的方法名，如果DLL和C#中声明的方法名相同，则该属性可忽略。CallingConvention用于指定由哪一方负责处理堆栈，值Cdecl为调用方清理堆栈。

如果不使用CallingConvention，会提示：*“C#调用C++DLL文件 报错调用导致堆栈不对称。原因可能是托管的 PInvoke 签名与非托管的目标签名不匹配”*的错误。

```c++
[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
```


UnmanagedFunctionPointe表示动态使用未托管的dll函数指针

> CallingConvention.Cdecl：C调用约定（即用__cdecl关键字说明）按从右至左的顺序压参数入栈，由调用者把参数弹出栈。对于传送参数的内存栈是由调用者来维护的（正因为如此，实现可变参数的函数只能使用该调用约定）。另外，在函数名修饰约定方面也有所不同。_cdecl是C和C＋＋程序的缺省调用方式。每一个调用它的函数都包含清空堆栈的代码，所以产生的可执行文件大小会比调用_stdcall函数的大。函数采用从右到左的压栈方式。VC将函数编译后会在函数名前面加上下划线前缀，是MFC缺省调用约定；
> CallingConvention.StdDecl：__stdcall调用约定相当于16位动态库中经常使用的PASCAL调用约定。在32位的VC++5.0中PASCAL调用约定不再被支持（实际上它已被定义为__stdcall。除了__pascal外，__fortran和__syscall也不被支持），取而代之的是__stdcall调用约定。两者实质上是一致的，即函数的参数自右向左通过栈传递，被调用的函数在返回前清理传送参数的内存栈，但不同的是函数名的修饰部分（关于函数名的修饰部分在后面将详细说明）。_stdcall是Pascal程序的缺省调用方式，通常用于Win32Api中，函数采用从右到左的压栈方式，自己在退出时清空堆栈。VC将函数编译后会在函数名前面加上下划线前缀，在函数名后加上"@"和参数的字节数；

还有一些其他的属性，详情看：[平台调用P-INVOKE(一)--(基础篇)](https://blog.csdn.net/sdl2005lyx/article/details/6796037)

注意C#和DLL中的方法的返回值，参数个数、类型、顺序必须一致。方法名可以不一致。

（3）运行代码，控制台会输出对应方法及其返回值：

[![Image 0011555059881.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555059881.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555059881.png)

注：将DLL和用于调试的控制的控制台项目放入同一解决方案内，修改DLL的生成路径可以简化调试过程，避免每次生成DLL在拷贝到Debug目录中。

## 2.3在Unity下使用DLL

------

（1）将DLL编译为64位，不然在Unity中会有关于32/64位的问题。为了测试方便，统一调整为64位。

[![Image 0011555061641.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555061641.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555061641.png)

（2）新建Unity项目，在Assets目录下新建Plugins文件夹，在Plugins文件夹下建立x86，x86_64文件夹。如图（忽略Android）：

[![Image 0011555061782.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555061782.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555061782.png)

之所以新建x86，x86_64文件夹是为了区分32/64位的DLL，直接拖到Plugins也可以。Plugins文件夹中若存在这两个文件夹，则dll必须放到两个文件夹中，否则会出现找不到dll的情况。（我的2017.4.16f1不存在这种情况。）

（3）修改DLL的属性，将属性修改为和图片一致

[![Image 0011555062363.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555062363.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555062363.png)

（4）编写C#脚本：

```c++
using System.Collections.Generic;
using UnityEngine;
using System.Runtime.InteropServices;
public class NewBehaviourScript : MonoBehaviour {
    [DllImport("CFunction")]
    private static extern int NoArgReturnSth();
    int i = NoArgReturnSth();

    void OnGUI()

    {
        GUI.Button(new Rect(1, 1, 200, 100), i.ToString());

    }
}
```

csharpcsharp随便拖到一个GameObject上，运行：

[![Image 0011555062503.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555062503.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555062503.png)

### 异常信息

此报错只针对Windows，后面会有其他平台的异常信息。

（1）DllNotFoundException: 64位运行32位的DLL会报这个错。

（2）Failed to load 'Assets/Plugins/xxx.dll', expected x64 architecture, but was x86 architecture. You must recompile your plugin for x64 architecture.：DLL文件没有选择对应的CPU

（3）EntryPointNotFoundException：EntryPoint没写的情况下就是C#方法名字和DLL名字对不上，写了就是EntryPoint没写对。

# 三 Android下的链接库调用

下面介绍三种我用的，在windows下生成.so的方法。

## 3.1 通过NDK生成.so文件

### 3.1.1 准备环境

（1）配置好NDK，SDK，JDK以及环境变量。

（2）新建一个文件夹，将写好的.c，.h放入。新建Android.mk，Application.mk文件。

[![Image 0011555065573.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555065573.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555065573.png)

*注：.c文件和.h文件的写法与DLL中稍有不同。*

文件介绍：

https://developer.android.com/ndk/guides/android_mk

**Android.mk：**Android提供的一种makefile文件，用于引导生成.so。

**Application.mk：**androidNDK构建系统使用的一个可选构建文件，用于描述native模块。

### 3.1.2 配置文件

（3）配置**Android.mk：**

```css
#指向一个编译脚本，由编译系统提供
include $(CLEAR_VARS)

#这个不太清楚 应该是处理器类型
LOCAL_ARM_MODE  := arm

#当前文件路径
LOCAL_PATH      := $(NDK_PROJECT_PATH)

#库名称
LOCAL_MODULE    := libnative

# 表示用于 C 编译器的选项，Werror将所有的警告当成错误进行处理
LOCAL_CFLAGS    := -Werror

#指定编译的源文件名称,
LOCAL_SRC_FILES := NativeCode.c CFunction.c

#允许打印Log
LOCAL_LDLIBS    := -llog

#告诉编译器生成.so
include $(BUILD_SHARED_LIBRARY)
```

（4）配置**Application.mk**

```css
#优化选项 可选release/debug
APP_OPTIM        := release

#选择需要支持的cpu
APP_ABI          := armeabi

#支持的最低Android平台
APP_PLATFORM     := android-16

#指定执行脚本
APP_BUILD_SCRIPT := Android.mk
```

注：如果提示APP_PLATFORM版本过低，对应调高即可。

### 3.1.3 编译

（5）打开命令行，cd到文件目录编译。

编译指令：`ndk-build NDK_PROJECT_PATH=. NDK_APPLICATION_MK=Application.mk`

[![Image 0011555068565.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555068565.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555068565.png)

编译后目录内会产生两个文件夹libs和obj，在libs的armeabi-v7a文件夹下会有我们编译好的libnative.so。

## 3.2 通过Cmake生成.so文件

------

### 3.2.1 准备环境

我是通过装Android Studio来安装的以下配置，但实际上编译用不到AS。

（1）在Android Studio安装好LLDB，CMake，NDK。（NDK如果之前装过可以自行配置），LLDB，CMake也可以在官网下载单独安装。

（2）打开settings查看SDK和NDK的所在位置：

[![Image 0011555295955.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555295955.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555295955.png)

打开位置，我的是C:\Users\usename\AppData\Local\Android\Sdk，然后找到如下文件夹：

[![Image 0011555296277.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555296277.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555296277.png)

**配置cmake和ninja的环境变量：**

[![Image 0011555296407.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555296407.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555296407.png)

**验证是否配置成功：**

[![Image 0011555296521.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555296521.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555296521.png)

如图就是配置成功了。

（3）查看**android.toolchain.cmake文件**：在ndk所在的文件夹下，build-cmake-android.toolchain.cmake。该文件很重要，如果cmake目录下没有此文件，就去https://developer.android.com/ndk/guides/cmake.html下载。

（4）准备代码：随便找个地方新建一个文件夹，新建build，include，src文件夹，将.c放入src，.h放入include：

[![Image 0011555297411.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555297411.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555297411.png)



（5）新建CMakeLists.txt，并输入如下内容：

```css
cmake_minimum_required(VERSION 3.6)
file(GLOB native_srcs "${CMAKE_SOURCE_DIR}/src/*.c")
include_directories(${CMAKE_SOURCE_DIR}/include)

#STATIC表示编译结果为静态库.a,如果想为动态库.so,可改为SHARED

add_library(Add SHARED ${native_srcs}) 
target_link_libraries(Add)
```

（6）在build文件夹下新建批处理文件build.bat，内容如下：

```shell
set toolchain=C:/Users/wenmai/AppData/Local/Android/Sdk/ndk-bundle/build/cmake/android.toolchain.cmake
set android_ndk=C:/Users/wenmai/AppData/Local/Android/Sdk/ndk-bundle
set build_type=Release
set gernerator="Ninja"
if not exist %1 md %1
cd %1
cmake ../.. -DCMAKE_TOOLCHAIN_FILE=%toolchain% -DANDROID_NDK=%android_ndk% -DCMAKE_BUILD_TYPE=%build_type% -DANDROID_ABI="%1" -DCMAKE_GENERATOR=%gernerator%
ninja
```

cmake ../..的意思是使用cmake交叉编译当前工程，并将-DCMAKE_GENERATOR指向了ninja.exe。%1运行.bat要传的参数,需要传Android ABI名称，如：arm64-v8a,armeabi armeabi-v7a mips,mips64,x86,x86_64。

（7）打开命令行，cd到build目录，输入：build.bat armeabi-v7a：

[![Image 0011555298228.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555298228.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555298228.png)

可以在 armeabi-v7a目录下看到生成了libAdd.so文件。

如果出现如下图所示的错误，可以先仔细检查配置文件是否写对，并且build.bat传入的参数是否正确：

[![Image 0011555153592.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555153592.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555153592.png)

## 3.3 使用Android Studio生成.so

*注：Android Studio在最近的版本中弃用了使用gcc编译器，改为使用Clang代替。*

下面利用AS和NDK来生成so，Cmake也可以使用此方法，文字不再做介绍。

### 3.3.1 准备环境

------

（1）安装好Android Studio，确保可以正常运行。

（2）[![Image 0011555137646.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555137646.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555137646.png)

界面位置：File-Settings-System Settings-Android-SDKTools

在AS中下载这些东西似乎需要翻墙，没有验证过。

文件介绍：

- CMake：一款外部构建工具，可与 Gradle 搭配使用来构建原生库。如果您只计划使用 ndk-build，则不需要此组件。
- LLDB：一种调试程序，Android Studio 使用它来调试原生代码。

### 3.3.2 项目配置

（1）先新建一个项目，（我的叫FuckSO）。然后切换工程目录为Project，（默认是Android），然后打开Project Structure-SDK Location，确认ndk已经配置好。

[![Honeycam 2019-04-13 15-11-35.gif](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Honeycam%202019-04-13%2015-11-35.gif)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Honeycam 2019-04-13 15-11-35.gif)

（2）在gradle.properties中添加一句：**android.useDeprecatedNdk=true**。为了向后兼容

（3）在build.gradle文件下添加ndk，CMake节点：

[![Image 0011555140513.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555140513.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555140513.png)

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 28
    defaultConfig {
        applicationId 'com.example.hellojni'
        minSdkVersion 23
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
    externalNativeBuild {
        cmake {
            version '3.10.2'
            path "src/main/cpp/CMakeLists.txt"
        }
    }

    flavorDimensions 'cpuArch'
    productFlavors {
        arm7 {
            dimension 'cpuArch'
            ndk {
                abiFilter 'armeabi-v7a'
            }
        }
        arm8 {
            dimension 'cpuArch'
            ndk {
                abiFilters 'arm64-v8a'
            }
        }
        x86 {
            dimension 'cpuArch'
            ndk {
                abiFilter 'x86'
            }
        }
        x86_64 {
            dimension 'cpuArch'
            ndk {
                abiFilter 'x86_64'
            }
        }
        universal {
            dimension 'cpuArch'
            // include all default ABIs. with NDK-r16,  it is:
            //   armeabi-v7a, arm64-v8a, x86, x86_64
        }
    }
}
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
}
```

（4）（如果不需要在Android Studio中调试此步骤可以跳过）在java目录下新建一个类，**CCodeHelper**。在静态代码中加载.c文件，并定义一个方法用于链接.c中的方法。

[![Image 0011555141665.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555141665.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555141665.png)

（5）在项目文件夹-app-src-main下新建cpp文件夹，并新建.c文件：

[![Image 0011555141480.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555141480.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555141480.png)

```
#include  <jni.h>
#include <string.h>
int  Java_com_example_fuckso_CCodeHelper_magicMethod()
{
#if defined(__arm__)
    #if defined(__ARM_ARCH_7A__)
    #if defined(__ARM_NEON__)
      #if defined(__ARM_PCS_VFP)
        #define ABI "armeabi-v7a/NEON (hard-float)"
      #else
        #define ABI "armeabi-v7a/NEON"
      #endif
    #else
      #if defined(__ARM_PCS_VFP)
        #define ABI "armeabi-v7a (hard-float)"
      #else
        #define ABI "armeabi-v7a"
      #endif
    #endif
  #else
   #define ABI "armeabi"
  #endif
#elif defined(__i386__)
#define ABI "x86"
#elif defined(__x86_64__)
#define ABI "x86_64"
#elif defined(__mips64)  /* mips64el-* toolchain defines __mips__ too */
#define ABI "mips64"
#elif defined(__mips__)
#define ABI "mips"
#elif defined(__aarch64__)
#define ABI "arm64-v8a"
#else
#define ABI "unknown"
#endif

    return 999999999;
}
```

注意方法的名字**Java_com_example_fuckso_CCodeHelper_magicMethod，**该方法与CCodeHelper中的方法对应。但多了Java_com_example_fuckso_CCodeHelper_，这是在Android项目中定义C/C++库方法的规则，规则为：“包名_类名_方法名”，在java类中直接使用方法名来进行调用。

但把so拿给unity使用时，必须要使用全名也就是**Java_com_example_fuckso_CCodeHelper_magicMethod**才能调用。

（如果不需要在Android Studio中调试，方法名随便写就可以。）

（6）再和build.gradle同级别目录新建CMakeLists.txt，写入如下内容：

```
cmake_minimum_required(VERSION 3.4.1)
add_library(
        CNativeFunction
         SHARED
        app/src/main/cpp/CNativeFunction.c
)
find_library(
          log-lib
          log
)
target_link_libraries(
                   CNativeFunction
                   ${log-lib}
 )
```

（7）ctrl+F9 编译。编译成功后，会在FuckSO\app\build\intermediates\cmake\debug\obj下对应的cpu类型下找到.so文件。

## 3.3 Unity调用.so

使用`nm -D libNativeCode.so` 来查看so中是否包含了写好的方法：

[![Image 0011555068534.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555068534.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555068534.png)

将.so拖入Assets-Plugins-Android文件夹。在属性面板修改库的相关设置：

[![Image 0011555136600.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555136600.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555136600.png)

然后新建C#脚本：

```
using UnityEngine;
using System.Collections;
using System.Runtime.InteropServices;

public class CallNativeCode : MonoBehaviour {

  [DllImport("native")]
  private static extern int NoArgReturnSth();
 
    int i = NoArgReturnSth();
    void OnGUI ()
  {
    float x = 3;
    float y = 10;
    //GUI.Label (new Rect (15, 125, 450, 100), "adding " + x  + " and " + y + " in native code equals " + add(x,y));
        GUI.Label(new Rect(15, 225, 450, 100), i.ToString());
    }
}
```

拖拽到任意物体，这是Unity会报错：DllNotFoundException: native，此时不用理会，因为.so文件无论如何也无法在windows环境下调用。所以需要打包成apk在模拟器上运行，发布设置按图片中的对应即可：

[![Image 0011555136869.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555136869.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555136869.png)

安装APK运行，这时可以看到c#从.so调用了方法并返回了一个值：

[![Image 0011555136994.png](http://img.dongbeigtl.top/A%E6%96%87%E7%AB%A0%E9%9C%80%E8%A6%81%E7%9A%84%E6%88%AA%E5%9B%BE/CSharp%E8%B0%83%E7%94%A8C%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93DLL/Image%200011555136994.png)](http://img.dongbeigtl.top/A文章需要的截图/CSharp调用C动态链接库DLL/Image 0011555136994.png)

**需要注意的几点：**

- [DllImport("filename")]，filename不要写lib前缀以及.so后缀，如例子中生成的.so叫libnative.so，DllImport引入时只需写native。（强制规定）
- 如果在模拟器中依然出现DllNotFoundException异常，会有几种情况： 1.发布APK时对应的.so库位数不一致；2.DllImport没有找到.so库；3.找到了.so库但是没有找到对应的方法。
- Win32Exception提示apksigner.bat找不到，我的SDK最高API是26，但是26的文件夹下没有apksigner.bat。临时的解决办法是删除26版本。（把文件夹删除）
- CommandInvokationFailure: Gradle build failed。解决办法：Unity编辑器 File->Build Setting->Android->Build System选择Internal

# 四 多平台适应

------

打包出Android在模拟器运行太麻烦，可以通过配置多平台的链接库实现平台自适应。首先准备好各平台的链接库，然后在

File->Build Settings-Switch Platform进行平台转换。

适配代码如下：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System.Runtime.InteropServices;

public class NewBehaviourScript : MonoBehaviour {
#if UNITY_STANDALONE_WIN
    const string fcuk = "CFunction";
#else
    const string fcuk = "native";
#endif

    [DllImport(fcuk)]
    private static extern int NoArgReturnSth();

    void OnGUI()
    {
        int i = NoArgReturnSth();
        GUI.Button(new Rect(1, 1, 200, 100), i.ToString());
    }
}
```

# 五 备注

文档中没有写关于如何生成静态库的方法，但其实和生成动态库是一样的只是参数不同。

生成Android可以使用的.so文件的方法不止文中写的这些，但需要注意的是有些方法生产的so库只能在Linux下使用，如利用VisualGDB和Linux虚拟机交叉编译生成的.so库（[使用Visual Studio创建Linux库](https://visualgdb.com/tutorials/linux/libraries/)），以及在windows环境下利用gcc编译出obj再转化为.so库的方法。

C++的链接库生成，和语法以及一些细微的差别在文中没有介绍。

.dll和.so的方法参数以及传参各有不同，具体区别下篇文档再做说明。可以先看看：

1. [C#调用C/C++ DLL 参数传递和回调函数的总结](https://yq.aliyun.com/articles/678154)
2. [C#调用C/C++动态库 封送结构体，结构体数组](https://www.cnblogs.com/stemon/p/4515522.html)

 

文中部分资料摘自：

- 微软官方资料：[创建和使用你自己动态链接库 （c + +）](https://docs.microsoft.com/zh-cn/cpp/build/walkthrough-creating-and-using-a-dynamic-link-library-cpp?view=vs-2019)
- C语言中文网：[ C中函数指针简介及其用法](http://c.biancheng.net/view/228.html)
- 阿里云社区：[C#调用C/C++ DLL 参数传递和回调函数的总结](https://yq.aliyun.com/articles/678154)
- [C#调用C/C++动态库 封送结构体，结构体数组](https://www.cnblogs.com/stemon/p/4515522.html)
- [C#调用C函数(DLL)传递参数问题](https://blog.csdn.net/imfengyitong/article/details/51549010)
- [平台调用P-INVOKE(一)--(基础篇)](https://blog.csdn.net/sdl2005lyx/article/details/6796037)
- [CallingConvention理解](https://www.cnblogs.com/cg88/p/9143866.html)
- [［科普小短文］在C#中调用C语言函数](https://blog.csdn.net/yapingxin/article/details/7288325)
- [Visual C++ 2010 Express Tips: 用 C 和 C++ 创建动态链接库](https://blog.csdn.net/yapingxin/article/details/7288164)
- [动态库和静态库的区别](https://blog.csdn.net/bad_boy627056049/article/details/73658073)
- [VS2017编写纯C库以及使用C#调用C库方法](https://blog.csdn.net/qq21497936/article/details/83825098)
- [VS2015_动态链接库学习](https://www.cnblogs.com/EltonLiang/p/7392410.html)
- [用VS2010将C程序做成动态链接库dll](https://blog.csdn.net/leo9150285/article/details/8804705)
- [VS2015设置DLL和LIB的输出目录](https://blog.csdn.net/zt_xcyk/article/details/78006223)
- Android中IDE、ADT、SDK、JDK、NDK的含义解释
- [ABI： abiFilters 详解](https://blog.csdn.net/afei__/article/details/81272251)
- [介绍linux/windows OS下静态库（.a、.lib）和动态库（.so、.dll）的 link & load](https://www.cnblogs.com/scotth/p/3977928.html)
-  

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 















































[有道词典](http://fanyi.youdao.com/translate?i=且Plugins文件夹中若存在这两个文件夹，则dll必须放到两个文件夹中，否则会出现找不到dll的情况。&keyfrom=chrome)

且Plugins ...

[详细](http://fanyi.youdao.com/translate?i=且Plugins文件夹中若存在这两个文件夹，则dll必须放到两个文件夹中，否则会出现找不到dll的情况。&smartresult=dict&keyfrom=chrome.extension)X

And if there are these two folders in the Plugins folder, the DLL must put in two folders, otherwise can't find the DLL would happen.







