# 1-概述

介绍对所有FC的最基础的需求，具体来讲是定义了整个框架的功能，包括common data types，public interfaces

# 7-需求规约

## 7.1-适用于所有FC的通用需求

这个需求需要所有供应商严格遵守。

- [SWS_CORE_90001] Include folder structure
- [SWS_CORE_90002] Prevent multiple inclusion of header file
  - 使用#ifdefine保护
- [SWS_CORE_90003]DRAFT C/C++ symbols that start with ARA are reserved for use by AUTOSAR
  - AP拒绝使用预处理宏macro，但是如果要使用的话，以ARA开头
- [SWS_CORE_90004]fDRAFTg Implementation-defined declaration classifiers
  - 按要求严格使用声明标志符

### 7.1.1-初始化和去初始化

ara::core::Initialize allows a central initialization of all included shared libraries of the ARA framework. This could include static initializers or the setup of daemon links (details are up to the platform vendor).

## 7.2-FC功能定义

### 7.2.1-错误处理

#### 7.2.1.1-失败操作的类型

需要定义以下类型的错误

#### 7.2.1.2-c/cpp的传统错误处理

c的ErrorCode有以下风格：

- 返回值
- 输出参数
- error结构（比如errno）

一般来讲，错误代码是int型，使得其非常底层，而且基本没有类型安全可言。<br />cpp继承了c语言的这些特性，也在cpp标准中增加了其错误处理方式，比如异常exception，又叫做错误传播。错误传递有很多优点，STL大量使用到了异常。<br />同时，errorcode依然得到了广泛的使用，在STL中也是如此。原因在于，有的时候需要与C进行二进制兼容，但是一些新的库会基于下列原因使用errorno，而不是使用异常。

- 使用异常，很难判断程序的控制流的哪儿出了故障
- 异常的运行时开销很大（不管是在通用流程还是在异常抛出流程）

第一个原因跟开发人员和代码分析器有关，因为异常本质上是一种隐藏的控制流，像goto一样，使用了异常的函数除了return会返回外，抛出异常也可以返回，这样的话，函数很难进行阅读和评审，对于静态分析工具而言也是如此。<br />另一个原因是安全相关的，当前cpp编译器厂商通过ASIL的编译器都不支持异常，因为抛出异常在实现上采用了动态内存分配，这样一般都无法产生确定性的行为，执行时间没有明确的上界，使得汽车工业界

#### 7.2.1.3-AP中的错误处理

错误的处理：An Error shall be returned from the function as an instance of ara::core::Result or ara::core::Future.<br />非法操作的处理：

- throwing an exception that is not a subclass of ara::core::Exception
- explicitly terminating the process abnormally via a call to ara::core::Abort

#### 7.2.1.4-错误处理的四个方式

##### 7.2.1.4.1-ErrorCode

是一个类，基本参考了std::error_code，包含a low-level error code value and a reference to an error domain.<br />• error code value: an integral representation of a low-level error code<br />• error domain: reference to the context for which the error code value is applicable<br />• support data value: an optional vendor-specific additional piece of data about the error

##### 7.2.1.4.2-ErrorDomain

。。。

##### 7.2.1.4.3-Result

The ara::core::Result type follows the ValueOrError concept from the C++ proposal p0786 [5]. It either contains a value (of type ValueType), or an error (of type ErrorType).<br />ValueOrError主要依靠的是std::optional。

##### 7.2.1.4.4-Future和Promise

在前面几篇文章中基本都用到thread对象，它是C++11中提供异步创建多线程的工具。但是我们想要从线程中返回异步任务结果，一般需要依靠全局变量；从安全角度看，有些不妥；为此C++11提供了std::future类模板，future对象提供访问异步操作结果的机制，很轻松解决从异步任务中返回结果。<br />**异步操作的创建者能用各种方法查询、等待或从 std::future 提取值。若异步操作仍未提供值，则这些方法可能阻塞。**<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669864411409-0f46835f-d826-4074-9965-ce7694a24122.png#averageHue=%23fefdfd&clientId=udc3a5309-940f-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u8005352f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=500&originWidth=716&originalType=url&ratio=1&rotation=0&showTitle=false&size=33094&status=done&style=none&taskId=ub96e5ebb-0754-49e0-b7c7-65c26d77218&title=)

### 7.2.2 async signal safety

### 7.2.3-显式的中断操作

to do this; either by throwing certain kinds of exceptions (if the implementation supports C++ exceptions), or by calling ara::core::Abort.

#### 7.2.3.1-AbortHandler

This handler can be installed with ara::core::SetAbortHandler. It is invoked in turn when ara::core::Abort is called, and it may perform arbitrary operations and then has these four principal choices for its final statements: it can either<br />• terminate the process, or<br />• return from the function call, or<br />• defer function return by entering an infinite loop, or<br />• perform a non-local goto operation such as std::longjmp.<br />The use of non-local goto operations, including std::longjmp, is strongly discouraged and also expressively prohibited by MISRA, the AUTOSAR C++14 Coding Guidelines, and most other coding guidelines as well.

#### 7.2.3.2-SIGABRT Handler

In addition to the ara::core::AbortHandler, or alternatively to it, the application can also influence this mechanism by installing a signal handler for SIGABRT.

### 7.2.4-AdvancedTypes

#### 7.2.4.1-AUTOSAR Types

##### 7.2.4.1.1-InstanceSpecifier

Instances of ara::core::InstanceSpecifier are used to identify service port prototype instances within the AUTOSAR meta-model and are therefore used in the ara::com API and elsewhere.

##### 7.2.4.1.2-ScaleLinearAndTexttable

A ara::core::ScaleLinearAndTexttable type is a struct type that emulates an enumeration type with extended capabilities, such as those given in [9, the C++17 standard]<br />In particular, it can hold the values of the enumeration, but also any value of the underlying type of the Enumeration Data Type with which it was defined.

#### 7.2.4.2-从cpp标准继承来的类型

Array and SteadyClock

#### 7.2.4.3-从新cpp标准继承来的类型

Byte

### 7.2.5-初始化和去初始化

初始化应该包括以下流程<br />• initialization of ARA framework specific data structures初始化数据结构<br />• initialization of system resources初始化系统资源<br />• spawning of background threads启动后台线程<br />去初始化应该包括以下流程<br />• orderly shutdown of spawned background threads关闭相关后台线程<br />• deallocation of dynamically allocated memory释放动态分配的内存<br />• deallocation of other system resources释放其他系统资源

# 8-具体规范

慢慢看
