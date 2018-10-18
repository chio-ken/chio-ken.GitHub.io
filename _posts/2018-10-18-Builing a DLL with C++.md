---
layout:       post
title:        "如何用C++构建DLL文件 「译」"
subtitle:     "C++, 中间件, 翻译，DLL开发"
date:         2018-10-18
author:       "Chou"
header-img:   "img/post-bg-building-a-dll.jpeg"
header-mask:  0.3
catalog:      true
multilingual: false
tags:
    - 中间件开发
    - 英语译文

---

最近在写一个用于网络间通信的中间件，简单来说就是将处理过程用C++写成DLL文件，再由客户端调用该DLL实现一些简单的通信与处理数据的功能。其中用到了富士通开发的基于CORBA的interstage web server，这里不赘述。在查找关于如何创建DLL文件时，发现中文资料相对较少，这里选了一篇我认为很简洁明了的教程，简单翻译一下。

***以下为译文***

### 概述

Microsoft Visual C++ （MSVC）这个IDE，一些初学者用起来会比较棘手，这篇文章就是为了帮助那些想要编译DLL文件以供LabVIEW使用的人。

`注意`：此文档的IDE为MSVC 2010



### 第一步：新建DLL工程

选择```文件```»```新建工程```打开```新建工程对话框```。从模板列表里选择```Win32 Project```，给新工程命名之后，点击```OK```。

![java-javascript](/img/in-post/building-a-dll-with-visual-c++/newproject.png)

在下一个对话框中可以看到这个工程被设置为```Windows Application```，点击```下一步```把应用类型更改为```DLL```。

![java-javascript](/img/in-post/building-a-dll-with-visual-c++/win32applicationwizard.png)

![java-javascript](/img/in-post/building-a-dll-with-visual-c++/easydllfinished.png)

MSVC创建了一个带有一个源文件（.cpp）的DLL工程，该文件与工程的名字相同。同时生成的还有```stdafx.cpp```文件，这是个必不可少的文件，不过由于是自动生成好的，你也不必手动去编辑它。



### 第二步：编辑源文件

每个DLL文件都必须具有DllMain方法，该方法是库的入口点。 除非你必须对库进行特定初始化，否则MSVC创建的默认DllMain就足够了。 请注意，此函数不执行任何操作。

```c++
BOOL APIENTRY DllMain( HANDLE hModule,
                        DWORD  ul_reason_for_call,
                        LPVOID lpReserved )
{
    return TRUE;
}
```

如果要对库进行初始化，就要完善这个DLLMain方法。

```c++
BOOL WINAPI DllMain(  
         HINSTANCEhinstDLL,  // handle to DLL module
         DWORD fdwReason,     // reason for calling function
         LPVOID lpReserved )  // reserved
{
    // Perform actions based on the reason for calling.
    switch( fdwReason )
    {
    case DLL_PROCESS_ATTACH:
        // Initialize once for each new process.
        // Return FALSE to fail DLL load.            
        break;

    case DLL_THREAD_ATTACH:        
        // Do thread-specific initialization.
        break;        
   
    case DLL_THREAD_DETACH:
        // Do thread-specific cleanup.            
        break;
   
    case DLL_PROCESS_DETACH:        
        // Perform any necessary cleanup.
        break;    
    }
        return TRUE;
}

```

DllMain方法完善之后就需要写其他的文件了。

```C++
//Function declarations
int GetSphereSAandVol(double radius, double* sa, double* vol);
double GetSA(double radius);
double GetVol(double radius);
...
int GetSphereSAandVol(double radius, double* sa, double* vol)
//Calculate the surface area and volume of a sphere with given radius
{
	if(radius < 0)
		return false; //return false (0) if radius is negative
			*sa = GetSA(radius);
			*vol = GetVol(radius);
			return true;
}
double GetSA(double radius)
{
	return 4 * M_PI * radius * radius;
}
double GetVol(double radius)
{
	return 4.0/3.0 * M_PI * pow(radius, 3.0);
}
```

为了使DLL能正确编译，必须要声明``pow``函数（pow(x, y)等价于x^y）和常量`M_PI`(3.14159)。

声明的方法如下，在`.cpp`文件的顶部插入以下代码：

```C++
include "stdafx.h"
#include "math.h"    //library that defines the pow function
#define M_PI 3.14159 //declare our M_PI constant
```

此时就可以编译并且生成DLL了，但是由于没有导出任何方法，并不能被外部调用，所以也没什么用处。



### 第三步：导出符号

为了外部可以访问DLL内部的方法，必须要告知编译器去导出固定的符号。但是首先要解决C++的名字修饰（译者注：又称名字重整）问题。

（`译者注`：为了便于C编译器编译成目标代码，必须对相同的函数名进行修饰。

例如把

```C++
int  f (void) { return 1; }
int  f (int)  { return 0; }
```

修饰为

```C++
int  __f_v (void) { return 1; }
int  __f_i (int)  { return 0; }
```

）

扩展名为```.cpp```或者```.cxx```的文件会被当做C++文件进行编译，扩展名为```.c```就会被当做C文件编译。如果被当做C++文件编译，在输出代码中函数名就会被**修饰**，由于修饰后函数名被添加了额外的字符，就可能会产生一些问题。为了避免这个问题，就统一将函数用```'extern C'```修饰，如下：

```C++
extern "C" int GetSphereSAandVol(double radius, double* sa, double* vol);
```

这样就防止编译器对C++函数名字进行修饰（重整）。

警告⚠️：不对C++函数名进行修饰，就无法实现函数的多态了。

当完成C++装饰之后，就可以导出这些函数了。有两个方法通知[链接器](https://zh.wikipedia.org/wiki/%E9%93%BE%E6%8E%A5%E5%99%A8)去导出哪些函数。

第一种，也是最简单的一种，就是用```__declspec(dllexport)```标签来标记需要被导出的函数。将该标签插入函数的声明和定义部分，如下：

```C++
// Function declarations
extern "C" __declspec(dllexport) int GetSphereSAandVol(double radius, double* sa, double* vol);
...
// Function definition
__declspec(dllexport) int GetSphereSAandVol(double radius, double* sa, double* vol)
{
     ...
}
```

第二种就是用一个```.def```文件明确声明导出哪个函数。这个文件包含了链接器决定导出哪些内容的信息。格式如下：

```C++
LIBRARY   <Name to use inside DLL>
DESCRIPTION "<Description>"
EXPORTS
    <First export>   @1
    <Second export>  @2
    <Third export>   @3
    ...

```

对于本文的DLL，对应的```.def```文件如下：

```C++
For the example DLL, the .def file will look like this:

LIBRARY   EasyDLL
DESCRIPTION "Does some sphere stuff."
EXPORTS  
    GetSphereSAandVol   @1
```

如果你建的DLL工程没有问题的话，则链接器会自动查找与项目目录中项目同名的```.def```文件。选择```工程```»```属性```，在`Linker`文件夹下，点击`Input`属性页，可以将模块定义文件（Module Definition File）属性改为```/DEF:<filename.def>```。

![java-javascript](/img/in-post/building-a-dll-with-visual-c++/easydllpropertypages.png)



### 第四步：明确调用约定

在构建DLL之前，你需要做的最后一件事情就是明确你想要导出的函数的调用约定。通常有两个选择：C调用约定或者标准调用约定，也叫Pascal和WINAPI。大多数的的DLL函数使用标准调用，但是LabVIEW也可以调用。

如果使用C调用约定，你什么都不需要做。除非你在```Project```»```Properties```»```C / C ++```»``Advanced``中另行指定。如果指定为C调用，在函数的声明和定义中使用```__cdecl```关键字就可以了。

```C++
// Function declarations
extern "C" __declspec(dllexport) int __cdecl GetSphereSAandVol(double radius, double* sa, double* vol);
...

// Function ddfinitions
__declspec(dllexport) int __cdecl GetSphereSAandVol(doublt radius, double* sa, double* vol)
{
     ...
}
```

如果使用标准调用，把```__cdecl```换成```__stdcall```就可以了。

```C++
// Function declarations
extern "C" int __stdcall GetSphereSAandVol(double radius, double* sa, double* vol);
...

// Function ddfinitions
int __stdcall GetSphereSAandVol(doublt radius, double* sa, double* vol)
{
     ...
}
```

使用标准调用约定时，函数名会在DLL中进行修饰。 你可以通过使用导出函数的.def文件方法而不是__declspec（dllexport）方法来避免这种情况。 因此，National Instrument建议你使用.def文件方法导出stdcall函数。



### 第五步：构建DLL

当完成以上编码之后，声明了导出的函数，设定了调用约定，就可以构建DLL了。选择你的工程进行编译，链接DLL。现在就可以在LabVIEW中使用或者debug你的DLL文件了。附件```EasyDLL.zip```包括了用于创建这个DLL的工作空间和访问DLL的LabVIEW VI。（`译者注`：附件请点击最后的原文链接进行下载）





> 著作权声明

本文译自 [Building a DLL with Visual C++](http://www.ni.com/white-paper/3056/en/)
译者 [张健](http://chioken.com/about/)，首次发布于 [Chou Blog](http://chioken.com/)，转载请保留以上链接