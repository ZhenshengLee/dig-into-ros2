# ROS与AP对比总结

## 特点对比

|  | ROS2 | AP | 备注 |
| --- | --- | --- | --- |
| 定位 | 智能机器人领域的事实规范 | 车规级软件的开发规范 | 针对AP，不同供应商的实现不一样 |
| 理念 | SOA | SOA | 两者理念高度相似，但解耦程度不一样 |
| 时间 | ROS2立项时间为2015年 | AP发布时间是2017年 |  |
|  | ROS2是代码先行 | AP是规范先行 |  |
|  | ROS2开源，允许商业化<br />发展速度快，参与人数多 | 车企供应商深度参与协作，迭代速度稍慢，有包袱 |  |
| 通信中间件<br />com-middleware | ROS2拥抱了DDS规范<br />这是一个先进的规范，是网络中间件发展的最新进展 | AP定义规范，取决于实现<br />跨主机通信AP拥抱SOME/IP，这是基于车载以太网的协议 |  |

## 概念对应

AP中的SOA概念：service consumers (proxy) and service providers (skeleton)<br />ROS中的SOA概念：service consumers (sub and server) and service providers (pub and client)

| 类型&#124; 框架 | AP | ROS2 | Cyber | 备注 |
| --- | --- | --- | --- | --- |
| SOA实体 | AP Node/ AP SWC | Node | Node |  |
| SOA通讯管理 | CM | rclcpp |  | CM不是Daemon-based的，CM向下依赖的Logger，TCP/IP协议栈  |
| 发布订阅模式：发布 | EventSkeleton | Publisher | Writer |  |
| 发布订阅模式：订阅 | EventProxy | Subscriber | Reader |  |
| 请求响应模式：请求 | MethodProxy | ServiceClient | ServiceClient |  |
| 请求响应模式：响应 | MethodSkeleton | ServiceServer | ServiceServer |  |
| 请求汇报模式：请求 | - | ActionClient | - |  |
| 请求汇报模式：汇报 | - | ActionServer | - |  |
| 参数服务：客户端 | FieldsProxy | ParametersClient |  |  |
| 参数服务：服务端 | FieldsSkeleton | ParameterService |  |  |
| 服务标识 | ServiceType<br />InstanceID | TopicName<br />ServiceName<br />IDL |  | 1-服务标识，AP的细粒度更大，ROS的细粒度更小，ROS的更加灵活<br />2-AP以服务为中心，通过Service找Node，ROS以数据（Name&Type）为中心，只看数据不看来源。<br />（安全机制：只有指定的节点能够在指定的话题发送数据，SROS2） |
| 服务定义 | SI+IDL | SI+IDL |  | SI是设计出来的具体接口，IDL是实现SI的工具和语言 |

## SOA概念及模式

与SOA对应的是Signal Oriented Arch<br />Service Oriented Comm 对应的是Signal Oriented Comm<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670930661983-c47aa14d-711f-4ba9-9b51-3b1eeaea2065.png#averageHue=%23fcfcfc&clientId=u6972b7f5-3edc-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u9f36f3b8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=390&originWidth=850&originalType=url&ratio=1&rotation=0&showTitle=false&size=105401&status=done&style=none&taskId=u7dad1b97-de4e-47e6-a97b-7c330eb37f4&title=)

## 对ROS2的态度

ROS2架构明确，各组件解耦性好，已经拥有支持车规级定义的软件特性，或者拥有针对特性的扩展性；<br />由于ROS2是开源系统，面向不同层次和不同领域的用户，拥有复杂和多种特性，而且灵活可配；<br />ROS2基于现代Cpp打造，上手容易，用好难，开发难；<br />我们对ROS2的态度是

- 基于AP的规范，扩充ROS2的特性，完善ROS2的机制
- 严格限制使用的特性（apex的feature reduction），仅使用符合AP规范的特性

## 参考文献

Schmidt D C, Buschmann F. Patterns, frameworks, and middleware: their synergistic relationships[C]//25th International Conference on Software Engineering, 2003. Proceedings. IEEE, 2003: 694-704.<br />Rumez M, Grimm D, Kriesten R, et al. An overview of automotive service-oriented architectures and implications for security countermeasures[J]. IEEE access, 2020, 8: 221852-221870.

# 1-EXP_PlatformDesign

## 3.3-AP的主要特点

前述章节已经讲述了AP的驱动力，在这些驱动力下，AP必须具备一些特征，来处理上述驱动力带来的挑战。<br />这些特征包括以下的新技术。

### 3.3.1-Cpp

From top-down, the applications can be programmed in C++. It is now the language of choice for the development of new algorithms and application software in performance critical complex applications in the software industry and in academics. This should bring faster adaptations of novel algorithms and improve application development productivity if properly employed.<br />对于有性能需求的软件和算法而言，cpp是主流的选择，如果使用得当，可以加快软件开发效率。

### 3.3.2-SOA

To support the complex applications, while allowing maximum flexibility and scalability in processing distribution and compute resource allocations, AP follows serviceoriented-architecture (SOA).<br />为了支持复杂的应用，同时具备最大的灵活性，扩展性，在分布式架构和资源受限系统中，AP遵循SOA的架构体系。<br />**The SOA is based on the concept that a system consists of a set of services, in which one may use another in turn, and applications that use one or more of the services depending on its needs.**<br />SOA架构能支持：功能映射到服务；服务使用；服务发现<br />_SOA的第一个视角在于，系统由若干Service组成，Service之间有使用的关系，应用程序根据需要使用一些Service。_<br />_ROSSOA的概念在于，系统由若干Node组成，Node之间有使用的关系，应用程序根据需要使用一些Node。_<br />**关键点在于Service之间的通信和Node之间的通信规约，Service之间的通信是比较紧耦合的，Node之间的通信是比较松耦合的。**<br />SOA也具备SOS的特点，System-of-System，即一个Service内部可能是一个AP实例，并由可能<br />_ROS也具备SOS的特点，即一个Node可内部可能是一个ROS实例，采用的是bridge模式，_<br />**The application code is the same in both cases the communication infrastructure will take care of the difference providing transparent communication**. **Another way to look at this architecture is that of distributed computing, communicating over some form of message passing. At large, all these represent the same concept. This message passing, communication-based architecture can also benefit from the rise of fast and high-bandwidth communication such as Ethernet.**<br />应用程序保持一致，通信基础设施来保证数据的透明传输。<br />_另一个视角就是基于消息传递的分布式计算通信，这是一种message-passing, communication-based的体系结构。_<br />_而ROS的体系结构与此概念一致。_<br />_注意：与SOA相对应的概念是Signal Oriented Arch_

### 3.3.6-受管控的不确定性(Planned dynamics)

在部署方面，为了尽量提升软件开发和集成的效率，AP支持动态部署应用程序，这样就可以支持快速迭代的软件流程。<br />在安全方面，AP允许集成商谨慎限制软件的动态行为，来减少非预期的风险，以做到安全认证。动态行为约束在ExecutionManifest中。<br />尽管如此，在执行期间，程序的计算资源的动态获取，只允许在configured阶段进行。<br />受管控的不确定性特点如下

- Pre-determination of the service discovery process
  - 服务发现过程的确定性
- Restriction of dynamic memory allocation to the startup phase only
  - 动态内存分配只允许在启动阶段进行
- Fair scheduling policy in addition to priority-based scheduling
  - 公平调度策略加上优先级调度策略
- Fixed allocation of processes to CPU cores
  - 进程与CPU核的绑定
  - 进程亲和性是不够的，应该要做资源隔离
  - 从一个container多个进程，变成一个container一个进程，才是最佳实践
  - [https://www.tutorialworks.com/containers-single-or-multiple-processes/](https://www.tutorialworks.com/containers-single-or-multiple-processes/)
- Access to pre-existing files in the file-system only
  - 只允许访问预先定义的文件
  - 不允许动态创建文件
- Constraints for AP API usage by Applications
  - 严格研制应用程序能够使用的API
- Execution of authenticated code only
  - 只允许执行可信的代码

## 8.2-ServiceOrientedCommunication

The notion of a service means functionality provided to applications beyond the functionality already provided by the basic operating software.<br />在这里服务的概念表示提供给客户程序的功能，在基本功能基础上的封装。<br />CM的目的就是提供机制来保证服务使用的透明性，方便使用服务，不管是Intra-machine还是Inter-machine通信<br />A service consists of a combination of

- Events事件
- Methods方法
- Fields字段

Communication paths between communication partners can be established at design-, at startup- or at run-time. An important component of that mechanism is the Service Registry that acts as a brokering instance and is also part of the Communication Management software。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669701781204-cf2b8422-4eb7-4bc7-bc96-c2ba5b5827de.png#averageHue=%23d6cec4&clientId=u3324c857-ac3c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=448&id=u09426ce7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=448&originWidth=573&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43665&status=done&style=none&taskId=ufb1fb65f-b2c8-4999-b00d-618fe512c54&title=&width=573)<br />ROS的特点是broker-less，即采用了分布式发现机制，跟交换机二层广播协议类似，需要发现的时候就广播。

## 8.3-LanguageBinding and NetworkBinding

CM提供一种标准的方式来向app开发者展示可以用的服务，针对app，有一个layer叫做LanguageBinding，针对中间件的层次叫做NetworkBinding。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669702378239-35d746ae-510c-4746-91b2-08cbb6ea6d0c.png#averageHue=%23e8cfaa&clientId=u3324c857-ac3c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=508&id=ubac85d02&margin=%5Bobject%20Object%5D&name=image.png&originHeight=508&originWidth=921&originalType=binary&ratio=1&rotation=0&showTitle=false&size=119033&status=done&style=none&taskId=ud9ed3334-6a03-4735-b1e7-2133d057e91&title=&width=921)<br />The Language Binding defines how the methods, events, and fields of a service are translated into directly accessible identifiers by using convenient features of the targeted programming language. Performance and type safety (as far as supported by the target language) are the primary goals. Therefore, the Language Binding is typically implemented by a source code generator that is fed by the service interface definition.<br />语言绑定用于定义，方法，事件和字段如何在特定编程语言中表示，重点考虑的是性能和类型安全。所以一般是由一个code_gen来做的。<br />The Network Binding defines how the actual data of a configured service is serialized and bound to a specific network. It can be implemented based on Communication Management configuration (interface definition of the AUTOSAR meta model) either by interpreting a generated service specific recipe or by directly generating the serializing code itself. Currently, Communication Management supports SOME/IP, DDS, IPC (Inter-Process-Communication or any other custom binding), Signal PDU (SignalBased Network binding) and Signal-Based Static Network binding.<br />网络绑定用于定义数据如何串行化，通过指定的网络通路发送出去。当前支持的通路包括SOME/IP，DDS，IPC，Signal PDU，Signal-based Static Network binding.<br />_在ROS中LanguageBinding的公共部分叫做rclcpp，动态生成的部分叫做rosidl_typesupport_cpp和rosidl_typesupport_c，_<br />_由于只支持DDS，NetworkBinding在ROS2中细分为支持多种DDS，可以称作是DDSBinding，公共的部分叫做rmw，动态生成的部分叫做rosidl_typesupport_fastrtps_c，rosidl_typesupport_fastrtps_cpp，以上动态生成的内容叫做IDL的类型支持，服务的类型支持，统称为rosidl_typesupport_

## 8.4-Generated Proxies and Skeletons of C++ Language Binding

A generator that is part of the development tooling for the Communication Management software generates C++ classes that contain type safe representations of the fields, events, and methods of each respective service.<br />On the service implementation side, these generated classes are named Service Provider Skeletons. On the client side, they are called Service Requester Proxies.<br />For Service Methods, a Service Requester Proxy provides mechanisms for synchronous (blocking the caller until the server returns a result) and asynchronous calling (called function returns immediately). A caller can start other activities in parallel and receives the result when the server’s return value is available via special features of the Core Type ara::core::Future<br />对于Methods，则提供同步调用和异步调用，异步调用也需要等待future，来判断是否调用成功。<br />_Methods对应ROS2中的Service。_<br />_在ROS2中其实也有Service Provider Skeletons和Service Requester Proxies，不过这些都作为公共部分，定义在rclcpp中了，不需要由code_gen生成。_<br />Service Requester Proxies由Clients直接使用，而Service Provider Skeletons由Servers通过集成实现虚方法的方式来实现。<br />Events和Fields一起构成ROS2中的Topic。<br />The interfaces of ara::com can also provide proxies and skeletons for safety-related E2E protected communication. These interfaces are designed that compatibility to the applications is assured independent whether E2E protection is switched on or off.<br />ara::com api是用来保证E2E通信的，其也产生特定的Proxy和Skeleton。

# 2-概述

为什么AUTOSAR重新定义一种新的中间件API，而不是复用当前市场中已经有的技术？<br />在设计新接口之前，我们调研了现有技术，包括以下几种：

- ROS API
- DDS API
- CommonAPI（GENNIVI）
- DADDY API(Bosch)

AP的接口定义，一方面是严格遵循的SOA的架构术语，一方面主要参考了DADDY，图为iceoryx的术语<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670897437485-ea48a5a9-2862-458e-971f-ac6676501ee3.png#averageHue=%23252b2d&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=JvAnk&margin=%5Bobject%20Object%5D&originHeight=376&originWidth=1340&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ufcb36a9b-a879-4b2b-b641-80215fe05b4&title=)<br />调研的最终决策是要设计一种符合AUTOSAR特定需求的通信管理API，因为以上解决方案不能解决我们的关键需求

- 我们需要一个CM通信管理，不绑定到任何通信协议，其必须支持SOME/IP但是也需要灵活性来对外协作
  - 这个不是我们的强需求
- 对AUTOSAR service model，即methods，events，和fields的支持应该是自然和直接的 natural和straight forward
  - 这个不是我们的强需求
- API需要同时支持event-driven和polling model的通信模式，polling model是实时系统的典型需求，可以避免不需要的上下文切换，event-driven针对非实时应用来讲，更加方便。
- 尽可能无缝衔接端到端的通信保护，以满足ASIL的需求
- 支持静态或者动态的服务发现协议

所以最终设计的API，其实可以发现有很多现有协议的影子，比如：

- Proxy (or Stub)/Skeleton approach (CORBA, Ice, CommonAPI, Java RMI, ...)
  - 根据DateExchangeFormat来生成Stub供应用程序使用（CORBA通信思想）
- Protocol independent API (CommonAPI, Java RMI)
  - 不依赖特定协议的API，为底层替换协议提供灵活性（分层架构思想）
- Queued communication with configurable receiver-side caches (DDS, DADDY, Classic Platform)
  - 基于缓存队列的通信（来自DDS）
- Zero-copy capable API with possibility to shift memory management to the middleware (DADDY)
  - 具备零拷贝能力的API，应用将内存所有权移交给中间件（来自DADDY）
- Data reception filtering (DDS, DADDY)
  - 数据过滤能力（来自DDS）

# 3-术语

| Abbreviation / Acronym:  | Description: |
| --- | --- |
| ctor  | C++ constructor |
| dtor  | C++ destructor |
| IPC  | Inter Process Communication |
| RT  | Realtime |
| SI  | Service Interface |
| WET  | Worst Case Execution Time |

| Terms:  | Description: | 备注 |
| --- | --- | --- |
| Binding  | This typically describes the realization of some abstract concept with a specific implementation or technology.<br />In AUTOSAR, for instance, we have an abstract data type and interface model described in the methodology. Mapping it to a concrete programming language is called language binding. In the AUTOSAR Adaptive Platform for instance<br />we do have a C++ language binding.<br />In this explanatory document we typically use the tech term binding to refer to the implementation of the abstract (technology independent) ara::com API to a concrete communication transport technology like for instance sockets, pipes, shared memory, ... | 描述的是抽象概念的实现。<br />在AUTOSAR中，我们在方法论中有抽象数据类型和接口模型，将其映射到编程语言的过程叫做language binding，在AP中有cpp language binding.<br />在本文档中，binding指的是ara::com的API映射到cpp的具体接口，涉及到socket，pipes，shared memory等等。 |
| Callable  | In the context of C++ a Callable is defined as: A Callable type is a type for which the INVOKE operation (used by, e.g., std::function, std::bind, and std::thread::thread) is applicable. This operation may be performed explicitly using the library function std::invoke.(since C++17) | 在cpp的上下文中，callable是可调用对象，包括std::function，bind的结果，thread线程对象等。在cpp17后可以使用std::invoke来统一调用。 |

# 4-API设计概览

API设计的目标是尽可能简介。意味着，只会提供功能的最小集合，来支持methods，events和fields。<br />在这个上下文中尽可能简洁指的是，基本上API只会处理method，field和event的通信，其他的一概不管。<br />所以，ara::com不会提供任何component model和framework来处理组件生命周期，管理程序流程，或者单纯设置ara::com对象。<br />这些都可以构建在ara::com API之上，不需要标准化。<br />我们的设计出发点是：支持polling和event-driven通信模型，轮询模型对于实时性程序更好，基于事件的模型对于编程来讲更好。<br />当前ROS2主要是基于event的通信模型，但是也有polling的模型。

# 5-高层API结构

## 5.1-Proxy/Skeleton架构

熟悉中间件架构的都知道，这个架构，这个架构又叫做stub/skeleton架构<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669972428442-f62cf3be-585e-49a9-b105-aec6e07a55a8.png#averageHue=%23fae8e1&clientId=ue3640442-b2b7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=467&id=uba23c262&margin=%5Bobject%20Object%5D&name=image.png&originHeight=467&originWidth=867&originalType=binary&ratio=1&rotation=0&showTitle=false&size=62343&status=done&style=none&taskId=u9fdab939-7a8d-4099-a5d1-c1878ce50e2&title=&width=867)<br />The basic idea of this pattern is, that from a formal service definition two code artifacts are generated:<br />基本理念在于，一份service description产生两份代码：

- ServiceProxy: 代码交给服务使用方，作为Facade门面来标识所要使用的服务
  - 在面向对象的代码下，往往提供了一系列的类，提供服务的所有功能，所以服务使用方可以通过这个门面来使用服务
- ServiceSkeleton: 这个代码给到Service服务实现方，根据服务描述来提供相应的功能，这个代码联系了底层中间件层和上层的服务实现，使得服务实现可以借由中间件将服务远程提供给各个使用方使用。
  - 在面向对象的代码下，往往提供了一系列的类，服务实现通过继承这些类来实现服务
  - 所以服务端的代码通过继承类，实现虚方法或者调用一些已有方法来实现服务

ROS的也有Stub<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670901090395-7a356724-442b-4af2-bc4d-015ffc978948.png#averageHue=%2321262c&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=461&id=u9c8baad1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=461&originWidth=413&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42006&status=done&style=none&taskId=u3d603e16-d204-4486-8fc4-6f2d8524359&title=&width=413)

## 5.2-Runtime Interface

runtime singleton，运行时单例，不在proxy/skeleton中，但是也需要提供特定的功能

## 5.3-DataTypeAbstractions

ara::com引入了数据类型，这些数据类型大致分为四个类型：

- Pointer types: for pointers to data transmitted via middleware
  - 指针类型，指向数据
- Collection types: for collections of data transmitted via middleware
  - 数据集合类型，定义具体的数据集合
- Types for async operation result management: ara::com relies on AUTOSAR AP specific data types (see [4]), which are specific versions of C++ std::future/std::promise
  - 异步通信管理API，类似std::future和std::promise
- Function wrappers: for various application side callback or handler functions to be called by the middleware
  - 仿函数，定义回调或者handler

ara::com定义了这些函数的签名，但是不规定其实现，其实现是由各个供应商来决定和优化的。<br />显然，collection和pointer types是IPC实现的主要数据类型。

## 5.4-错误通知

ara::com遵循AP中的错误处理方法论，ErrorCode的方式，集成在ara::core::Result中。<br />return code based error handling

### 5.4.1-checked errors

ara::com的api只在context of a call of service interface method<br />通过return code based error

### 5.4.2-unchecked errors

The ara::com API does not throw any Execption. The only way to have exceptions is calling the get method of ara::core::Future, if the user decides to use this approach.<br />通过std::future based error check

# 6-API 元素

```yaml
RadarService {
// types used within service
type RadarObjects {
active : bool
objects : array {
elementtype: uint8
size: variable
}
}

// radar的数据类型

type Position {
x: uint32
y: uint32
z: uint32
}

// events provided by service
event BrakeEvent {
type:RadarObjects
}

// radar的核心数据通过events发出，这个event的名字叫做brake_event，其实指的就是雷达检测到的点云数据
// 在ros2中，events由timer callback发出，一般名字叫做radar_data，不与特定的业务强绑定，即不假设radar用于brake_event
// 从这个角度讲，ros2的灵活性是更大的

// fields provided by service
field UpdateRate {
type:uint32
get: true
set: true
}

// 在ros2中,fields是参数parameter

error CalibrationFailed {
errorCode : 1
errorContext {
failureText : string
}
}

// ap中将错误集成到了service中进行统一定义
// ros2中也有，是service的response

error InvalidConfigString {
errorCode : 2
errorContext {
invalidConfig : string
currentValidConfig : string
}
}

// 另一个error

 // methods provided by service
 method Calibrate {
 param configuration {
 type: string
 direction: in
 }
 param result {
 type: bool
 direction: out
 }
 raises {
 CalibrationFailed
 InvalidConfigString
 }
 }

// ap中将methods的result和error分开定义
// ros2中的service的response也可以这样定义，将response定义为一个结构体，包含result和error即可

 method Adjust {
 param target_position {
 type: Position
 direction: in
 }
 param success {
 type: bool
 direction: out
 }
 param effective_position {
 type: Position
 direction: out
 }
 }

// 另一个service

 oneway method LogCurrentState {}
 }

// ap支持扩充类型的method，即无返回的method
// 在ros2中

//总的来讲，ros2定义的service更加松耦合，更加宽松，使得proxy/skeleton模式进一步解耦，合并为一份代码
//服务调用和服务提供的stub合并到rclcpp中，publisher和subscriber，service_client和service_server
```

_简单讲，event对应topic，method对应service，fields对应parameter_<br />_ros2还支持更加复杂的交互模式，action，比ap更加灵活_<br />_但是ap更加严格，将error定义到了service中，不过ros2也提供了这种机制，没有那么严格，将选择权交给了开发者_

## 6.1-InstanceIdentifiers

Instance of a service<br />实例标识，作为一个中心概念，被发布和订阅方共同使用。**发布方在服务示例创建好之后创建这个示例，订阅方在查询服务实例的时候使用该实例标识**。在技术层面，其完全就是一个通用的标识，UIID，SOME/IP使用一个16bit的int整型变量来区分不同的service实例，DDS规范使用一个string<256>作为service_instance_name

### InstanceSpecifier

InstanceIdentifiers到InstanceSpecifier，方便软件开发者进行识别的一个Local name。<br />Then, where does the software-developer get such a highly binding specific ara::com::InstanceIdentifier to be used in ara::com API calls?<br />The answer is: By an ara:com provided functionality, which translates a logical local name used typically by the software developer in his realm into the technology/binding specific ara::com::InstanceIdentifier. This indirection masters both challenges:

- developer using ara::com does not need to know anything about bindings and their specifics
  - 开发者不需要关注使用的标志符和组中的Identifier的关系
- Integrators can adapt bindings in deployments
  - 开发者可以做具体的适配和扩充

The local name from which the ara::com::InstanceIdentifier is constructed comes basically from AUTOSAR meta-model, describing your software component model. <br />这样specifier和具体的executable绑定起来，而identifier和具体的service绑定起来，做到了解耦，给了开发者一定的灵活性<br />比如开发者可以通过manifest的方式来定义specifier的名字<br />The API ara:com provides the following function, to do the translation from the ara::core::InstanceSpecifier (local name in the software developers realm) to the technical ara::com::InstanceIdentifier:

### multi-binding多个InstanceID

AUTOSAR支持一个Sepcifier绑定多个Identifier<br />在server端multi-binding是正常操作，因为需要支持不同的clients去使用特定的binding，本地通路/远程通路<br />在client端multi-binding是不好的，除了特定的fail-over场景，如果bindingA失败，则bindingB

### 6.1.1-InstanceID和InstanceSP

一般情况使用InstanceSP即可，特殊场景自己调用ResolveInstanceIDs()<br />The fundamental difference between the two variants is this: An ara::com::InstanceIdentifier can be exchanged more easily between Adaptive Applications/processes!<br />两者的基本区别就是：InstanceID更容易在进程间交换！

#### 6.1.1.1-InstanceID的传输

风险很大，vendor-specific

## 6.2-ProxyClass

ara::com已经标准化了proxy class，由供应商来做具体的实现。<br />ara::com要求proxy相关的制品在proxy名字空间内

```cpp
class RadarServiceProxy {
public:
/**
* \brief Implementation is platform vendor specific
*
* A HandleType must contain the information that is needed to create
* a proxy.
*
* This information shall be hidden.
 * Since the platform vendor is responsible for creation of handles,the
 * ctor signature is not given as it is not of interest to the user.
 */
 class HandleType {
 /**
 * \brief Two ServiceHandles are considered equal if they represent
 * the same service instance.
 *
 * \param other
 *
 * \return bool
 */
 inline bool operator==(const HandleType &other) const;
 const ara::com::InstanceIdentifier &GetInstanceId() const;
 };

 /**
 * StartFindService does not need an explicit version parameter as this
 * is internally available in ProxyClass.
 * That means only compatible services are returned.
 *
 * \param handler this handler gets called any time the service
 * availability of the services matching the given
 * instance criteria changes. If you use this variant of
 * FindService, the Communication Management has to
 * continuously monitor the availability of the services
 * and call the handler on any change.
 *
 * \param instanceId which instance of the service type defined
 * by T shall be searched/found.
 *
 * \return a handle for this search/find request, which shall
 * be used to stop the availability monitoring and related
 * firing of the given handler. (\see StopFindService())
 */
 static ara::com::FindServiceHandle StartFindService(
 ara::com::FindServiceHandler<RadarServiceProxy::HandleType> handler,
 ara::com::InstanceIdentifier instanceId);

 /**
 * This is an overload of the StartFindService method using an
 * instance specifier, which gets resolved via service instance
 * manifest.
 * \param instanceSpec instance specifier
 */
 static ara::com::FindServiceHandle StartFindService(
 ara::com::FindServiceHandler<RadarServiceProxy::HandleType> handler,
 ara::core::InstanceSpecifier instanceSpec);

 /**
 * This is an overload of the StartFindService method using neither
 * instance specifier nor instance identifier.
 * Semantics is, that ALL instances of the service shall be found, by
 * using all available/configured technical bindings.
 *
 */
 static ara::com::FindServiceHandle StartFindService(
 ara::com::FindServiceHandler<RadarServiceProxy::HandleType> handler);

 /**
 * Method to stop finding service request (see above)
 */
 static void StopFindService(ara::com::FindServiceHandle handle);

 /**
 * Opposed to StartFindService(handler, instance) this version
 * is a "one-shot" find request, which is:
 * - synchronous, i.e. it returns after the find has been done
 * and a result list of matching service instances is
 * available. (list may be empty, if no matching service
 * instances currently exist)
 * - does reflect the availability at the time of the method
 * call. No further (background) checks of availability are
 * done.
 *
 * \param instanceId which instance of the service type defined
 * by T shall be searched/found.
 *
 */
 static ara::com::ServiceHandleContainer<RadarServiceProxy::HandleType>
FindService(
 ara::com::InstanceIdentifier instanceId);

 /**
 * This is an overload of the FindService method using an
 * instance specifier, which gets resolved via service instance
 * manifest.
 */
 static ara::com::ServiceHandleContainer<RadarServiceProxy::HandleType>
FindService(
ara::core::InstanceSpecifier instanceSpec);

 /**
 * This is an overload of the StartFindService method using neither
 * instance specifier nor instance identifier.
 * Semantics is, that ALL instances of the service shall be found, by
 * using all available/configured technical bindings.
 */
 static ara::com::ServiceHandleContainer<RadarServiceProxy::HandleType>
dService();

 /**
 * \brief The proxy can only be created using a specific
 * handle which identifies a service.
 *
 * This handle can be a known value which is defined at
 * deployment or it can be obtained using the
 * ProxyClass::FindService method.
 *
 * \param handle The identification of the service the
 * proxy should represent.
 */
 explicit RadarServiceProxy(HandleType &handle);

 /**
 * proxy instances are not copy constructible.
 */
 RadarServiceProxy(RadarServiceProxy &other) = delete;

 /**
 * proxy instances are not copy assignable
 */
 RadarServiceProxy& operator=(const RadarServiceProxy &other) = delete;

 /**
 * \brief Public member for the BrakeEvent
 */
 events::BrakeEvent BrakeEvent;

 /**
 * \brief Public Field for UpdateRate
 */
 fields::UpdateRate UpdateRate;

 /**
 * \brief Public member for the Calibrate method
 */
 methods::Calibrate Calibrate;

 /**
 * \brief Public member for the Adjust method
 */
 methods::Adjust Adjust;

 /**
 * \brief Public member for the LogCurrentState fire-and-forget method
 */
 methods::LogCurrentState LogCurrentState;
 };
```

### 6.2.1-构造函数和句柄Handle

What it is, should be straightforward: After the call to the ctor you have a proxy instance, which allows you to communicate with the service, therefore the handle has to contain the needed addressing information, so that the Communication Management binding implementation is able to contact the service.

### 6.2.2-FindingServices

由于Service的可用性是动态变化的，所以可用性是有生命周期的，ara::com提供两种方式来findService

- StartFindService is a class method, which starts a continuous “FindService” activity in the background, which notifies the caller via a given callback anytime the availability of instances of the service changes.
  - 异步操作，后台查找，异步通知
- FindService is a one-off call, which returns available instances at the point in time of the call.
  - 同步操作，会阻塞

返回一个handles容器，可能包含多个InstanceID，可能返回空<br />有不同的查找服务的方式

- one taking an ara::com::InstanceIdentifier
- one taking an ara::core::InstanceSpecifier
- one taking NO argument.
  - 寻找所有的服务

#### 6.2.2.1-AutoUpdateProxyInstance

使用FindService之后，获取了Handle，但是Handle的活跃状态是需要维护的，比如ServiceProvider下线了，然后又上线，这种情况下Proxy是否能够继续使用之前的InstanceID。<br />为了解决这个问题，AP提供一个Service Descovery Infrastructure, service providers和service comsumers的生命周期进行管理。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670911076366-c80b9f2b-a74e-4ba1-b937-098fb77c2094.png#averageHue=%23faf9f8&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=558&id=ub881d8f0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=558&originWidth=616&originalType=binary&ratio=1&rotation=0&showTitle=false&size=62398&status=done&style=none&taskId=u5057de30-b0a9-4da1-9232-b7fbdeafb4d&title=&width=616)<br />所有的通知由ServiceDiscovery基础设施来做

- T0: The service consumer may successfully call a service method of that proxy (and GetSubscriptionState() on subscribed events will return kSubscribed according to 6.2.3.2).
- T1: The service instance goes down, correctly notified via service discovery.
  - ServiceInstance down掉，CM需要通知到Proxy
- T2: A call of a service method on that proxy will lead to a checked exception (ara::com::ServiceNotAvailableException), since the targeted service instance of the call does not exist anymore. Correspondingly GetSubscriptionState() on any subscribed event will return kSubscriptionPending (see also 6.2.3.2) at this point even if the event has been successfully subscribed (kSubscribed) before.
- T3: The service instance comes up again, notified via service discovery infrastructure. The Communication Management at the proxy side will be notified and**will silently update the proxy object instance with a possibly changed transport layer addressing information**. This is illustrated in the figure with transport layer part of the proxy, which changed the color from blue to rose. The Binding implementer hint part below discusses this topic more detailed.
- T4: Consequently service method calls on that proxy instance will succeed again and GetSubscriptionState() on events which the service consumer had subscribed before, will return kSubscribed again.

### 6.2.3-Events（topic subscriber）

对于每一个Event提供方，proxy包含一个wrapper类用于用于访问events和event data，这些事件和数据是由service instance发送出来的。<br />（DDS的底层也是change event）

```cpp
class BrakeEvent {
/**
* \brief Shortcut for the events data type.
*/
using SampleType = RadarObjects;
/**
* \brief The application expects the CM to subscribe the event.
*
* The Communication Management shall try to subscribe and resubscribe
* until \see Unsubscribe() is called explicitly.
* The error handling shall be kept within the Communication Management
*
* The function returns immediately. If the user wants to get notified,
* when subscription has succeeded, he needs to register a handler
* via \see SetSubscriptionStateChangeHandler(). This handler gets
* then called after subscription was successful.
*
* \param maxSampleCount maximum number of samples, which can be held.
*/
void Subscribe(size_t maxSampleCount);
/**
* \brief Query current subscription state.
 *
 * \return Current state of the subscription.
 */
 ara::com::SubscriptionState GetSubscriptionState() const;

 /**
 * \brief Unsubscribe from the service.
 */
 void Unsubscribe();

 /**
 * \brief Get the number of currently free/available sample slots.
 *
 * \return number from 0 - N (N = count given in call to Subscribe())
 * or an ErrorCode in case of number of currently held samples
 * already exceeds the max number given in Subscribe().
 */
 ara::core::Result<size_t> GetFreeSampleCount() const noexcept;

 /**
 * Setting a receive handler signals the Communication Management
 * implementation to use event style mode.
 * I.e. the registered handler gets called asynchronously by the
 * Communication Management as soon as new event data arrives for
 * that event. If the user wants to have strict polling behavior,
 * where no handler is called, NO handler should be registered.
 *
 * Handler may be overwritten anytime during runtime.
 *
 * Provided Handler needs not to be re-entrant since the
 * Communication Management implementation has to serialize calls
 * to the handler: Handler gets called once by the MW, when new
 * events arrived since the last call to GetNewSamples().
 *
 * When application calls GetNewSamples() again in the context of the
 * receive handler, MW must - in case new events arrived in the
 * meantime - defer next call to receive handler until after
 * the previous call to receive handler has been completed.
 */
 void SetReceiveHandler(ara::com::EventReceiveHandler handler);

 /**
 * Remove handler set by SetReceiveHandler()
 */
 void UnsetReceiveHandler();

 /**
 * Setting a subscription state change handler, which shall get
 * called by the Communication Management implementation as soon
 * as the subscription state of this event has changed.
 *
 * Communication Management implementation will serialize calls
 * to the registered handler. If multiple changes of the
 * subscription state take place during the runtime of a
 * previous call to a handler, the Communication Management
 * aggregates all changes to one call with the last/effective
 * state.
 *
 * Handler may be overwritten during runtime.
 */
 void SetSubscriptionStateChangeHandler(
 ara::com::SubscriptionStateChangeHandler handler);

 /**
 * Remove handler set by SetSubscriptionStateChangeHandler()
 */
 void UnsetSubscriptionStateChangeHandler();

 /**
 * \brief Get new data from the Communication Management
 * buffers and provide it in callbacks to the given callable f.
 *
 * \pre BrakeEvent::Subscribe has been called before
 * (and not be withdrawn by BrakeEvent::Unsubscribe)
 *
 * \param f
 * \parblock
 * callback, which shall be called with new sample.
 *
 * This callable has to fulfill signature
 * void(ara::com::SamplePtr<SampleType const>)
 * \parblockend
 *
 * \param maxNumberOfSamples
 * \parblock
 * upper bound of samples to be fetched from middleware buffers.
 * Default value means "no restriction", i.e. all newly arrived samples
 * are fetched as long as there are free sample slots.
 * \parblockend
 *
 * \return Result, which contains the number of samples,
 * which have been fetched and presented to user via calls to f or an
 * ErrorCode in case of error (e.g. precondition not fullfilled)
 */
 template <typename F>
 ara::core::Result<size_t> GetNewSamples(
 F&& f,
 size_t maxNumberOfSamples = std::numeric_limits<size_t>::max());
 };
```

_支持subscribe语义，而且支持buffer队列(叫做LocalCache)，subscribe语义支持设置回调，这基本上是ROS1的能力_<br />_ROS2不仅仅具备ROS1的能力，还具备QoS，支持在弱网络环境中保证通信质量。_<br />**_注意：_**_和ROS不同的是，AP的通信由CM管理，而ROS的通信是一对一的TCP/UDP/SHM_

#### 6.2.3.1-EventSubscription和LocalCache

```cpp
void Subscribe(size_t maxSampleCount);
```

App需要维护一个缓存队列，叫做localCache

#### 6.2.3.2-MonitoringEventSubscription

The call to the Subscribe method is asynchronous by nature. This means that at the point in time Subscribe returns, it is just the indication, that the Communication Management has accepted the order to care for subscription.<br />订阅方法是异步的，意思仅仅表示CM收到了这个消息，开始检测Event的发生。订阅流程下的操作，取决于具体的实现，最终涉及到event provider side。<br />由于订阅方法马上返回，所以并不意味着Provider已经确认了此事，当然IPC的具体实现也可以这么实现。<br />但是用户可以通过这个方法来确认订阅已经成功

```cpp
ara::com::SubscriptionState GetSubscriptionState() const;

enum class SubscriptionState { kSubscribed, kNotSubscribed, kSubscriptionPending };
```

这个语句会实现一个机制，叫做subscription acknowledge，这个确认回复来自于event provider，这个函数是同步的，马上返回状态。<br />如果用户想监控订阅的状态，有两种可选方法：

- Polling via GetSubscriptionState
  - 轮询
- Registering a handler, which gets called, when the subscription state changes
  - 注册回调handler的方式

```cpp
using SubscriptionStateChangeHandler = std::function<void(SubscriptionState)>;
```

在AP中有subscription_state的概念，订阅成功，订阅失败，订阅挂起<br />_而ROS中解耦程度更大，订阅者不需要知道关注发布者存不存在，当然，AP的订阅者也不需要关注发布者存不存在_<br />_ROS也提供类似于subscription_state的概念，_<br />[https://docs.ros2.org/foxy/api/rclcpp/classrclcpp_1_1PublisherBase.html](https://docs.ros2.org/foxy/api/rclcpp/classrclcpp_1_1PublisherBase.html)

```cpp
size_t  get_subscription_count () const
size_t  get_intra_process_subscription_count () const
```

_由于解耦程度更高，这些是publisher可以知道的状态，订阅方不需要知道_<br />_在ROS中，订阅方通过判断时间戳是否过期来判断订阅关系是否发生异常，而不需要publisher去通知订阅状态_<br />_AP中的Proxy对应于ROS中的subscriber和service_client_

#### 6.2.3.3-访问事件数据-Samples

当订阅成功后，如何访问事件的数据？event data从event emitter发送而来，从IPC实现的角度讲，缓存在一些buffer中，比如kernel buffer，SHM空间，所以需要一种显式的动作，从buffer中fetch一些数据sample，进一步解串，将其放置在wrapper class的cache中，数据类型是SampleType，这个API如下

```cpp
 /**
 * \brief Get new data from the Communication Management
 * buffers and provide it in callbacks to the given callable f.
 *
 * ....
 */
 template <typename F>
 ara::core::Result<size_t> GetNewSamples(
 F&& f,
 size_t maxNumberOfSamples = std::numeric_limits<size_t>::max());
```

第一个是回调函数，以SampleType为参数，第二个是最大样本数量<br />ara::com实现首先检查最大数量是否超限，然后检查缓冲区是否有数据，如果有新数据，则解串到一个数据类型，调用回调函数<br />所以说AP的回调的调用是主动的，而不是被动的，但与ROS相比，AP的回调调用方式比较直接，需要开发者一个一个考虑<br />其实ROS的回调调用也是主动的，但是ROS的解耦程度更高，不需要专门的主动调用回调，而是由executor统一调回调，而且回调类型包括了timer, subscription, service_server, action_server等，统一起来了，降低了开发难度<br />This processing (checking for further samples in the buffer and calling back the application provided callback f) is repeated until either:

- there aren’t any new samples in the buffers
- there are further samples in the buffers, but the application provided maxNumberOfSamples argument in call to GetNewSamples() has been reached.
- there are further samples in the buffers, but the application already exceeds its maxSampleCount, which it had committed in Subscribe().

_数据取用的过程与ROS是类似的，不过ROS将这块工作，get的功能交给了DDS_<br />_ROS当前采用的是Proactor模式，简单讲是wait&Take模式，polling轮询数据到来的事件，然后通过dds去取数据_<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670847825175-e652d0ce-5136-4a5f-acff-6728aceccdcf.png#averageHue=%23fcfcfc&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=459&id=u8e738a73&margin=%5Bobject%20Object%5D&name=image.png&originHeight=612&originWidth=1273&originalType=binary&ratio=1&rotation=0&showTitle=false&size=72902&status=done&style=none&taskId=u36d6710c-ac67-4d9d-91a1-5cf438b9a55&title=&width=955)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670847941864-17ac70be-bdf8-489d-80eb-015c80eb6cbc.png#averageHue=%2397cef7&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=471&id=u399dcdda&margin=%5Bobject%20Object%5D&name=image.png&originHeight=942&originWidth=1834&originalType=binary&ratio=1&rotation=0&showTitle=false&size=446190&status=done&style=none&taskId=ub3069a50-7970-4640-8752-9272c578fd8&title=&width=917)<br />galactic\src\ros2\rclcpp\rclcpp\src\rclcpp\executors\single_threaded_executor.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670293212018-e1249163-e36f-4d59-b4e0-288414f83b79.png#averageHue=%23252b34&clientId=uaa0a647e-fa58-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=274&id=u7f073ed6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=274&originWidth=596&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34756&status=done&style=none&taskId=u8eae363b-0793-4e29-9e0e-5ffbef9ab9d&title=&width=596)<br />galactic\src\ros2\rclcpp\rclcpp\src\rclcpp\executor.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670293236519-b53cba26-be59-4301-9d6f-f6a88c921d6a.png#averageHue=%23252b34&clientId=uaa0a647e-fa58-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=480&id=uaaa0f26a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=480&originWidth=823&originalType=binary&ratio=1&rotation=0&showTitle=false&size=87112&status=done&style=none&taskId=u912ad8a3-6104-4110-ac04-3ec981be96d&title=&width=823)<br />galactic\src\ros2\rclcpp\rclcpp\src\rclcpp\subscription_base.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670293251312-de41c35c-9faf-4662-a6d7-3ff1138818c0.png#averageHue=%23262c36&clientId=uaa0a647e-fa58-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=323&id=u57f58b95&margin=%5Bobject%20Object%5D&name=image.png&originHeight=323&originWidth=466&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35880&status=done&style=none&taskId=uf066fb48-1d86-4150-b6de-e500126f19f&title=&width=466)<br />galactic\src\ros2\rcl\rcl\src\rcl\subscription.c<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670293272706-07afc93b-ebc0-40d3-a979-b45d9f28bec9.png#averageHue=%23252b35&clientId=uaa0a647e-fa58-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=453&id=udee073a0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=453&originWidth=805&originalType=binary&ratio=1&rotation=0&showTitle=false&size=81165&status=done&style=none&taskId=u192c8ed2-1178-46f6-bb02-79ac13c3440&title=&width=805)<br />galactic\src\ros2\rmw_fastrtps\rmw_fastrtps_shared_cpp\src\rmw_take.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670293300489-da86b98e-353c-4ed2-b6ca-966bc10470d2.png#averageHue=%23242932&clientId=uaa0a647e-fa58-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=569&id=u223b61df&margin=%5Bobject%20Object%5D&name=image.png&originHeight=569&originWidth=807&originalType=binary&ratio=1&rotation=0&showTitle=false&size=75646&status=done&style=none&taskId=u50224885-13f3-4e4b-bb73-8d38ad12bde&title=&width=807)<br />在DDS这一层，也是take_next_sample的方法，可见AP的定义也是差不多的

#### 6.2.3.4-EventSample管理-通过SamplePtrs

一个指针，从ara::com传递到用户层，是一个std::unique_ptr，ara::com将所有权传递到用户层后，由用户决定其生命周期，如果用户不销毁该SamplePtr，ara::com无法重新申请这块内存，这块内存本来是由ara::com申请的。申请的过程一般是发生在subscribe()函数调用中，根据maxSampleCount申请的。在GetNewSamples()调用之后，ara::com将内存填满数据，将SamplePtr传递到use callback<br />在回调的实现中决定如何使用SamplePtr指向的数据，如果应用想在后面继续访问该数据，则会拷贝一份到外面。是否要拷贝取决于应用。<br />对于拷贝的场景，callback的角色是一个filter，如果不拷贝，就是move<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670309081939-105288e5-08bb-471f-80d0-08e74d6f2590.png#averageHue=%23fdfcfb&clientId=u412d9f7f-23f6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=482&id=u53dd69d0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=482&originWidth=916&originalType=binary&ratio=1&rotation=0&showTitle=false&size=117803&status=done&style=none&taskId=u8424cf57-7d5a-4fa2-9d0f-a3f47c98412&title=&width=916)

#### 6.2.3.5-Event-Driven还是Polling-Based access

AP支持event-driven的和polling 的方式来访问新数据。对于polling approach，不需要额外的API。典型的场景是，app循环做一些处理流程，输出信息，这是control算法的常见模式，这样的周期性激活需要real-time实时定时器激活，来保证更小的jitter<br />在polling下，在每个循环中格斗GetNewSamples，使用最新的数据作为算法的输入，所以这是最充分的做法，保证在算法线程调度到的第一时间就获取最新数据。<br />但是polling的缺点在于有可能适得其反，如果CM在任何时刻都通知你的app，则可能导致进程不必要的上下文切换，在收到通知的时候，你不想去处理新数据，因为时机不对。<br />但是其他用况下是有用的，如果应用程序没有这种轮询机制，而是采用事件触发的机制，则polling的方式来调用GetNewSamples则是效率低下的。（这句话是废话吧）<br />这种情况下，CM通知app即可，异步触发上下文切换，用这种方式

```cpp
void SetReceiveHandler(ara::com::EventReceiveHandler handler);
```

该函数不需要可重入，因为CM对回调的调用是完全顺序执行的。<br />注意：ROS的回调调用是可以多线程可重入的！<br />It is explicitly allowed to call GetNewSamples() from within the registered callback!<br />下面的例子使用event-driven方式<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670330216087-6bcd8b3f-1a9f-4479-86c4-153733d2591f.png#averageHue=%23fdfdfc&clientId=u54e457c7-0ddf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=168&id=u1e3f79e5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=168&originWidth=938&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36565&status=done&style=none&taskId=u5832b6ab-d656-4c2d-8bf8-2716156b74c&title=&width=938)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670330228808-9bf73396-83e9-4554-a80c-2c56076aa313.png#averageHue=%23fdfcfb&clientId=u54e457c7-0ddf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=486&id=u0704b837&margin=%5Bobject%20Object%5D&name=image.png&originHeight=486&originWidth=911&originalType=binary&ratio=1&rotation=0&showTitle=false&size=117444&status=done&style=none&taskId=u5fd4791f-138b-45a4-b5f7-c28f244b087&title=&width=911)<br />_注意：ROS2的callback方式也是event-driven的_

#### 6.2.3.6-buffering策略

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670379351717-8b510aeb-7ebd-4bc9-8e4c-6c6a6fcf9239.png#averageHue=%23fbf8f6&clientId=u54e457c7-0ddf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=530&id=u22fa5b57&margin=%5Bobject%20Object%5D&name=image.png&originHeight=530&originWidth=754&originalType=binary&ratio=1&rotation=0&showTitle=false&size=59653&status=done&style=none&taskId=u966a9284-f99a-4c68-b215-0c2685222a0&title=&width=754)<br />有两个SWCs通过proxy订阅了该事件，从图中可以看出，两个proxy都有一块local event cache，这块Cache在GetNewSamples()之后填充。Skeleton将数据发送到CM的Buffer处，这个Buffer与Proxy进程和Skeleton进程是空间独立的，这个buffer可能由内核维护，也可能由一个SHM中间件维护，可能由一个后台进程来维护。<br />这个图背后还隐含一个假设：AP应用运行在虚拟内存地址空间。<br />Event数据不能在Skeleton进程的地址空间缓存，如果是这样的话，proxy需要切换到skeleton的上下文。<br />Buffer可能有三个地方

- Kernel Space: Data is sent to a memory region not mapped directly to an application process. This is typically the case, when binding implementation uses IPC primitives like pipes or sockets, where data written to such a primitive ends up in kernel buffer space.
  - 内存发送到内核空间，这个是典型的场景，通过pipes，sockets等IPC手段可以访问到数据
- Shared Memory: Data is sent to a memory region, which is also directly readable from receivers/proxies. Writing/reading between different parties is synchronized specifically (lightweight with mem barriers or with explicit mutexes).
  - SHM空间，用户态程序可以直接访问，SHM空间访问需要合适的内存竞争策略
- IPC-Daemon Space: Data is sent to an explicit non-application process, which acts as a kind of demon for the IPC/binding implementation. Note, that technically this approach might be built on an IPC primitive like communication via kernel space or shared memory to get the data from service process to demon process
  - IPC-Daemon空间，这个空间是由一个daemon进程来维护的，通过IPC技术可以交换数据

上述方法有不同的优缺点，需要针对因素进行考虑，这些因素包括：

- flexibility，灵活性
- buffer size，空间大小
- access speed，访问速度
- overhead，开销

所以AP产品需要考虑不同的约束条件，不同的分析会导致不同的解决方案<br />但是有一点需要强调一下，规范鼓励供应商以引用的方式来访问数据，这样可以尽量减少拷贝。<br />在1: N的场景下，主要涉及下列两个问题：

- Local Event Cache的更新，是更新value还是更新reference?
- consumers之间共享数据

_与ROS2对比，ROS比AP解耦性更强，~~体现在没有一个CM作为中间商进行数据缓存~~，但是_

- _DDS也有一个中心化的数据存储，比如iceoryx有一个中心化的daemon memory pool_

_DDS有更先进的通信质量管理机制，叫做QoS_

### 6.2.4-Methods(service client)

通过()操作符来访问Methods，远程调用一个函数<br />得到一个output，包括两个信息：成功与否，请求的feedback

#### 6.2.4.1-One-Way aka Fire-and-Forget Methods

触发然后忘记，请求响应通信模式的简化版本，不需要响应，no feedback<br />从资源的角度讲，one-way是轻量级的，就像一个remote function call

#### 6.2.4.2-Event-driven和polling的方式

对于future补充说明如下：

- std::async 、 std::packaged_task 或 std::promise 能提供一个std::future对象给该异步操作的创建者
- 异步操作的创建者能用各种方法查询、等待或从 std::future 提取值。若异步操作仍未提供值，则这些方法可能阻塞。
- 异步操作准备好发送结果给创建者时，它能通过接口（eg,std::promise::set_value std::future） 修改共享状态的值。

同步访问，get会造成阻塞<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670382129380-3416f412-e075-46a8-9163-f2f5b5720ebd.png#averageHue=%23fefdfd&clientId=u5dd2006e-87b5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=479&id=uf8cef954&margin=%5Bobject%20Object%5D&name=image.png&originHeight=479&originWidth=907&originalType=binary&ratio=1&rotation=0&showTitle=false&size=81397&status=done&style=none&taskId=u5efac8c7-7675-435b-906e-355bcf29dc7&title=&width=907)<br />异步访问，有两种方法<br />There are other ways for the user to get a notification from the Communication Management implementation as soon as the method result is available beside resuming<br />execution from a blocking call to get():

- The variants of “wait” , which the ara::core::Future has taken over from std::future. They basically provide the functionality of a blocking wait for the fulfillment of the future.
  - 等待，有超时机制，这个机制依然会造成阻塞，但是超时机制可以解决死锁问题
- Registering a callback method via then(). This is one of the extensions to the std::future; see [4] for details.
  - 回调机制，这个才是event-driven的方法

get(), then()的语法实现在iceoryx上得到体现<br />[https://github.com/eclipse-iceoryx/iceoryx/blob/master/iceoryx_examples/icedelivery/iox_publisher.cpp](https://github.com/eclipse-iceoryx/iceoryx/blob/master/iceoryx_examples/icedelivery/iox_publisher.cpp)

```cpp
        publisher.loan()
            .and_then([&](auto& sample) {
                sample->x = sampleValue1;
                sample->y = sampleValue1;
                sample->z = sampleValue1;
                sample.publish();
            })
            .or_else([](auto& error) {
                // Do something with error
                std::cerr << "Unable to loan sample, error: " << error << std::endl;
            });
```

ROS2的service支持上述所有机制。

#### 6.2.4.3-Canceling Method Result

这么一种情况

- 你调用了方法，方法也反馈了future，但是你不关注这个结果了
- 你注册了方法的回调，但是想忽略这个callback

**discarding a future**<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670383072033-0ff447bf-25df-4f95-8934-194754e8540c.png#averageHue=%23fefefd&clientId=u5dd2006e-87b5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=217&id=u8d7b46a3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=217&originWidth=878&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32432&status=done&style=none&taskId=u1434bc1b-6898-4e0f-972a-100c91ecb0c&title=&width=878)<br />_这个需求的本质是future的使用和销毁，目前ROS2不支持取消callback，但是支持future的这种使用方法_

### 6.2.5-Fields(parameter client)

是一个值，相比event而言有以下不同

- 如果对field的订阅完成了，则马上会有一个初始值回传到subscriber
- 可以通过Get()和Set()来查询和修改值
- 值修改之后有on-change-notification

Fields的使用方式是method和events的混合

- 通过Get()和Set()来读取和查询值
- 通过events来接受数据更新通知

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670568422338-46493745-9fbe-479c-bf54-1ab2460ef086.png#averageHue=%23fbfaf8&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=329&id=u14546abf&margin=%5Bobject%20Object%5D&name=image.png&originHeight=329&originWidth=948&originalType=binary&ratio=1&rotation=0&showTitle=false&size=99991&status=done&style=none&taskId=u2ec33f57-b985-4df0-8d05-8c246cebd4d&title=&width=948)<br />在ROS中有一个单独的概念Parameter来处理Fields<br />Fields和Config在AP中是两个概念，在ROS中Fields和Configuration在ROS中统一处理，使用Parameter的概念，parameter通过文件加载的方式，则是config，通过在线配置的方式，则是Fields的方式。<br />关于Parameter的实现全部在rclcpp中。包括<br />galactic\src\ros2\rclcpp\rclcpp\include\rclcpp\parameter_client.hpp 等<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670415056121-d8da55d1-c900-44a6-bbbc-c4ff8c8a918a.png#averageHue=%23262c34&clientId=u1aa05264-c46b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=151&id=u18268c55&margin=%5Bobject%20Object%5D&name=image.png&originHeight=151&originWidth=246&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5212&status=done&style=none&taskId=u0f9e811b-37b5-479e-a787-910981cee15&title=&width=246)<br />Arg的概念和Configuration的概念类似，但是必须进行区分，在ROS中是单独处理的，称为--ros-args

## 6.3-SkeletonClass

Skeleton和Proxy Class的区别在于，Skeleton的类是虚基类，无法直接实例化，而是要开发者通过继承的方式实现特定的虚函数来做到。

```cpp
 class RadarServiceSkeleton {
 public:
 /**
 * Ctor taking instance identifier as parameter and having default
 * request processing mode kEvent.
 */
 RadarServiceSkeleton(ara::com::InstanceIdentifier instanceId,
 ara::com::MethodCallProcessingMode mode =
 ara::com::MethodCallProcessingMode::kEvent);

 /**
 * Ctor taking instance identifier container as parameter and having
 * default request processing mode kEvent.
 * This specifically supports multi-binding.
 */
 RadarServiceSkeleton(ara::com::InstanceIdentifierContainer instanceIds,
 ara::com::MethodCallProcessingMode mode =
 ara::com::MethodCallProcessingMode::kEvent);

 /**
 * Ctor taking instance specifier as parameter and having default
 * request processing mode kEvent.
 */
 RadarServiceSkeleton(ara::core::InstanceSpecifier instanceSpec,
 ara::com::MethodCallProcessingMode mode =
 ara::com::MethodCallProcessingMode::kEvent);

 /**
 * skeleton instances are nor copy constructible.
 */
 RadarServiceSkeleton(const RadarServiceSkeleton& other) = delete;

 /**
 * skeleton instances are nor copy assignable.
 */
 RadarServiceSkeleton& operator=(const RadarServiceSkeleton& other) =
lete;

 /**
 * The Communication Management implementer should care in his dtor
 * implementation, that the functionality of StopOfferService()
 * is internally triggered in case this service instance has
 * been offered before. This is a convenient cleanup functionality.
 */
 ~RadarServiceSkeleton();

 /**
 * Offer the service instance.
 * method is idempotent - could be called repeatedly.
 */
 void OfferService();

 /**
 * Stop Offering the service instance.
 * method is idempotent - could be called repeatedly.
 *
 * If service instance gets destroyed - it is expected that the
 * Communication Management implementation calls StopOfferService()
 * internally.
 */
 void StopOfferService();

 /**
 * For all output and non-void return parameters
 * an enclosing struct is generated, which contains
 * non-void return value and/or out parameters.
 */
 struct CalibrateOutput {
 bool result;
 };

 /**
 * For all output and non-void return parameters
 * an enclosing struct is generated, which contains
 * non-void return value and/or out parameters.
 */
 struct AdjustOutput {
 bool success;
 Position effective_position;
 };

 /**
 * This fetches the next call from the Communication Management
 * and executes it. The return value is a ara::core::Future.
 * In case of an Application Error, an ara::core::ErrorCode is stored
 * in the ara::core::Promise from which the ara::core::Future
 * is returned to the caller.
 * Only available in polling mode.
 */
 ara::core::Future<bool> ProcessNextMethodCall();

 /**
 * \brief Public member for the BrakeEvent
 */
 events::BrakeEvent BrakeEvent;

 /**
 * \brief Public member for the UpdateRate
 */
 fields::UpdateRate UpdateRate;

 /**
 * The following methods are pure virtual and have to be implemented
 */
 virtual ara::core::Future<CalibrateOutput> Calibrate(
 std::string configuration) = 0;
 virtual ara::core::Future<AdjustOutput> Adjust(
 const Position& position) = 0;
 virtual void LogCurrentState() = 0;
 };
```

### 6.3.1-实例化

构造函数有多个签名，意味着有多重实例化的方式，要求传入unique的instance id，如果instance_id一样的情况下重复实例化，则需要前一个service先销毁，所以skeleton class不支持拷贝构造和拷贝赋值

- variant with ara::com::InstanceIdentifier: Service instance will be created with exactly one binding specific instance identifier.
- variant with ara::com::InstanceIdentifierContainer: Service instance will be created with bindings to multiple distinct instance identifiers. This is mentioned as "multi-binding" throughout this document and also explained in more detail in section 9.3
- variant with ara::core::InstanceSpecifier: Service instance will be created with bindings to the instance identifier(s) found after "service manifest" lookup with the given ara::core::InstanceSpecifier. Note, that this could also imply a "multi-binding" as the integrator could have mapped the given ara:- :core::InstanceSpecifier to multiple technical/binding specific instance identifiers within the "service manifest".

```cpp
ara::com::MethodCallProcessingMode mode = ara::com::MethodCallProcessingMode::kEvent
```

注意：在子类继承实现skeleton之后，创建instance，这个instance不会可见，只有通过OfferService API才能让这个instance对用户可见<br />_在ROS中没有service instance的概念，在rcl内部可能维护了这个instance，但是用户不需要关心。用户需要关心的是publisher, subscriber, service_client, service_server, parameter, parameter_server等ROS核心概念，这也进一步说明了ROS对service的解耦相比AP更加彻底。_

### 6.3.2-OfferingServiceInstance

在Service的实现方，实现skeleton类，并且设置好instance之后，此时意味着已经可以准备好向consumers提供服务了，此时调用OfferService()。在OfferService()没有返回的时刻，你的实例已经开始提供服务了<br />对于你不想提供服务的时刻，可以调用StopOfferService()。<br />_在ROS中，通过LifecycleNode机制对publisher，subscriber，service_client, service_server的行为进行控制_<br />_不同的是，publisher和service_server对应AP的Skeleton，publisher已经具有lifecycle了，service_server目前没有lifecycle，这个是可以向社区提需求的，已经开发完了，还有一个是根据node的lifecycle自动变更publisher的lifecycle，这个需求已经开发完了_<br />[https://github.com/ros2/rclcpp/issues/1846](https://github.com/ros2/rclcpp/issues/1846)<br />[https://github.com/ros2/rclcpp/issues/1506](https://github.com/ros2/rclcpp/issues/1506)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670486506180-375e6f66-3a1a-4fbb-aa30-d6b262b76a1d.png#averageHue=%23fdfdfc&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=428&id=uba1e4a1a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=428&originWidth=921&originalType=binary&ratio=1&rotation=0&showTitle=false&size=85105&status=done&style=none&taskId=u72fa5bbf-ae4c-4ded-addc-63523bc9813&title=&width=921)

### 6.3.3-event-driven和polling的运行模式（service_server）

**在service provider side也可以支持这两种模式。**<br />**在纯event-driven的配置下，CM产生调用事件call events，然后将事件转换为skeleton的methods**<br />这种配置造成如下后果：

- general reaction to a service method call might be fast, since the latency is only restricted by general machine load and intrinsic IPC mechanism latency.
  - 一般情况下，负载和IPC延迟都正常，service 的method执行会很快
- rate of context switches to the OS process containing the service instance might be high and non-deterministic, decreasing overall throughput.
  - 在高负载下，随着上下文切换开销增大，service的服务执行延迟可能比较高，而且行为是不确定的，导致总体吞吐量下降。

另外一个是polling approach，在这种行为模式下，service provider side explicitly calls an ara::com provided API to process explicitly one call event. 这样可以打造实时应用程序RT-Application<br />在polling模式下，不阻塞的方式轮询检查event queue中的事件，来判断队列中有多少events要处理。只有一些RT timers能够触发上下文切换，异步的事件不会触发上下文切换<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670503881323-75c447b9-a8cc-417d-ada5-788bf23c6ced.png#averageHue=%23efebe6&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=158&id=u5effa705&margin=%5Bobject%20Object%5D&name=image.png&originHeight=158&originWidth=935&originalType=binary&ratio=1&rotation=0&showTitle=false&size=67274&status=done&style=none&taskId=u35a5a4a0-fb19-451f-a845-b20f9cad37c&title=&width=935)<br />具体来讲，MethodCallProcessingMode可以分为如下选项

```cpp
enum class MethodCallProcessingMode { kPoll, kEvent, kEventSingleThread };
```

_所以，在AP环境下，proxy可以通过事件机制通知CM，CM然后通过事件机制通知skeleton_<br />_同时，在polling模式下，skeleton会轮询事件队列，在发现事件后马上处理_<br />这里的事件event主要是指methods_call()

#### 6.3.3.1-PollingMode

如果设置为kPoll，则CM不会异步调用Skeleton的任何方法。<br />应用程序需要显式调用任何调用

```cpp
ara::core::Future<bool> ProcessNextMethodCall();
```

典型的流程是

- RT application gets scheduled.
  - 应用调度到
- it calls ProcessNextMethodCall and registers a callback with ara::core::Future::then()
  - 调用polling轮询程序，注册一个回调（std::future机制）
- the callback is invoked after the service method called by the middleware corresponding to the outstanding request has finished.
  - 回调得到调用
- in the callback the RT application decides, if there is enough time left for serving a subsequent service method. If so, it calls another ProcessNextMethodCall
  - 回调中判断是否还有时间，如果还有时间，则继续轮询

这种利用std::future的机制，比不停调用ProcessNextMethodCall要好一些。（不过本质上是一样的，前者只不过借用了future的机制）<br />当然，有一个是实现者需要考虑的，就是当事件来了之后，服务端是在当前上下文中处理，还是新起一个线程去处理。

#### 6.3.3.2-EventDrivenMode

如果选项设置成kEvent活kEventSingleThread，则CM将会将事件异步派发到Skeleton。<br />与kPoll相反，service comsumer，也就是Proxy，是隐式控制/触发 service provider的线程调度的。<br />kEvent和kEventSingleThread的区别在于，kEvent意味着MultiThread，会以并发的方式调用call<br />kEventSingleThread意味着是单线程调用call，这些call是排队执行的，会阻塞。<br />为什么提供两种选项呢？从功能上来讲MultiThread已经够用了，如果多个call不能并发运行，比如有数据竞争问题或者一致性问题，那么就用mutex就可以。但是从效率上讲，对于有强烈同步的需求，用很多个mutex就是资源的浪费。

### 6.3.4-Methods(service_server)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670506266754-65dbdb0e-48f5-4d86-9d1f-77065f720623.png#averageHue=%23fcfbfa&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=338&id=u133e1a03&margin=%5Bobject%20Object%5D&name=image.png&originHeight=338&originWidth=785&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60338&status=done&style=none&taskId=ufbe7349c-ec9e-4876-9937-508ae401a21&title=&width=785)<br />入参是一个const ref，返回值是future。<br />也许实现方会将这个调用派发到一个中央worker-thread-pool，这样会很ugly，因为返回success不意味着service call的完成。<br />在thread pool场景下，service call会等待thread-pool返回的通知，然后返回future，这种阻塞是不可接受的。<br />同步执行的方式如下<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670506753697-c1087a0f-81d9-4517-b726-b1eb577e24fb.png#averageHue=%23fdfbfa&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=581&id=uaa9e3c2c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=581&originWidth=909&originalType=binary&ratio=1&rotation=0&showTitle=false&size=101731&status=done&style=none&taskId=u4d1dd974-8e65-48c0-b340-9b0454d6855&title=&width=909)<br />future是从Promise中创建的。<br />对于caller而言，可以马上调用Future::get()，而且这个函数不会阻塞，马上返回。<br />异步执行如下<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670506919124-e8391ef0-7915-4f73-a9c4-5ef9225f2ba9.png#averageHue=%23fdfcfb&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=554&id=u79f3253f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=554&originWidth=933&originalType=binary&ratio=1&rotation=0&showTitle=false&size=105771&status=done&style=none&taskId=u42d65205-ebb4-40c2-ae19-78cf3b40595&title=&width=933)<br />用一个detach的thread来执行，这个实现只能是示例，因为detach是不好的实践

#### 6.3.4.1-Fire-and-Forget

不需要返回

#### 6.3.4.2-RaisingApplicationError返回错误

通过在future中设置错误值即可

### 6.3.5-Events(topic publisher)

Skeleton这边对Events，负责通知事件的发生。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670549530060-d6de839a-8c32-45fb-b0f8-61de78518075.png#averageHue=%23fdfdfc&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=461&id=uc56caee4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=461&originWidth=786&originalType=binary&ratio=1&rotation=0&showTitle=false&size=84774&status=done&style=none&taskId=ud6347210-c2ac-4919-a9d1-4ab6f238eef&title=&width=786)<br />有两种Send签名

- 第一种参数是SampleType的引用
  - 这种很直接，应用已经分配好内存，然后给引用即可
  - 在call完之后，这个数据内存已经释放了，所以底层实现需要做一个copy
- 第二种签名，参数是SampleAllocateePtr<SampleType>，本质上是一种std::unique_ptr<T>，这个的意思是，所有权只能有一个，如果想传递所有权，唯一的办法就是move
  - 为什么需要unique_ptr语义？
  - 这个内存不是由用户分配，而是交给底层软件分配的，然后用户写入数据即可，为什么交给底层软件进行内存分配，简单的答案就是为了数据拷贝优化

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670551373338-eeb077e4-196b-4f73-8c6d-dea2943dbf3a.png#averageHue=%23fdf8f7&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=529&id=ucabc6774&margin=%5Bobject%20Object%5D&name=image.png&originHeight=529&originWidth=904&originalType=binary&ratio=1&rotation=0&showTitle=false&size=108773&status=done&style=none&taskId=u0cb706ef-2ba3-431b-945f-e935f83aaac&title=&width=904)<br />第一种Send，用户分配一次内存，然后底层中间件拷贝一次，底层中间件拷贝一次才能让对端访问，当数据较大，拷贝开销是很大的。<br />第二种Send，由<br />请注意：fill the event data也可能会拷贝，这是另外的话题，属于construction phase 的拷贝是避免不了的。<br />ROS的实现从Galactic/Humble开始已经默认采用std::unique_ptr，如果传入的是const&，则会改成unique_ptr<br />[https://github.com/ros-perception/vision_opencv/issues/413#issuecomment-1272525465](https://github.com/ros-perception/vision_opencv/issues/413#issuecomment-1272525465)<br />[https://github.com/ros2/rclcpp/blob/7f575103d8fe06285bd46474a0324c7dfbb2a2f0/rclcpp/include/rclcpp/publisher.hpp#L287-L304](https://github.com/ros2/rclcpp/blob/7f575103d8fe06285bd46474a0324c7dfbb2a2f0/rclcpp/include/rclcpp/publisher.hpp#L287-L304)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670550776815-3557735c-5590-4bf9-815c-f43b94758cd6.png#averageHue=%23d5eed5&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=820&id=u6f6ba9e8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=820&originWidth=909&originalType=binary&ratio=1&rotation=0&showTitle=false&size=87451&status=done&style=none&taskId=u5e1935b1-e7f2-46f8-ac75-06eb350d189&title=&width=909)

### 6.3.6-Fields(parameter server)

Fields是events和methods的混合

- 发布数据变更通知

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670566785322-ea881d8f-af9f-4d7b-a34d-a142721a564e.png#averageHue=%23fefefd&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=344&id=uc3942132&margin=%5Bobject%20Object%5D&name=image.png&originHeight=344&originWidth=897&originalType=binary&ratio=1&rotation=0&showTitle=false&size=58260&status=done&style=none&taskId=u63c4174f-5526-442f-a685-d17d75124f8&title=&width=897)<br />这个跟发送事件events是一样的，而且是send的第一个版本，所以底层中间件一定会做一份拷贝<br />update调用后，由CM来通知到subscriber端，做通知

- 响应Get()

```cpp
void RegisterGetHandler(std::function<ara::core::Future<FieldType>()> getHandler);
```

- 响应Set()

```cpp
void RegisterSetHandler(std::function<ara::core::Future<FieldType>(const FieldType& data)> setHandler);
```

#### 6.3.6.1-注册Getters

当服务使用方调用Get时候调用的回调函数。<br />注意，Getter回调函数不是必须的，因为Update已经将最新数值传递到CM了，所以Get()可以通过CM知道最新数值，这个回调函数不实现也没有关系。<br />AP规范依然提供这个回调函数的场景是，允许按需获取最新数据，而不是主动Update，因为更新数据可能很耗时，这是Lazy Calculation，当你要的时候我才计算。<br />当field配置成“on-change-notification”的时候，GetHandler的行为没有什么意义，因为用户根本不会调用Get<br />这种情况下，fields的subscriber可能会得到过期的数据，因为GetHandler延迟计算了。<br />注意一点：CM不会自动将GetHandler返回的数据和on-change-notification事件的数据保持一致！<br />所以GetHandler的函数中没有调用Update，则on-change-notification事件的数据是不一样的。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670568644050-789d2089-d32b-4119-904f-18975af6822e.png#averageHue=%23f1e7e3&clientId=u84a5f9a1-b246-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=297&id=ub5e70150&margin=%5Bobject%20Object%5D&name=image.png&originHeight=297&originWidth=944&originalType=binary&ratio=1&rotation=0&showTitle=false&size=119336&status=done&style=none&taskId=u0a7bedcd-bb86-4284-8e57-3e103d6e428&title=&width=944)

#### 6.3.6.2-注册Setters

当服务使用方调用Set时候调用的回调函数，与Getter相反，Setter一定需要。<br />sethandler接受一个数值，返回一个数值。语义是：返回的数值是最终生效的数值。这样允许服务方检查client设置的数值。<br />注意：sethandler中没有必要显式调用Update()，CM已经帮你做了。

#### 6.3.6.3-确保SetHandler存在

由CM确保，如果OfferService了，但是没有注册setHandler，则CM会报错

#### 6.3.6.4-确保数值合法性

服务实现方需要确保数值是合法的<br />ROS中Parameter的实现，匹配上述的需求

## 6.4-Runtime

是一个全局性的单例

# 7-DataTypes on ServiceInterfaceLevel

## 7.1-Optional

std::optional，按照std来用就行了

```cpp
/**
* \brief Data structure with optional contained values.
*/
struct BatteryState {
Voltage_t voltage;
Temperature_t temperature;
ara::core::Optional<Current_t> current;
ara::core::Optional<Health> health;
};
```

# 8-RawDataStreamingInterface

略

## 8.1-概述

ARA::COM基本上是按照SOA的方式来通信的，这对于实现灵活的架构来讲很有帮助。<br />在ADAS应用设计过程中，还是会有提供raw binary data streams的场景，在这种场景下，SOA会带来额外的开销，而且传感器可能没有其他数据可以发送，除了raw data。<br />RawDataStreamingInterface的特点就是允许不带任何数据类型，这样的好处是高效，而且一般是基于sockets来实现。<br />在ARA架构中，RDS API是静态的，所以不能被发现。

# 9-Appendix

## 9.1-Serialization

[https://isocpp.org/wiki/faq/serialization](https://isocpp.org/wiki/faq/serialization)<br />Serialization有多个同义词：Marshal混淆, Encode编码<br />串行化的意思是将内存中的数据结构转化为一种标准格式，通过网络传递给对端。这是一种公认的格式，所以叫做DateExchangeFormat，将内存中的数据表示转化为适用于网络传输的格式。具体实现需要考虑大小端，数据对齐等等。<br />CP是不需要串行化的，基于如下的假设：

- 在所有是SWC中，大小端是确定的
- 在所有SWC中，数据对齐规则是确定的
- 数据交换在内存中是连续的

在AP中需要考虑串行化，上述假设是不成立的，串行化方法完全取决于供应商实现<br />_比如ROS的串行化实现由供应商决定，而且是不开放源码的_<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670909309850-78e9c14f-cb78-4692-a7e0-a4e2b210d91b.png#averageHue=%232c333e&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uecd8b4e8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=441&originWidth=1008&originalType=binary&ratio=1&rotation=0&showTitle=false&size=106612&status=done&style=none&taskId=u6db3a042-c796-406e-bd82-da6046aab92&title=)

### 9.1.1-零拷贝的考虑

减少拷贝次数是中间件实现方要考虑的重点。zero-copy用来描述这个。<br />在AP中，由于是进程内部的虚拟内存分布，所以至少需要一次拷贝。<br />有些中间件能够将这次拷贝去掉，通过设置SHM空间的方式。<br />不过需要注意的是：如果无法解决串行化的问题，也无法从SHM上得到好处，因为只要存在串行化，则拷贝一定会存在，所以不会存在真正的零拷贝。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670575539701-2b96727a-6c9b-4b61-9de0-49be26f63974.png#averageHue=%23ede8e3&clientId=u1c943a25-34b3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=154&id=u533cb65c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=154&originWidth=936&originalType=binary&ratio=1&rotation=0&showTitle=false&size=73459&status=done&style=none&taskId=u156c9419-1c52-4e38-97b6-f9596df265a&title=&width=936)<br />[https://github.com/ZhenshengLee/ros2_shm_msgs/issues/25#event-7984346528](https://github.com/ZhenshengLee/ros2_shm_msgs/issues/25#event-7984346528)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670917291914-a6c75515-abda-4af3-a277-20c61621684d.png#averageHue=%23fef7f6&clientId=ub7e7be5a-b131-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=428&id=u6b353b3d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=571&originWidth=729&originalType=binary&ratio=1&rotation=0&showTitle=false&size=81131&status=done&style=none&taskId=u875778e0-daf3-48bc-9a4c-7f5615d1958&title=&width=547)

### 单机是否需要串行化？

Intra-machine，multi-core的情况下，不需要考虑大小端，数据对齐

- Intra-process不需要
  - 传递所有权即可
- socket based IPC需要吗？
  - 需要，会影响通信性能
- SHM based IPC需要吗？
  - 不需要
  - 某些数据结构在内存中不是连续的，不过IDL不一定支持复杂结构如std::map

## 9.2-ServiceDiscovery实现策略

ara::com要求供应商实现服务发现机制，ara::com只定义API，包括FindService, OfferService and StopOfferService，底层实现和协议都是放开的。在这里提出AP node或AP SWC的概念，当AP node 提供服务的时候，服务发现协议就在运行。

### 9.2.1-集中式和分布式的发现机制

集中式的发现机制，往往有一个中心实体（一个后台进程）来维护所有SOA实体的关系：

- 维护所有service instances的地址信息
- 响应所有instance的服务发现请求，包括all FindService, OfferService and StopOfferService
- 响应所有通过SOME/IP协议传递的网络服务请求，SOME/IP Offer Service，SOME/IP Find Service
- 将网络上的所有实体信息广播出去，用于服务发现

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670588230036-b40f1308-1cf3-4017-bf58-5af93a08e506.png#averageHue=%23f9f6f4&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=598&id=u371eda10&margin=%5Bobject%20Object%5D&name=image.png&originHeight=598&originWidth=903&originalType=binary&ratio=1&rotation=0&showTitle=false&size=102488&status=done&style=none&taskId=u1a7926e4-5141-4018-bf96-225c4f8f0e0&title=&width=903)<br />分布式的发现机制，没有一个中心化的后台进程维护所有信息。而是有一个广播机制，每一个成员的加入，都会向所有成员广播自己的信息。这样的好处在于不依赖于一个特定的后台进程，避免单点失效。<br />但是，对于通过SOME/IP传递的服务注册和查询报文，需要一个代理实例，因为有专门的端口是用来接受SOME/IP SD信息的，所以需要一个专门的代理进程来处理SOME/IP报文。（接受外部的）<br />同时还需要一个实例来做内部服务协议信息和网络服务协议信息的交换工作，这个进程一般也是Bridge进程来做。（内部发送出去）<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670588613087-9cf781ad-115d-484f-9cdf-655afacf59d0.png#averageHue=%23f9f6f4&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=590&id=u3e7af289&margin=%5Bobject%20Object%5D&name=image.png&originHeight=590&originWidth=913&originalType=binary&ratio=1&rotation=0&showTitle=false&size=104427&status=done&style=none&taskId=u8fa4c62c-c273-4d56-8bd8-a0ff361bfd8&title=&width=913)

### ROS2中的服务发现机制

#### DDS默认发现机制带来性能问题

分布式的发现机制带来广播风暴问题

- DDS以Participant参与者为单位进行服务发现
  - ROS2从Foxy开始，Node与Participant不是一对一的关系，Process（Context）和Participant是一对一的关系
- 进程数量多导致广播风暴
  - ROS2 daemon就是缓解广播风暴导致的问题而产生的措施
- 多网络主机环境下的广播风暴更为严重

#### Zenoh解决方案缓解网络间服务发现的性能问题

[https://zenoh.io/blog/2021-03-23-discovery/](https://zenoh.io/blog/2021-03-23-discovery/)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670589028876-8a7d1bef-5b80-4600-9b3e-d784fc8cfed5.png#averageHue=%23f6f5f5&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=603&id=ud6bb7919&margin=%5Bobject%20Object%5D&name=image.png&originHeight=603&originWidth=1152&originalType=binary&ratio=1&rotation=0&showTitle=false&size=69693&status=done&style=none&taskId=uc570b739-7ffb-4666-9341-62434f2bdfb&title=&width=1152)

#### FastDDS的解决方案

[https://fast-dds.docs.eprosima.com/en/latest/fastdds/ros2/discovery_server/ros2_discovery_server.html](https://fast-dds.docs.eprosima.com/en/latest/fastdds/ros2/discovery_server/ros2_discovery_server.html)<br />搞一个中心化的DiscoveryServer来缓解广播风暴<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670589187256-7aea9888-24ed-4534-854f-7d5dfbeec3f4.png#averageHue=%23f9f4ef&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=388&id=u1e077a68&margin=%5Bobject%20Object%5D&name=image.png&originHeight=388&originWidth=953&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36495&status=done&style=none&taskId=u80926d9c-7ca8-4a3c-ac3d-58c201f05da&title=&width=953)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670589695748-10f885bb-cd84-485c-a4aa-16776f9f6af2.png#averageHue=%23f9f9f9&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=467&id=ud8d5dcb4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=467&originWidth=608&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23571&status=done&style=none&taskId=u31cc1507-9380-4602-904e-4f79b9ec19c&title=&width=608)

#### 动态改静态的解决方案

可以静态的配置所有的节点间关系，但是不现实。

## 9.3-Multi-Binding

Multi-Binding指的是对于某个Proxy/Skeleton，底层传输方式不一样，这样做的原因是：

- Proxy使用不同的传输机制来与不同的Skeleton进行交互：不同的service  instance由于部署的原因支持不同的机制
- 对应的，Skeleton也可以提供不同的Transport/IPC机制供Proxy使用，不同的Proxy使用对应的binding即可

### 9.3.1-简单的Multi-Binding场景

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670591745689-400dd578-f97f-4fca-b469-480279739575.png#averageHue=%23faf5f3&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=706&id=u1f3c005d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=706&originWidth=879&originalType=binary&ratio=1&rotation=0&showTitle=false&size=77702&status=done&style=none&taskId=u073a9a6d-a184-4ca1-91ef-84d474eab53&title=&width=879)<br />In this example, which only deals with node local (inside one AP product/ECU) communication between<br />service consumers (proxy) and service providers (skeleton), there are two instances of the same proxy class on the service consumer side. You see in the picture, that the service consumer application has triggered a “FindService” first, which returned two<br />handles for two different service instances of the searched service type. The service consumer application has instantiated a proxy instance for each of those handles. Now in this example the instance 1 of the service is located inside the same adaptive application (same process/address space) as the service consumer (proxy instance 1), while the service instance 2 is located in a different adaptive application (different process/address space).<br />在这里，不同的Handle代表不同的通信通路。<br />进程内通信的Method应该简化为纯函数调用，因为在一个进程内。<br />而跨进程通信，是一个IPC，典型的使用pipes，sockets，SHM的方式触发相关控制。<br />不管怎么样，在proxy眼里，通过Handle透明地调用方法，底层中间件负责具体实现，实现透明服务。<br />但是Proxy怎么知道要走哪条通路呢？所以在Handle里面必须包含一些信息，Handle里面的信息谁来填充？CM，一般来讲是在注册的时候提供信息。一般来讲，一个AP厂家只会实现一种高度优化过的IPC机制，比如说UDS，那么提供到Handle里面的信息就是filedescriptor。ServiceInstance1和ServiceInstance2在OfferService时候就会提供UDS Port信息，然后Proxy在FindService的时候就会找到两个filedescriptor，然后就会发现ServiceInstance1的Handle是可以优化成MethodCall的。所以Hanlde里面可能还要包含addressing信息。<br />上述的优化机制只能说是一种可选的方法，在FastDDS等其他中间件设计中，每家都有每家的策略。

### 9.3.2-Local/Network Multi-Binding

更加接近真实的例子，前面的例子不必要，只是针对Intra-process做了额外的优化。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670642334929-277d322e-9d33-4c81-b4aa-99f37de75533.png#averageHue=%23faf8f6&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=533&id=ua818287e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=533&originWidth=867&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56651&status=done&style=none&taskId=u291d5212-6c72-417b-b7d5-eab86d525d9&title=&width=867)

### 9.3.3-Typical SOME/IP Multi-Binding

上面的场景不是真的，因为不会SOME/IP的端口数目是确定且优先的，必须有一个代理做统管，否则端口资源不够用。<br />所以一般会有一个bridge做路由，统一处理SOME/IP报文，这是一种多路复用策略。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670642696157-20c8f634-bdce-483b-b8d4-ca9df9c1373a.png#averageHue=%23fbf8f6&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=530&id=u9c44dba4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=530&originWidth=883&originalType=binary&ratio=1&rotation=0&showTitle=false&size=53837&status=done&style=none&taskId=u26e6a8fc-0740-4a8b-9990-e3d12d64c68&title=&width=883)<br />从上图可以看书，SOME/IP协议使用范围很狭窄，仅限于Inter-machine的SOA，在Intra-machine的场景，还是会优先用PointerBasedCall，或者SHM-IPC。<br />注意：绿色和蓝色使用的传输机制是不一样的，绿色使用的是vendor-specific-IPC，蓝色使用的是SOME/IP，vendor-specific指的是具体IPC机制和Serialization都不一样（**这里AP规范好像暗示IPC都要做串行化**），在这里，vendor所指定的串行化协议可能和SOME/IP的串行化协议不一致，但是如果不一致，则在bridge侧会导致额外的拷贝开销。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670653085859-c9474396-b365-40b0-aa62-873db3548ce1.png#averageHue=%23ede7e2&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=550&id=ueadeb0b5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=550&originWidth=918&originalType=binary&ratio=1&rotation=0&showTitle=false&size=259488&status=done&style=none&taskId=u16cb218a-b5b5-4709-b514-78cd38b7c56&title=&width=918)<br />对于SOME/IP bridge和ServiceRegistryDaemon两个组件，一般都是会集成到一个进程中实现。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670653074159-c2ef46cc-2176-4f11-84cd-4e8c5206ceec.png#averageHue=%23eee9e4&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=239&id=ub6810e7c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=239&originWidth=922&originalType=binary&ratio=1&rotation=0&showTitle=false&size=108102&status=done&style=none&taskId=u4e216202-c0b4-4c0f-a40e-55e05080a0b&title=&width=922)

### ROS2对Multi-Binding的考虑

ROS2也会遇到Intra-process，Inter-process，Inter-host的通信

- 借助DDS的以数据为中心的机制，无感解决该问题。
- 从另一个角度说，ROS2没有考虑Multi-Binding的问题，只考虑基于DDS的通信，由DDS来处理各种情况
  - 进一步，考虑到通信效率，单独处理了Intra-process的情况，并在Node机制下提供了灵活的配置，只有配置了intra-process-comm的才走进程内通信

### ROS2对通信的优化

Intra-process下只对Topic有优化，Service和Action没有优化（没有实现AP描述的优化成MethodCall），Parameter是基于Service的<br />Inter-process的各路通信取决于DDS实现<br />Inter-machine的通信取决于DDS实现，基于UDP的DDS-RTPS通信

### 各种场景下的串行化

Intra-process: <br />Intra-machine, Inter-process: <br />Inter-machine:

### SOME/IP-DDS-Zenoh

从SOME/IP的使用范围来看，SOME/IP的地位和Zenoh更像，是用于连接本地网络，本地网络通信方式是高效率的SHM-IPC，只有在Inter-host的场景下采用网络通信，而SOME/IP是针对AP-SOA(Proxy/Skeleton架构)数据传输专门优化的数据承载协议，为什么说专门优化过呢，那是因为数据报文的设计，与上层应用层的设计的语义是straight-forward的，没有多余的开销进行转换。<br />这个项目给出了一些性能比较<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670848639058-419d2c5d-8fb5-44d5-85c8-e9fcf1076cb2.png#averageHue=%23fbfbf2&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=218&id=u34cc9bd4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=218&originWidth=855&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24918&status=done&style=none&taskId=ub433c799-36ba-437a-b8e6-d4cad0fb25a&title=&width=855)<br />[https://github.com/YuanYuYuan/ros2-dds-zenoh-comparison](https://github.com/YuanYuYuan/ros2-dds-zenoh-comparison)

### ROS2-AP-Bridge的设计

Intra-process的场景过于复杂，暂时不考虑<br />![企业微信截图_16693464655372.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670652370255-7aad52a9-5398-4c8d-ad8a-d2e604186513.png#averageHue=%23f6f5f4&clientId=ufe2fbbfa-64d3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=540&id=u1b78f693&margin=%5Bobject%20Object%5D&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16693464655372.png&originHeight=1080&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=294511&status=done&style=none&taskId=ufc6c0cea-498c-4e17-a3c9-08a31b83d4e&title=&width=960)

#### Intra-machine

要用到SOME/IP吗？

#### Inter-machine

要用到SOME/IP

## 9.4-ara::com与AP组件的关系

### 9.4.1-ServiceInterface

服务接口，在ROS中叫做IDL，由于ROS解耦更强烈，所以IDL只定义了数据的类型，没有定义交互的类型，也定义了交互的类型，使用msg src action的后缀。<br />ara::com的proxy和skeleton类是基于SI生成的。

### 9.4.2-SoftwareComponent

软件组件是从复用的角度来定义的，在AP中有多重组件类型，最重要的叫做AdaptiveApplicationSwComponentType<br />组件模型的对外接口叫做ports，Ports通过SI来定义，P-Ports，Provided Ports，R-Ports，RequiredPorts<br />ROS的SI定义不是基于（组件提供的）服务，而是基于运行在系统中的数据。所以ROS的SI是pub and sub，本质上讲也是一样的。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670849861435-4387b3d4-b1d9-4f31-8eb9-2a8681822f6a.png#averageHue=%23fcfaf4&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=857&id=u3d31c42b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=857&originWidth=1202&originalType=binary&ratio=1&rotation=0&showTitle=false&size=130906&status=done&style=none&taskId=uf383ca34-9a50-4c63-88f2-66f060e6b02&title=&width=1202)

### 9.4.3-AP Executables and Process

Proxy和Skeleton的类都可以实例化多份到具体的executable中<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670850108885-d5347f27-09e9-4714-97eb-d56cae01b58f.png#averageHue=%23faf8f3&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=843&id=u5a1980d8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=843&originWidth=1224&originalType=binary&ratio=1&rotation=0&showTitle=false&size=132396&status=done&style=none&taskId=u16c4e914-1bd7-4dc6-86c7-b8edbce9518&title=&width=1224)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670850149089-bff7f24e-4454-4f61-a284-f9ff186a696c.png#averageHue=%23e8e5e1&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=355&id=u71540235&margin=%5Bobject%20Object%5D&name=image.png&originHeight=355&originWidth=1251&originalType=binary&ratio=1&rotation=0&showTitle=false&size=181342&status=done&style=none&taskId=ua6b1e4c5-0d41-46d7-ab47-d1268e97800&title=&width=1251)<br />composit的概念。ROS也有了。<br />AP 中App一般包含一个executables，executable指的是部署层面的概念<br />Process指的是执行层面的概念，启动之后的App就是进程。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670850690438-5ece65ba-0e9f-4e9f-a3ad-35b106f767ec.png#averageHue=%23f9f8f6&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=660&id=uacae8c0d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=660&originWidth=1115&originalType=binary&ratio=1&rotation=0&showTitle=false&size=87778&status=done&style=none&taskId=u3a9b0582-0b7c-4a9b-9c3b-593360787be&title=&width=1115)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1670850707876-28864a1d-6622-41f4-8b89-3fd774c13714.png#averageHue=%23ece9e6&clientId=u601296f8-baa8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=214&id=u1742e70a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=214&originWidth=1248&originalType=binary&ratio=1&rotation=0&showTitle=false&size=92584&status=done&style=none&taskId=uf3db6c4f-1291-4aea-ad9c-6e22e5f15f7&title=&width=1248)<br />ROS2允不允许deplyment twice? 允许，在launch层面就允许了。

### 9.4.4-Usage of meta-model identifiers within ara::com based application code

略

### 9.5.5-Abstract Protocol Network Binding Examples

略
