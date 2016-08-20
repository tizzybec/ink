title: C++动态初始化（Dynamic Initializer）
date: 2015-10-04 21:43:00 +0800
update: 2015-10-04 21:43:00  +0800
author: me
#cover: -/images/default.png
tags:
    - C++

--- 

之前看公司有人收集类元信息用的方法很巧妙，所以早就想写一篇关于动态初始化的博客，一则是想分享，二则是作为个人摘记。

<!--more-->


在c++标准的initialization of non-local variable章节有如下描述

> There are two broad classes of named non-local variables: those with static storage duration (3.7.1) and  
those with thread storage duration (3.7.2). Non-local variables with static storage duration are initialized  
as a consequence of program initiation. Non-local variables with thread storage duration are initialized as a  
consequence of thread execution. Within each of these phases of initiation, initialization occurs as follows.  
> Together, zero-initialization and constant initialization are called static initialization ; all other initialization is   
dynamic initialization . Static initialization shall be performed before any dynamic initialization takes place. Dynamic initialization   
of a non-local variable with static storage duration is either ordered or unordered. Definitions of explicitly specialized class template  
static data members have ordered initialization.   
> It is implementation-defined whether the dynamic initialization of a non-local variable with static storage  
duration is done before the first statement of main. If the initialization is deferred to some point in time  
after the first statement of main, it shall occur before the first odr-use (3.2) of any function or variable  
defined in the same translation unit as the variable to be initialized.  

非局部静态变量的初始化是否发生在主函数main之前则以依赖于编译器的实现,在msdn上能看到如下文档

> C Run-Time Error R6030  
CRT not initialized  
This error occurs if you are using the CRT, but the CRT startup code was not executed.   
It is possible to get this error if the linker switch /ENTRY is used to override the   
default starting address, usually mainCRTStartup, wmainCRTStartup for a console   
EXE,WinMainCRTStartup or wWinMainCRTStartup for a Windows EXE, or _DllMainCRTStartupfor a DLL.  
Unless one of the above functions is called on startup, the C Runtime will not be initialized.  

比如我们使用vc100编译器，默认的入口函数wmainCRTStartup，当然项目配置或者链接器提供/ENTRY参数进行入口函数的指定，对于DLL又有所不同，执行的_DLLMainCRTStartup函数，一般来说这些函数做了如下事情

> including calling _CRT_INIT, which initializes the C/C++ run-time library and invokes C++ constructors on static, non-local variables  

所以对于VC系列编译器CRT提供了动态初始化的实现,在CRT Initialization文档部分,我们看到如下描述

> By default, the linker includes the CRT library, which provides its own startup code. This startup code initializes the CRT library,   
calls global initializers, and then calls the user-provided main function for console applications.  

并且提供一段示例代码

```
int func(void)  
{  
    return 3;  
}  
  
int gi = func();  
  
int main()  
{  
    return gi;  
}  
```

通过dumpbin /all your.obj可以看到所有放置在.CRT$XCA 和 .CRT$XCZ之间的.CRT$XCU内变量都会进行全局初始化。我们暂时不看dumpbin出来的具体内容，稍后我们会结合自己写的例子进行说明。

回到文章开头,为什么要用动态初始化来进行元数据的收集?我的理解至少有两点便利:

1.  在main函数或者在DLL全局函数执行之前所有类的元信息已经收集完毕，无论是对类进行序列化还是反序列化都有足够的信息

2.  通过一些trick我们可以不定义全部变量来进行元信息的收集，这样避免了声明过多的全局变量，同时整个注册部分完全隐藏了起来，通过宏的包装,完全可以简化元信息收集
下面我们来看一看具体的实现。

首先申明一个不完全类型的元信息收集器：

```
template<class Ty>  
struct MetaDataCollector;  
```

然后声明一个激活器来激活元信息收集器

```
template <class T>
class Activator {
 public:
  static T& Activate() {
    Magic(ins_);
    static T a;
    return a;
  }
 private:
  static void Magic(const T& x) { (void)x; };
  static T& ins_;
};

template <class T>
T& Activator<T>::ins_ = Activator<T>::Activate();
```

最后定义一个进行类数据收集的宏，我们假设每个要注册的类都提供一个RegType函数进行信息收集

```
#define REGCLASS(cls) \  
template<>\  
struct MetaDataCollector<cls>{\  
    MetaDataCollector()\  
    {\  
        cls::RegType();\  
    }\  
    static void Use()\  
    {\  
        Activator< MetaDataCollector<cls> >::Activate();\  
    }\  
}  
```

把这几段代码编成动态库给其它程序调用，之后我们新建一个项目，新建一个类C，并且对类进行元信息收集

```
class __declspec(dllexport) C{  
public:  
    static void RegType(){  
          
    };  
};  
  
REGCLASS(C);  
```

通过调试，我们顺利在GetInstance函数内得到断点，调用堆栈中显示了Dynamic initializtion字样

```
>    MyMain.exe!`dynamic initializer for 'Activator<MetaDataCollector<C> >::ins_''()  行 28 + 0x28 字节 C++  
```

同时查看obj的dump输出在..CRT$XCU组内看到了需要动态初始化的变量

```
Offset    Type              Applied To         Index     Name  
--------  ----------------  -----------------  --------  ------  
00000000  DIR32                      00000000        47  ??__E?ins_@?$Activator@U?$MetaDataCollector@VC@@@@@@2U?$MetaDataCollector@VC@@@@A@@YAXXZ (void __cdecl `dynamic initializer for 'public: static struct MetaDataCollector<class C> Activator<struct MetaDataCollector<class C> >::ins_''(void))  
```

可以看到 **ins_** 进行了动态初始化，通过定义宏可以看到这种实现还是比较优雅的，元信息类的初始化放在了单例类中,但是需要注意的初始化ins_的过程中调用的Activator函数必须使用ins_静态成员变量，否则模板类是无法触发Activate方法来实例化对象实例。

综合以上实现,我们用到了不完整类型定义元信息收集器，通过宏来定义具体的元信息收集器进行模板的激活，模板激活的过程中发现有静态成员变量需要初始化，
则首先调用Activator函数,如果Activator函数内没有任何影响静态变量值的代码，则无法触发静态初始化，类型实例也无法无法激活。

原因解释在c++标准中

> An implementation is permitted to perform the initialization of a non-local variable with static storage  
duration as a static initialization even if such initialization is not required to be done statically, provided  
that  
-   the dynamic version of the initialization does not change the value of any other object of namespace  
scope prior to its initialization, and  
-   the static version of the initialization produces the same value in the initialized variable as would be  
produced by the dynamic initialization if all variables not required to be initialized statically were  
initialized dynamically.  
