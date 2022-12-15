# 概述

This specification describes the AUTOSAR Adaptive Platform (AP) design. The purpose of this document is to provide an overview of AP but is not to detail all the elements of AP design. It is to provide the overall design of the AP and key concepts for both AP users and AP implementers.<br />介绍平台设计的基本概念

# 3-技术范围和方法

## 3.1-智能ECU的趋势

主要就是应对自动驾驶在车上的应用，传统的CP是deeply-embedded的，已经不适应自动驾驶的软件开发需要

## 3.2-技术驱动力

两个主要的技术驱动力：一个是以太网，一个是处理器<br />车载以太网的数据传输带宽不断增加，对比传统的CAN总线通信，传输的数据越来越大，点对点通信场景增加。<br />尽管CP也支持以太网，但CP设计的初衷是支持传统的通信方式，对于基于以太网的通信，没有做优化。<br />处理器算力的增加，使得车辆能够具备人工智能能力，在CP中已经使用到多核处理器，但是这远远不够。<br />现代的计算场景，需要使用到众核处理器，需要使用到GPGPU，FPGA，以及专用ASIC加速器，这些计算单元的算力比MCU大很多个数量级。<br />核心数量的增加，已经超过了CP当初的设计，CP当初只是用来应对单核MCU。但是当下的场景已经成为异构计算，或者说是HPC高性能计算，已经远远超过了CP的范畴<br />同时，处理器之间的高速通信也已经实现，随着越来越多的ASIC集成到一个Chip上，处理器之间数据传输能力已经高出传统ECU几个数量级。比如已经有的技术Network-on-Chip(NoC)，片上网络。

## 3.3-AP的主要特点

前述章节已经讲述了AP的驱动力，在这些驱动力下，AP必须具备一些特征，来处理上述驱动力带来的挑战。<br />这些特征包括以下的新技术。

### 3.3.1-Cpp

From top-down, the applications can be programmed in C++. It is now the language of<br />choice for the development of new algorithms and application software in performance<br />critical complex applications in the software industry and in academics. This should<br />bring faster adaptations of novel algorithms and improve application development productivity if properly employed.<br />对于有性能需求的软件和算法而言，cpp是主流的选择，如果使用得当，可以加快软件开发效率。

### 3.3.2-SOA

To support the complex applications, while allowing maximum flexibility and scalability in processing distribution and compute resource allocations, AP follows serviceoriented-architecture (SOA).<br />为了支持复杂的应用，同时具备最大的灵活性，扩展性，在分布式架构和资源受限系统中，AP遵循SOA的架构体系。<br />**The SOA is based on the concept that a system consists of a set of services, in which one may use another in turn, and applications that use one or more of the services depending on its needs.**<br />SOA的概念在于，系统由若干Service组成，Service之间有使用的关系，应用程序根据需要使用一些Service。<br />ROSSOA的概念在于，系统由若干Node组成，Node之间有使用的关系，应用程序根据需要使用一些Node。<br />**关键点在于Service之间的通信和Node之间的通信规约，Service之间的通信是紧耦合的，Node之间的通信是松耦合的。**<br />SOA也具备SOS的特点，System-of-System，即一个Service内部可能是一个AP实例，并由可能<br />ROS也具备SOS的特点，即一个Node可内部可能是一个ROS实例，采用的是bridge模式，<br />**The application code is the same in both cases the communication infrastructure will take care of the difference providing transparent communication**. **Another way to look at this architecture is that of distributed computing, communicating over some form of message passing. At large, all these represent the same concept. This message passing, communication-based architecture can also benefit from the rise of fast and high-bandwidth communication such as Ethernet.**<br />应用程序保持一致，通信基础设施来保证数据的透明传输。<br />另一个视角就是基于消息传递的分布式计算通信，这是一种message-passing, communication-based的体系结构。<br />而ROS的体系结构与此概念一致。

### 3.3.3-并行处理

并行处理，一方面是来自于分布式计算，分布式计算其实就是多进程的分布式架构，功能分布在多个进程中，这个架构很常见。<br />另一个方面是并行计算，GPGPU以及FPGA提供的超算能力。

### 3.3.4-遵循已有的标准

复用和兼容已有的标准，对于标准已提供的特性，AP尽量做到reuse。<br />**这其实导致了AP的包袱比较重。**

### 3.3.5-安全

AP新引入的特性应该保证安全。<br />To cope with the challenge,<br />AP combines architectural, functional, and procedural approaches. The architecture is based on distributed computing based on SOA, which inherently makes each component more independent and free of unintended interferences, dedicated functionalities to assist achieving safety and security, and guidelines such as C++ coding guideline, which facilitates the safe and secure usage of complex language like C++, for example.<br />AP为了解决安全难题，在架构上，功能上，和流程上做了充分准备。架构上引入SOA，组件尽可能做到功能解耦；在功能上做到符合功能安全要求；在流程上推出cpp guidline。

### 3.3.6-受管控的不确定性(Planned dynamics)

在部署方面，为了尽量提升软件开发和集成的效率，AP支持动态部署应用程序，这样就可以支持快速迭代的软件流程。<br />在安全方面，AP允许集成商谨慎限制软件的动态行为，来减少非预期的风险，以做到安全认证。动态行为约束在ExecutionManifest中。<br />尽管如此，在执行期间，程序的计算资源的动态获取，只允许在configured阶段进行。

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

## 3.4 外部互联

互联通信方式主要是SOME/IP<br />一个AP和CP共存的参考架构<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669691053626-47a7034d-adb2-4c6d-ad58-c80addfb520f.png#averageHue=%23f0e2ce&clientId=ufd4aa3ad-91ea-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=657&id=u49d52ddc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=657&originWidth=949&originalType=binary&ratio=1&rotation=0&showTitle=false&size=121235&status=done&style=none&taskId=uc93912b0-4973-4bfd-af83-b1497221468&title=&width=949)

# 4-架构

略

# 5-OS操作系统

## 5.1-概述

操作系统主要负责AP平台所有应用程序的运行时调度，资源管理（包括内存管理策略和时间约束策略），IPC网络通信。<br />OS和执行管理EM一起负责平台的初始化，基于OS的功能来启动和关闭各个应用程序。<br />AP并不会针对高性能处理器定义一个新OS，而是定义一个execution context以及OSI接口。<br />OSI规范包括API，是ARA的一部分，ARA则是the standard application interface of Adaptive Application。OS会提供其他接口，比如创建进程，而这个接口是EM所需要的。不过这个接口不在ARA中定义，而是平台相关的。<br />OSI同时提供c和cpp接口。对于C接口，主要包括POSIX standard, namely PSE51 defined in IEEE1003.13。对于Cpp接口，主要是defined in the C++ Standard and its Standard C++ Library。

## 5.2-POSIX

## 5.3-调度

The Operating System provides multi-threading and multi-process support. The standard scheduling policies are SCHED_FIFO and SCHED_RR, which are defined by the POSIX standard. Other scheduling policies such as SCHED_DEADLINE or any other operating system specific policies are allowed, with the limitation that this may not be portable across different AP implementations.

## 5.4-内存管理

One of the reasons behind the multi-process support is to realize ’freedom of interferences’ among different Functional Clusters and AA. The multi-process support by OS forces each process to be in an independent address space, separated and protected from other processes. Two instances of the same executable run in different address spaces such that they may share the same entry point address and code as well as data values at startup, however, the data will be in different physical pages in memory.<br />对多进程管理的支持，资源隔离，虚拟内存管理。

## 5.5-设备管理

Device management is largely Operating System-specific. Intentionally, the Adaptive Platform Foundation favors the creation of services to expose the main system functionalities. While there is no current plan to standardize the concrete APIs of device drivers themselves, higher-level functionality fulfilled by such drivers may be standardized through Adaptive Platform Services.

## 5.6-网络

The main interconnection mechanism between multiple Machines, as well as with other sensors is expected to be based on Ethernet. Consequently, the use of TCP/IP- and UDP/IP-based protocols is clearly described. It is therefore expected that the Operating<br />System will provide such a networking stack. Applications will transparently benefit from the networking support by using Communication Management. As part of the Adaptive Platform Foundation, additional features like VLAN, IPSEC and more are enabling secure communications within and across systems.

# 6-执行管理

# 7-StateManagement

# 8-通信管理

## 8.1-概述

CM负责应用程序之间在实时分布式系统环境中的通信。

## 8.2-ServiceOrientedCommunication

The notion of a service means functionality provided to applications beyond the functionality already provided by the basic operating software.<br />在这里服务的概念表示提供给客户程序的功能，在基本功能基础上的封装。<br />CM的目的就是提供机制来保证服务使用的透明性，方便使用服务，不管是Intra-machine还是Inter-machine通信<br />A service consists of a combination of

- Events事件
- Methods方法
- Fields字段

Communication paths between communication partners can be established at design-, at startup- or at run-time. An important component of that mechanism is the Service Registry that acts as a brokering instance and is also part of the Communication Management software。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669701781204-cf2b8422-4eb7-4bc7-bc96-c2ba5b5827de.png#averageHue=%23d6cec4&clientId=u3324c857-ac3c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=448&id=u09426ce7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=448&originWidth=573&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43665&status=done&style=none&taskId=ufb1fb65f-b2c8-4999-b00d-618fe512c54&title=&width=573)<br />ROS的特点是broker-less，即采用了分布式发现机制，跟交换机二层广播协议类似，需要发现的时候就广播。

## 8.3-LanguageBinding and NetworkBinding

CM提供一种标准的方式来向app开发者展示可以用的服务，针对app，有一个layer叫做LanguageBinding，在下层叫做NetworkBinding。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669702378239-35d746ae-510c-4746-91b2-08cbb6ea6d0c.png#averageHue=%23e8cfaa&clientId=u3324c857-ac3c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=508&id=ubac85d02&margin=%5Bobject%20Object%5D&name=image.png&originHeight=508&originWidth=921&originalType=binary&ratio=1&rotation=0&showTitle=false&size=119033&status=done&style=none&taskId=ud9ed3334-6a03-4735-b1e7-2133d057e91&title=&width=921)<br />The Language Binding defines how the methods, events, and fields of a service are translated into directly accessible identifiers by using convenient features of the targeted programming language. Performance and type safety (as far as supported by the target language) are the primary goals. Therefore, the Language Binding is typically implemented by a source code generator that is fed by the service interface definition.<br />语言绑定用于定义，方法，事件和字段如何在特定编程语言中表示，重点考虑的是性能和类型安全。所以一般是由一个code_gen来做的。<br />The Network Binding defines how the actual data of a configured service is serialized and bound to a specific network. It can be implemented based on Communication Management configuration (interface definition of the AUTOSAR meta model) either by interpreting a generated service specific recipe or by directly generating the serializing code itself. Currently, Communication Management supports SOME/IP, DDS, IPC (Inter-Process-Communication or any other custom binding), Signal PDU (SignalBased Network binding) and Signal-Based Static Network binding.<br />网络绑定用于定义数据如何串行化，通过指定的网络通路发送出去。当前支持的通路包括SOME/IP，DDS，IPC，Signal PDU，Signal-based Static Network binding.<br />在ROS中LanguageBinding叫做rosidl_typesupport_cpp和rosidl_typesupport_c，由于只支持DDS，NetworkBinding在ROS2中细分为支持多种DDS，叫做rosidl_typesupport_fastrtps_c和rosidl_typesupport_fastrtps_cpp，这些统称为rosidl_typesupport

## 8.4-Generated Proxies and Skeletons of C++ Language Binding

A generator that is part of the development tooling for the Communication Management software generates C++ classes that contain type safe representations of the fields, events, and methods of each respective service.<br />On the service implementation side, these generated classes are named Service Provider Skeletons. On the client side, they are called Service Requester Proxies.<br />For Service Methods, a Service Requester Proxy provides mechanisms for synchronous (blocking the caller until the server returns a result) and asynchronous calling (called function returns immediately). A caller can start other activities in parallel and receives the result when the server’s return value is available via special features of the Core Type ara::core::Future<br />对于Methods，则提供同步调用和异步调用，异步调用也需要等待future，来判断是否调用成功。<br />Methods对应ROS2中的Service。<br />在ROS2中其实也有Service Provider Skeletons和Service Requester Proxies，不过这些都作为公共部分，定义在rclcpp中了，不需要由code_gen生成。<br />Service Requester Proxies由Clients直接使用，而Service Provider Skeletons由Servers通过集成实现虚方法的方式来实现。<br />Events和Fields一起构成ROS2中的Topic。<br />The interfaces of ara::com can also provide proxies and skeletons for safety-related E2E protected communication. These interfaces are designed that compatibility to the applications is assured independent whether E2E protection is switched on or off.<br />ara::com api是用来保证E2E通信的，其也产生特定的Proxy和Skeleton。

## 8.5-静态和动态配置

这个好懂

## 8.6-服务契约配置管理

Client和Service有一个契约，这个契约定义会发生变化，AP支持contract的版本定义，称为不同的服务版本，AP的服务发现协议是支持版本向后兼容的。<br />在ROS2中，msg的变更是很重大的变更，需要重新编译代码，而且不向后兼容。这是个很大的问题，也是CyberRT使用Protobuf的原因。

## 8.7-RawDataStreamingInterface

Besides the Service Oriented Communication, the Communication Management also provides a standalone API for processing raw binary data streams towards an external ECU, e.g. a sensor in an ADAS system. The API is static and implements functionality for a client application to establish a communication channel to a server, and for a server application to wait for incoming connections from a client. The API provides functionality for both clients and servers, to destroy a communication channel, and to read and write raw data (a stream of bytes) over the communication channel. The Raw Data Stream channels can be configured by an integrator by applying deployment information, containing e.g. network endpoint information and selected protocols. Currently, TCP/IP sockets shall be used as a transport layer, but other alternatives can be added in the future. The Raw Data Stream interface is available in the namespace ara::com::raw.
