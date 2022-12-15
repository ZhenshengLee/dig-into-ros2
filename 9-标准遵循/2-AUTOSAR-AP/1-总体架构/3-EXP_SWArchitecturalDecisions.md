# 2-定义和术语

| Term  | Description | 备注 |
| --- | --- | --- |
| Adaptive Application  | See [1, AUTOSAR Glossary]. | 面向未来智能驾驶软件架构的适应性程序 |
| Execution Management  | A Functional Cluster. See [2, EXP_SWArchitecture] for an overview. | AP中所有的功能都是以Function Cluster来呈现 |
| Functional Cluster  | A logical group of functionality within the AUTOSAR Adaptive Platform. Functional Clusters are the subject of the individual specification documents that make up the AUTOSAR Adaptive Platform standard. See [2, EXP_SWArchitecture] for an overview of all Functional Clusters in the AUTOSAR Adaptive Platform. | FunctionCluster都是规范，而不是实际的程序，实际的程序需要符合规范 |
| Platform Health Management  | A Functional Cluster. See [2, EXP_SWArchitecture] for an overview. |  |
| Process  | See [1, AUTOSAR Glossary]. |  |
| State Management  | A Functional Cluster. See [2, EXP_SWArchitecture] for an overview. |  |
| Software Cluster  | See [1, AUTOSAR Glossary] and [2,EXP_SWArchitecture]. |  |
| Thread  | See [1, AUTOSAR Glossary]. |  |
| Watchdog  | An external component that supervises execution of the AUTOSAR Adaptive Platform. See [2, EXP_SWArchitecture] for an overview |  |

# 5-架构决策

## 5.1-动态内存分配

### 决策

The use of dynamic memory allocation by Adaptive Applications and Functional Clusters is allowed and assumed upon designing the AUTOSAR Adaptive Platform standard。<br />允许使用动态内存分配

### 考虑

由于AP使用C++，所以动态内存分配基本上不可避免<br />但是AP会用在安全系统上，而动态内存分配会造成不确定行为。两种典型的后果是内存碎片和内存分配回收导致的不确定性执行时间。对于非安全相关系统设计的allocators一般都会有这些问题，它们一般会兼顾内容效率或平均执行效率。<br />这些问题的解决依赖于确定性的memory allocators的设计，内存分配是一个成熟的领域，多种技术已经出现。多个供应商都报告已有有这种确定性内存分配器实现，用于量产系统。<br />注意这种分配器应该替代c编译器提供的默认的malloc/free，这个是new/delete和STL使用的默认分配器。

### 前提假设

供应商能够提供确定性的分配器来替换默认的分配器。

### 约束

在运行时的一些节点，确定性是强需求，这些特定阶段下，分配器需要替换成确定性版本的。

### 替代方案

不使用动态内存分配，在cpp开发中是不行的<br />在特定的阶段不允许动态内存分配，这是可行的，但是会使得软件开发变得异常困难。

### 需求

[RS_AP_00129] Public types defined by functional clusters shall be designed to allow implementation without dynamic memory allocation

## 5.2-使用final关键字来标识最终类

### 决策

AR类型应该使用final关键字，除非其需要作为基类

### 需求

[RS_AP_00140] Usage of "final specifier" in ara types

## 5.3-使用输出参数out parameter

### 决策

使用输出参数作为数据的原地修改，但是不能用作返回值

### 需求

[RS_AP_00141] Usage of out parameters

## 5.4-Usage of named constructors for exception-less object creation

## 5.5-引入单调的clock API

### 决策

ARA应用应该提供自己的单调的std::chrono::SteadyClock 来表示机器上电以来的时间，clock的精度由供应商来确定<br />时钟的精度用来作为特征值来确定平台对项目是否满足要求，比如时间同步就要求高精度的时钟。<br />epoch Of the clock由系统上电来决定，所以该时钟表示的是机器的上电时间。<br />所有FC应该使用这个是时钟作为时钟基础

### 考虑

时间同步功能应该使用std::chrono，因为时间同步对时钟精度要求较高

### 前提假设

The time synchronization cluster is typically a daemon-based architecture due to a single communication endpoint of the time sync messages. A standardized clock with a special accuracy as a common basis is required to synchronize the daemon with the library.

### 替代方案

也可以使用模板参数来确定使用何种时钟

## 5.6-监管功能（重点理解）

State Management, Execution Management, and Platform Health Management

### 决策

## 5.7-对象共享访问，使用local proxy objects

### 决策

Local proxy object(s) shall be used to provide shared access to object instance(s) via the AUTOSAR Runtime for Adaptive Applications interface。

### 考虑

local proxy objects封装了共享访问的实现细节。ARA接口应该返回一个值对象。调用方应该使用本地代理对象进行接下来的操作。值返回是最直接的返回数据的方法。这个决策统一了ARA接口的返回策略。内部实现由供应商决定。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669806497116-67f0bc09-5114-4414-a8fc-ebeaaa70e03b.png#averageHue=%23fcfbfa&clientId=u5e0e071f-2e9c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=425&id=uc4282393&margin=%5Bobject%20Object%5D&name=image.png&originHeight=425&originWidth=902&originalType=binary&ratio=1&rotation=0&showTitle=false&size=81906&status=done&style=none&taskId=u35e4d5b5-8460-4b1b-b0f2-ca188daa771&title=&width=902)

### 替代方案

对于proxy class，可选的替代方案是使用handles。这些句柄应该封装共享访问的具体细节。

### 相关需求

 [RS_AP_00135] Avoidance of shared ownership<br />避免共享所有权

## 5.8-标准化错误表示

### 决策

Functional clusters shall standardize production errors for common use-cases demanded by the market. The standardization shall summarize all production errors by a standardized table in all SWS documents specifying production errors.<br />对于市场要求的所有错误，FC要进行标准化，对SWS文档中定义好的所有产品错误进行表达。

### 考虑

产品错误是一个Fact，为了自动化分析和响应错误，错误的表达，存储和处理需要标准化。

### 前提假设

这些产品错误的概念是从CP过来的

### 替代方案

FC允许App去监控查看产品错误<br />（而不是FC主动上报）

## 5.9-虚函数中不允许有默认参数

### 决策

虚函数中不允许有默认参数

### 考虑

The according RQ of the "C++ core guidelines" are too weak .. (they state, that it needs be made sure that a default argument is always the same) ... this would lead to code duplication with dependencies and high risks of inconsistencies, which can easily lead to unexpected behavior。<br />C++标准中的规定太弱了

### 需求

[RS_AP_00148] Default arguments are not allowed in virtual functions

## 5.10-断言只有初始化的FC才能调用其API

### 决策

如果FC的API在初始化之前调用，则FC应该将次认定为非法。

### 考虑

在FC初始化之前调用功能，往往无法正确完成工作。调用的结果行为是未定义的。问题往往是系统在启动时刻没有正确配置，初始化不完整。这种错误API的调用方也没法处理。唯一的办法就是中断程序，避免后续产生更多失效。

### 替代方案

Extend every API that depends on prior initialization with a specific error code (e.g. kNotInitialized ) and force callers to check this error code at every call (and let them abort themselves).<br />每一个API调用都多一个错误代码kNotInitialized，当出现这个错误时由调用方负责中断程序，避免后续产生更多失效。

## 5.11-只提供即将被FC和App使用的接口

### 决策

It is explicitly prohibited to standardize implementation details, like:<br />• Classes, base-classes, functions etc. that are not used on the<br />application level or in platform extension APIs<br />• Implementation inheritance in the public APIs<br />• C++ SFINAE techniques of any kind<br />• Private members of classes

### 考虑

接口和实现应该尽可能解耦<br />隐藏实现细节，保护供应商权益

### 需求

[RS_AP_00150] Provide only interfaces that are intended to be used by<br />AUTOSAR applications and other Functional Clusters

## 5.12-遵循cpp核心guidlines

[https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md)

### 决策

AP cpp api应该遵循core guidlines，并遵循硬实时系统的例外条款。对于RS-General中的条款，如果与core-guid冲突的以RS-General为准。如果API中有不遵循core-guid的地方，应该明确其rational，说明处理细节，以及跟架构组达成一致。

### 考虑

这些core-guide是市场广泛接受的规范。这个规范的目的是让cpp程序员撰写更简单，更有效，更易于维护的代码。但是更具体的针对汽车行业的guideline目前是没有的。当MISRA Cpp标准出台之后，便可以遵循MISRA的规范了。

### 需求

[RS_AP_00151] C++ Core Guidelines

## 5.13-统一处理与后台断线错误

### 决策

### 考虑

## 5.14-诊断的细粒度

## 5.15-构造函数中的错误

### 决策

Calling a constructor that may throw exceptions as part of its defined behavior shall result in a compilation error if the compiler toolchain does not support exceptions. The compilation error shall result from a static_assert with the error message "This constructor requires exception support.".

### 考虑

Unintended calls to constructors that may throw exceptions are detected at compile time. static_assert is the only viable option. Declaring the constructor protected or private is more complicated. Moreover, static_assert supports a customized error message which explicitly states the cause.

## 5.16-重启的细粒度

## 5.17-Platform-independent development of Software Clusters of category APPLICATION_LAYER

## 5.18-统一的日志和跟踪

### 考虑

Standardized logging/tracing within Functional Clusters allows a harmonized evaluation of logging/tracing on vehicle-level<br />有标准的日志和跟踪，才有统一的分析。
