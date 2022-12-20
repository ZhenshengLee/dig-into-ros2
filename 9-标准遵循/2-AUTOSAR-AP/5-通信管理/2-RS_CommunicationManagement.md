# 总结

## rclcpp与CM区别

| 领域 | CM | rclcpp | 备注 |
| --- | --- | --- | --- |
| 设计思路 | CM以Service为核心，Methods，Events，Fields是依附于Service的 | rclcpp以数据为核心，围绕如何处理数据来设计，没有Service这么大的概念，可以包括任意数量的Topic, service, action | Cyber与ROS一致 |
| 组件行为 | 服务之间的行为协作没有定义，<br />**App如何提供服务，没有实现指南**<br />**(AP对回调调度没有任何规定，没有executor的概念)**<br />(规范只是指出要支持回调) | topic, service, action的行为交互<br />**以Node的方式组织在一起，提供协作机制**<br />**（executor作为执行回调的机制）**<br />**（callback-group作为回调调度的机制）**<br />（在ROS1中Node与进程有一对一关系，在ROS2中取消） | Cyber对回调任务的调度管理更加细致，基于静态配置？ |
| 部署 | 一般来讲一个Executable包含一个Service，对应一个SWC，部署到一个进程 | 一个Executable对应一个进程<br />一个Executable对应一个或多个SWC | ~~Cyber与ROS一致~~ |

总的来讲，rclcpp理念和实现都比CM要先进

## SOME/IP与DDS的区别

SOME/IP和DDS都支持分布式系统实现pub/sub通信模式和request/reply通信模式，但是他们有很大的不同。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671449992721-33cc63fc-81a2-410f-a7f4-173b06245292.png#averageHue=%23fbebd3&clientId=ub44ad8cb-df60-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=507&id=u8d1f50ed&margin=%5Bobject%20Object%5D&name=image.png&originHeight=507&originWidth=1005&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60767&status=done&style=none&taskId=u6ff5e259-0e82-4e22-88b6-17edfa1c9e0&title=&width=1005)

|  | SOME/IP | DDS | 备注 |
| --- | --- | --- | --- |
| 首次发布 | 2011年 | 2004年 |  |
| 成熟时间 | 2014年宝马量产 | 2017引入DDS-RPC，支持SOA |  |
| 定位 | SOME/IP是AUTOSAR规范的一部分，描述了 serialization protocol, service discovery, and a transformer for integration with Classic AUTOSAR | DDS面向IoT，为分布式实时系统而设计 | 分布式实时系统站位高<br />不兼容CP没有历史包袱 |
| 领域 | 专门为汽车领域应用设计的 | 工业IoT领域，交通，能源，医疗系统，工业自动化，宇航，国防 | DDS从宇航国防系统发展而来 |
| 通信<br />模式 | object-based Service-oriented Architecture | decoupled, data-centric publish subscribe model. Aso referred as a “databus” pattern<br />DDS不需要绑定到ServiceInstance | DDS is much more dynamic than SOME/IP. |
| API | SOME/IP没有定义标准API，不过AUTOSAR定义了ARA::COM，但是不完整 | DDS针对cpp, java等定义了完整的API | DDS一致性更好 |
| 传输<br />层 | 采用UDP和TCP，CP4.3引入支持大尺寸数据（大于1400bytes）的UDP协议，对于可靠传输使用TCP | 使用RTPS的平台无关协议，底层可以是UDP/TCP/SHM等，支持组播，基于UDP的大尺寸数据和可靠传输<br />还支持自定义custom transport sdk | DDS在传输多媒体数据上有明显优势 |
| 信息<br />安全 | SOME/IP also relies on the transport for security. So to use it securely it becomes required to run on TLS or DTLS. | DDS也可以采用传输层安全策略，TLS或者DTLS，但是DDS提供了DDS-Security规范，不依赖Transport，而且提供了更精细化的控制，在上层设计了访问控制，Domain等安全概念。 | DDS可以做app应用层的安全策略 |
| QoS | SOME/IP only offers one the “reliability” Qos setting used to select UDP versus TCP<br />其他策略需要app自己实现 | DDS 提供大量 QoS policies | DDS具有明显优势 |
| 标准<br />采纳 | 仅AUTOSAR AP |  Industrial Internet Consortium (IIC)<br /> [OpenFMB](https://openfmb.github.io/), [ROS2](https://github.com/ros2/ros2/wiki), [MD PnP](http://www.mdpnp.org/), [FACE](https://www.opengroup.us/face/)<br />[AUTOSAR Adaptive (starting in version 18.03)](https://www.autosar.org/standards/adaptive-platform/adaptive-platform-1803/) | DDS用途更广泛 |
| 主要<br />缺点 | SOME/IP的序列化采用了比较静态的定义方式，比如SOME/IP的Payload中的参数的类型，参数的顺序，字节序等，都是在配置文件中静态定义的，那么应用程序在使用这些类型时，必须要严格遵循配置文件中的定义去解析数据。对TLV的支持将使软件系统进一步解耦，让应用程序以更灵活的方式使用SOME/IP。但是灵活性和高效率往往是鱼与熊掌不可兼得，引入TLV的缺点也是显著的，额外的“标签”信息将占用更多的Payload空间，这会降低带宽的利用率，对实时性有一定影响 | DDS有着更丰富的特性，尤其是对QoS的支持。但是相对于SOME/IP，DDS也有显著的不足。首先，RTPS消息头部十分冗长，这会降低传输效率和实时性。另一方面，汽车作为一个相对封闭的系统，为了降低功耗，经常需要频繁的唤醒和休眠，这就要求系统有非常快的启动速度，而DDS并不是为这种场景设计的，DDS可能必须经过深入的优化才能满足严苛的时间要求。最后，DDS目前只能在Adaptive AUTOSAR框架下运行，Classic AUTOSAR目前并不支持，尽管有厂商使用复杂驱动（DDS）的方式在Classic AUTOSAR平台集成了DDS，但这并不是一种完美的解决方案。 | 都需要深度优化 |

### 通信模式不同

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671449876983-1abfa748-c676-4a50-9c17-d155369c7808.png#averageHue=%23fefdfd&clientId=ub44ad8cb-df60-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=559&id=u84c581e4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=559&originWidth=998&originalType=binary&ratio=1&rotation=0&showTitle=false&size=80742&status=done&style=none&taskId=uaaadbe55-1b23-4fa2-a728-cb177ebe857&title=&width=998)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671449866107-e4ce5b47-d558-45be-b590-a3b47690038d.png#averageHue=%23fefdfd&clientId=ub44ad8cb-df60-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=535&id=uf2e4f761&margin=%5Bobject%20Object%5D&name=image.png&originHeight=535&originWidth=1006&originalType=binary&ratio=1&rotation=0&showTitle=false&size=83376&status=done&style=none&taskId=ucb78cd84-7710-4c7e-a312-ebcfbffd79b&title=&width=1006)

## SOME/IP与DDS的互操作性

是在上层做桥还是在底层做桥？

## iceoryx作为简化的DDS协议

只能做intra-machine通信，不能inter-host

## rclcpp的接口定义

[https://docs.ros2.org/foxy/api/rclcpp/index.html](https://docs.ros2.org/foxy/api/rclcpp/index.html)<br />[https://docs.ros.org/en/humble/p/rclcpp/generated/index.html](https://docs.ros.org/en/humble/p/rclcpp/generated/index.html)

## rclcpp缺失特性的考虑

### idl versioning

[https://groups.google.com/g/comp.soft-sys.ace/c/E0ZqcZpvuzk](https://groups.google.com/g/comp.soft-sys.ace/c/E0ZqcZpvuzk)<br />目前还没有，是否直接使用IDL而不是使用ROSIDL?

## QA

### 是否需要动态发现？

AP明确需要支持dynamic communication<br />It allows establishing communication paths both at run-time, so it can be used to build up dynamic communication with unknown number of participants

# 1-概述

本文档包含AP CM的功能，API，和配置相关的需求项。<br />CM文档包含两个部分：

- ARAComAPI EXP文档，对于ara::com API设计和行为的描述
- 本文档，包含ara::com API的需求

# 2-术语

| Abbreviation / Acronym:  | Description: | 备注 |
| --- | --- | --- |
| CM  | Communication Management |  |
| IP  | Internet Protocol |  |
| SOME/IP  | Scalable service-Oriented MiddlewarE over IP |  |
| TCP  | Transmission Control Protocol |  |
| UDP  | User Datagram Protocol |  |
| E2E  | End-to-end communication protection |  |
| SoC  | Service-Oriented Communication |  |
| SecOC  | Secure Onboard Communication |  |
| DTLS  | Datagram Transport Layer Security |  |
| DDS  | Data Distribution Service |  |
| RTPS  | Real Time Publish Subscribe Protocol |  |
| TTL  | Time To Live |  |
| TLV  | Tag-Length-Value |  |
| RPC  | Remote Procedure Call |  |
| QoS  | Quality of Service |  |
| BOM  | Byte Order Mark |  |

| Term:  | Description: | 备注 |
| --- | --- | --- |
| Callable  | In the context of C++ a Callable is defined as: A Callable type is a type for which the INVOKE operation (used by, e.g., std::function, std::bind, and std::thread::thread) is applicable. This operation may be performed explicitly using the library function std::invoke. (since C++17) |  |
| serializedSample  | A serializedSample is the serialization of a C++ object to an array and consists of the header that is part of e2e protection and the serialized data. |  |
| Service Binding  | Act of connecting a Service Requester to a concrete Service In stance of a Service Provider. |  |
| Multi-Binding  | Multi-Binding describes setups having multiple connections im plemented by different technical transport layers and protocol be tween different instances of a single proxy or skeleton class, e.g.:<br />• A proxy class uses different transport/IPC to communicate with different skeleton instances.<br />• Different proxy instances for the same skeleton instance uses different transport/IPC to communicate with this in stance: The skeleton instance supports multiple transport mechanisms to get contacted. |  |

# 4-约束和假设

# 7-功能定义

## 7.1-概述

FC向App提供服务Service。CM也是一个FC，是ARA的一个组件。主要负责在App之间构建和管理通信，向App提供统一的通信服务，向下屏蔽不同的通信方式，通信协议，计算平台。所有通信支持在design, start-up, run-time三个阶段建立。<br />本文档的内容包括API语法，API之间的关系和语义，API的使用。这个规范不限制软件架构和具体的实现。

### 7.1.1-架构概念

CM可以分为以下四个部分：

- Language binding
- End-to-end communication protection
- Communication / Network binding
- Communication Management software

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671159180154-684d85ef-50c8-411b-9435-3dcb97f45a55.png#averageHue=%23839064&clientId=u8b239c67-225a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=605&id=ub293d1a2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=605&originWidth=833&originalType=binary&ratio=1&rotation=0&showTitle=false&size=47777&status=done&style=none&taskId=ubdaae682-7f1d-439f-aac1-1fe0460790d&title=&width=833)<br />在CM的上下文中，定义了以下接口

- Public Application Interface: Part of the Adaptive AUTOSAR API and specified in the SWS. This is the standardized ara::com API.
  - rclcpp中也定义了公共的API
- FC interactions: 与FC进行交互。这一块的接口定义是没有标准化的

对于ARA API的设计，遵循以下约束条件：

- App SWC是独立的，Support the independence of application software components，解耦
- SOC的通信，不依赖特定的通信协议 Use of Service-oriented communication without dependency on a specific communication protocol
- API尽可能简洁，不实现特定的具体用例（往下限制），也不实现高层的模型层次的高层概念（往上限制），API限制在实现核心的通信机制。
- **支持dynamic communication**
  - No discovery by application middleware, the clients know the server but the Server does not know the clients. Event subscription is the only dynamic communication pattern in the application.
  - 中间件不需要主动发现App，而是App去发现，Event Subscription是唯一的动态通信模式
  - Full service discovery in the application. No communication paths are known at configuration time. An API for Service discovery allows the application code to choose the service instance.
  - App去做服务发现，在配置阶段不建立通信通路。App应该能够通过API选择发现到的服务实例
    - 配置阶段不建立通信通路，这个在lifecycle node中有这样的思想
- Support both Event/Callback and Polling style usage of the API to enable classic RTE style paradigms. **To support high determinism demands in case of callbackbased / event-based interaction, there shall be the possibility to avoid uncontrolled context switches.**
  - 支持Event/Callback的方式和Polling的方式来执行。
  - **为了支持在callback模式下的高确定性，需要避免不受控制的上下文切换**
- Support both synchronous callback-based communication and asynchronous communication philosophy.同步和异步
- Support of client/server communication.
- Support of sender/receiver communication with queued semantics where the receiver caches are configurable.
- **Support of selection of trigger conditions for task activation.**
  - 这个不太理解
- Extensions for security。
- **Extensions for Quality Of Service QoS.**
- **Scalability for real-time systems.**
- Support of built-in end-to-end communication protection, where a use-casespecific behavior can be done on top of ARA API.

### 7.1.2-设计决策

ARA API设计决策遵循以下原则

- Proxy/Skeleton模式
  - The (service) proxy is the representative of the possibly remote (i.e. other process, other core, other node) service. It is an instance of a C++ class local to the application/client, which uses the service.
  - The (service) skeleton is the connection of the user provided service implementation to the middleware transport infrastructure. Service implementation class is derived from the (service) skeleton.
  - Beside proxies/skeletons, there might exist a so-called "Runtime" (singleton) class to provide some essentials to manage proxies and skeletons. But this is communication management software implementation specific and therefore not specified in this document, but may be specified in a future version.
- It supports callback mechanisms on data reception.
  - 数据到达时候自动调用回调
- The API has zero-copy capabilities including the possibility for memory management in the middleware.
  - 零拷贝机制
- It is aligned with the AUTOSAR service model (services, instances, events, methods, ...) to allow the generation of proxies and skeletons out of this model.
- Full discovery and service instance selection support on API level.
- Client/Server Communication uses concepts introduced by C++11 language, e.g. std::future, std::promise, to fully support method calls between different contexts.
- Abstract from SOME/IP specific behavior, but support SOME/IP service mechanisms, as methods, events and fields.
- Support/implement the standard end-to-end protection protocols, as specified in
- **Support of Service contract versioning.**
- Support Event and Polling style usage of the API equally to enable classic RT style paradigms.
- Fully exploit C++11/14 features in API design to provide usability and comfort for the application developer

### 7.1.3-通信范式

SOA解耦范式

### 7.1.4-Service contract versioning

service定义是变化的<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671174104756-21f8b0f4-5021-4547-8640-51520bdbbe60.png#averageHue=%23d9d3a5&clientId=u0682565d-45bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=304&id=uf06ade4d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=304&originWidth=888&originalType=binary&ratio=1&rotation=0&showTitle=false&size=180959&status=done&style=none&taskId=u12e9d546-737f-476e-b0eb-9d7bcaecf1d&title=&width=888)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671174132970-8701ec3d-b44e-4d4f-a8d2-60024aa195a2.png#averageHue=%23e5d3b2&clientId=u0682565d-45bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=649&id=u46249e21&margin=%5Bobject%20Object%5D&name=image.png&originHeight=649&originWidth=920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=481062&status=done&style=none&taskId=u3a2b7300-e73a-4f5a-afda-69fc11daa76&title=&width=920)<br />AP支持service contract versioning。service contract versioning分为design phase和deployment phase，意味着service在设计阶段有一个版本号，在部署阶段也有一个版本号。

- 主版本号意味着不后向兼容，子版本号意味着后向兼容

## 7.2 E2E for Events

略

## 7.3-E2E for Methods

略

## 7.4-E2E for Fields

略

## 7.5-RawDataStreaming

略

## 7.6-CommunicationGroup

CG是为了Adaptive State Management applications而设计的概念。可以视作多个Service的组合，来按照指定方式管理信息传递。<br />一个CG可以有一个Server和多个Clients。Server用来发送广播和P2P报文到CG的Clients。Clients可以确认这些消息的送达。CG的Server可以确认多少个Clients连接到CG。App可以通过CommunicationGroupServer or CommunicationGroupClient来连接到CG的实例。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671175335209-12cfacee-3e6c-4296-a388-d8dd9da34268.png#averageHue=%2392b49c&clientId=u0682565d-45bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=541&id=u653fe255&margin=%5Bobject%20Object%5D&name=image.png&originHeight=541&originWidth=457&originalType=binary&ratio=1&rotation=0&showTitle=false&size=58802&status=done&style=none&taskId=u40c28a53-ebda-44f0-adce-babbc01cbf7&title=&width=457)<br />相当于一个多路复用器，在Clients和Servers之间做一个路由，从单级C/S到两级C/S架构

- 为了解决大量Client共用Server带来的通信开销问题。
- 知道并管理接入的Client

### 7.6.1-接口

The Communication Group uses two Service interfaces, one for a Communication Group Server and one for Communication Group clients.<br />客户端通过自身的Skeleton通过CG的Group Clients端接入<br />服务端通过自身的Proxy通过CG的Group Server端接入

#### 7.6.1.1-CG Server（Skeleton）

##### [SWS_CM_99000]fDRAFTg CommunicationGroupServer Service dA Communication Group shall provide a CommunicationGroupServer Service to be used by the Server of a Communication Group

##### [SWS_CM_99001]fDRAFTg Broadcast method of CommunicationGroupServer Service dThe CommunicationGroupServer Service shall provide the method broadcast to broadcast messages to the clients of the Communication Group. This method shall take as input parameter the message to be broadcasted. In case the boardcast method fails the method return shall provide an error code as specified in [SWS_CM_99024]

CGServer广播报文到CGClient

##### [SWS_CM_99002]fDRAFTg Peer To Peer Message method of CommunicationGroupServer Service dThe CommunicationGroupServer Service shall provide a method message to send a message to a dedicated client of the Communication Group. This method shall take as input parameters the message to be sent, and the clientID of the client which shall be addressed. In case the message method fails the method return shall provide an error code as specified in [SWS_CM_99024]

CGServer单播到CGClient

##### [SWS_CM_99014]fDRAFTg Message Response event of CommunicationGroupServer Service dThe CommunicationGroupServer Service shall provide an event response that contains the respond of a dedicated client to a broadcast or a peer to peer message of the Communication Group. The event shall provide the response message and the clientID of this response

CGServer支持CGClient通过event给到response到CGServer

##### [SWS_CM_99015]fDRAFTg List Clients method of CommunicationGroupServer Service The CommunicationGroupServer Service shall provide a method listClients to report about the connected clients of the Communication Group. This method shall have no input parameters and shall return the list of clients. In case the listClients method fails the method return shall provide an error code as specified in [SWS_CM_99024]

CGServer能够列出连接到的CGClients

#### 7.6.1.2-CGClient(Proxy)

##### [SWS_CM_99007]fDRAFTg CommunicationGroupClient Service dThe clients of a Communication Group shall provide a CommunicationGroupClient Service to be used by a Communication Group

##### [SWS_CM_99008]fDRAFTg Message method of CommunicationGroupClient Service dThe CommunicationGroupClient Service shall provide a method message for the client to receive a message from the Communication Group. This method shall take as input parameter the message. In case the message method fails the method return shall provide an error code as specified in [SWS_CM_99024]

CG能够给CGClient发信息

##### [SWS_CM_99009]fDRAFTg Message Response event of CommunicationGroupClient Service dThe CommunicationGroupClient Service shall provide an event response for the client to send a response message to the Communication Group. The event shall provide the response message

CGClient能够发response信息到CG

### 7.6.2-行为

##### [SWS_CM_99010]fDRAFTg Broadcast task dA Broadcast task shall be triggered by the broadcast method of the CommunicationGroupServer Service. The CommunicationGroup shall forward this broadcast message to all connected clients by calling the message method of the CommunicationGroupClient Service from each connected client.c

CGServer将数据广播到所有CGClient

##### [SWS_CM_99011]fDRAFTg Peer To Peer message task dA Peer to Peer message task shall be triggered by the message method (which includes the client address) of the CommunicationGroupServer Service. The CommunicationGroup shall forward this message to the addressed client by calling the message method of the CommunicationGroupClient Service of this client

CGServer将数据单播到某个CGClient

##### [SWS_CM_99012]fDRAFTg Message Response task dThe Message Response task shall be triggered by the message response event of the CommunicationGroupClient Service from a client . The CommunicationGroup shall forward this response message with the client source address to the message response event of the CommunicationGroupServer Service

CGClient收到消息后回复Ack

##### [SWS_CM_99013]fDRAFTg List Clients task dThe List Clients task shall be triggered by the list clients method of the CommunicationGroupServer Service. The CommunicationGroup shall provide the list of all connected client addresses with the return of the list clients method of the CommunicationGroupServer Service

CGServer知晓有哪些CGClient

### 7.6.3-Connection

CGServer和CGClient通过AP的Service方式联系

### 7.6.4-限制条件

- There is only one Server for an instance of a Communication Group at a given time.
  - 每个CG只有一个CGServer
- A Client provides the CommunicationGroupClient Service to only one instance of a Communication Group at a given time.
  - 一个Client单个时刻只能连接一个CGClient

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671177826582-bc2f7725-6192-42d2-b468-5428442233b3.png#averageHue=%23faede8&clientId=u0682565d-45bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=481&id=ucd62aec9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=481&originWidth=773&originalType=binary&ratio=1&rotation=0&showTitle=false&size=112187&status=done&style=none&taskId=u0e072b2d-9d12-4ed8-a8ae-2a824fb7028&title=&width=773)

### 7.6.5-CGModel

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671178145802-fd280773-63e7-4092-ad8e-0773a3de3596.png#averageHue=%23b69466&clientId=u0682565d-45bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=582&id=ue0fb2d90&margin=%5Bobject%20Object%5D&name=image.png&originHeight=582&originWidth=914&originalType=binary&ratio=1&rotation=0&showTitle=false&size=92610&status=done&style=none&taskId=uf884ef8c-8f65-47a7-ac34-0f0f6c6fda7&title=&width=914)

### 7.6.6-CGCreation

通过一个CG template来创建的<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671178329989-73e870d7-e69e-48a7-9eae-2f82274baad9.png#averageHue=%23fbf9f8&clientId=u0682565d-45bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=630&id=u9d81b875&margin=%5Bobject%20Object%5D&name=image.png&originHeight=630&originWidth=927&originalType=binary&ratio=1&rotation=0&showTitle=false&size=177493&status=done&style=none&taskId=u641b5d6c-2278-4e3b-aa9d-ac9f119b121&title=&width=927)<br />通过这个模板自动生成CGServer和CGClient的ARXML文件<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671178376921-44576917-df68-4eef-a340-43e993fac799.png#averageHue=%23fde3e3&clientId=u0682565d-45bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=434&id=u0c8671a1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=434&originWidth=935&originalType=binary&ratio=1&rotation=0&showTitle=false&size=66062&status=done&style=none&taskId=ub41e19fd-8edf-4af6-a450-61d50da1407&title=&width=935)

### CG总结

- 应用场景主要是StateManagement，状态信息的广播，比如上面的例子传递PowerMode
- 之所以用一个路由模块来专门做消息广播，可能是因为SOME/IP来做开销较大
  - SOME/IP有局限性，也印证了行业内关于海量数据DDS比SOME/IP更具优势的评论

## 7.7-OptionalExecutionContext

略

## 7.8-NetworkBinding

选择具体的network protocol binding是集成部署过程中的决策，AP允许在部署过程中修改network binding，不需要重新编译App代码，而是指需要re-linking重链接就OK。

### 7.8.1-SOME/IP binding

SOME/IP Events:

- uni-cast is one-to-one communication 一对一通信采用单播
- multi-cast is one-to-many communication 一对多通信采用多播
- many-to-one communication using multiple uni-cast communications   多对一通信采用多个单播

#### 7.8.1.1-ServiceDiscovery

服务端注册服务，会触发SOME/IP服务注册协议的第一个阶段

- [TPS_MANI_03012] (Initial Wait Phase),
- [TPS_MANI_03013] (Repetition Wait Phase),
- [TPS_MANI_03014] (Main Phase).

客户端服务发现，会触发SOME/IP服务注册协议的第一个阶段

- [TPS_MANI_03026] (Initial Wait Phase),
- [TPS_MANI_03027] (Repetition Wait Phase).

##### 服务发现报文定义

[SWS_CM_00202] SOME/IP FindService message dThe entries in the SOME/IP FindService message shall be as follows:<br />SOME/IP FindService报文的格式<br />[SWS_CM_10202] Version blacklist<br />服务发现协议，支持版本黑名单，对于版本黑名单的版本，不会进行服务协商<br />[SWS_CM_00203] SOME/IP OfferService message 报文定义<br />[SWS_CM_00204] SOME/IP StopOffer message 报文定义

##### SubscribeEventgroup报文定义

[SWS_CM_10377] Proxy端调用Subscribe订阅至少一个Event，会发送一个SOME/IP的SubscribeEventGroup报文<br />如果之前已经订阅过，或者收到了Ack信息，就不会新发送了<br />[SWS_CM_10381]如果过期了，要重新发送报文来更新订阅<br />[SWS_CM_00205] Content of SOME/IP SubscribeEventgroup message<br />[SWS_CM_00206] SOME/IP SubscribeEventgroupAck message

#### 7.8.1.2-Accumulation of SOME/IP messages

#### 7.8.1.3-Execution context of message reception actions

upon reception的含义：收到消息后到API（比如GetNewSamples）执行之间执行的动作。

#### 7.8.1.4-Handling Events

##### [SWS_CM_10287] Conditions for sending of a SOME/IP event message

发送报文的条件

- OfferService提供服务
- 有订阅者

##### [SWS_CM_10288] Transport protocol for sending of a SOME/IP event message

根据配置文件，选择UDP

##### [SWS_CM_10289] Source of a SOME/IP event message

报文的地址信息应该采用IPv4/v6的单播地址和端口号

##### [SWS_CM_10290] Destination of a SOME/IP event message

##### [SWS_CM_10291] Content of the SOME/IP event message

event报文的内容定义

##### [SWS_CM_10292] Checks for a received SOME/IP event message

##### [SWS_CM_10293] Identifying the right event

##### [SWS_CM_10379] Silently discarding SOME/IP event messages for unsubscribed events

##### [SWS_CM_10296] Invoke receive handler

##### [SWS_CM_10294] Deserializing the payload

##### [SWS_CM_10295] Providing the received event data

##### [SWS_CM_10360]{DRAFT} Failures in sending a SOME/IP event message

#### 7.8.1.5-HandlingTriggers

与Event一致，此时传递的数据是触发信息，Trigger，所以数据类型会简单很多

#### 7.8.1.6-Handling Method Calls

大部分与Event一致，

- 包括请求request信息和回应response信息
- 回应信息中包括错误信息
- Future数据的处理

##### [SWS_CM_10306] Invoke the method - event driven

收到消息后再触发Method

##### [SWS_CM_10307] Invoke the method - polling

意思是，拿到数据后就马上执行method<br />ProcessNextMethodCall

##### [SWS_CM_10428] payload representing application error

##### [SWS_CM_10317] Making the Future ready

使用set_value operation<br />这个动作会解除阻塞，get, wait, wait_for, and wait_until calls

##### [SWS_CM_10318] Invoke the notification function

dIf a notification function has been registered with the Future’s then method

#### 7.8.1.7-Handling Fields

是Event和Method的混合

##### [SWS_CM_10338] Invoke the registered set/get handlers - event driven

##### [SWS_CM_10339] Invoke the registered set/get handlers - polling

#### 7.8.1.8-Serialization of Payload

类型映射<br />结构体映射<br />字符串映射<br />数据分片

### 7.8.2-Signal-Based binding

略

### 7.8.3-DDS binding

##### [SWS_CM_11000] DDS Compliance

ARA::COM API上层采用SOME/IP语义，下层采用DDS语义，DDS binding的原则是遵循SOME/IP规范的同时遵循DDS的规范。

##### [SWS_CM_90500]{DRAFT} Choice of Service Instance discovery protocol

SOME/IP的发现协议映射到DDS的发现协议<br />Allows the application to attach additional information to the Entity object so that when the entity is discovered the remote application can access the data and use it. For example, it can be used to attach the security credentials to authenticate the source from the remote application. See UserDataQosPolicy.

##### [SWS_CM_90501]{DRAFT} Topic naming for Domain Participant USER_DATA QoS - based Service Instances

the de-facto Topic naming scheme for events, methods and fields is the one described for SERVICE_INSTANCE_RESOURCE_PARTITION<br />events, methods, fields的名字来自于SERVICE_INSTANCE_RESOURCE_PARTITION

#### 7.8.3.1-Service Discovery via Domain Participant USER_DATA QoS policy

通过participant来进行服务发现

##### [SWS_CM_11001] Mapping of OfferService method

##### [SWS_CM_11002] Assigning a DDS DomainParticipant to a Service Instance

The DDS Binding shall assign a DDS DomainParticipant to every Service Instance<br />这是最自然的想法，在ROS2中一开始也是这样，一个Node对应一个DomainParticipant，但是后来的实践证明，由于DDS太重，Participant太多会导致性能问题，后来ROS2改成了一个进程一个Participant，所以恐怕不能一个SI对应一个Participant.<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671516235887-afb67e35-0f5b-478f-8b9a-6f323593fbc4.png#averageHue=%23f0eae4&clientId=ufbc602d6-318b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=133&id=uc5fbbe9d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=133&originWidth=790&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56532&status=done&style=none&taskId=u0096acc8-cf5f-4734-9324-7c8bec4c61e&title=&width=790)<br />事实上，AP标准就是一个进程对应一个Participant<br />而AP的标准，一个进程可能对应多个SI，所以存在多个SI，一个Participant的情况。<br />创建Participant之前要先搜索，如果已经创建就不重复创建。

##### [SWS_CM_11003] Assigning a DDS Topic and a DDS DataWriter to every Event in the ServiceInterface

对于Event，创建对应的DataWriter<br />先搜索已有的Topics，如果没有，则针对当前的Event创建新的DDS Topic，DDS Publisher，DataWriter

##### [SWS_CM_10550]{DRAFT} Assigning a DDS Topic and a DDS DataWriter to every Trigger in the ServiceInterface

对Triggers也是和Topic对应

##### [SWS_CM_11029] Assigning a DDS Request and Reply Topic, and DataWriters and DataReaders, to the Methods in the ServiceInterface

Methods对应一个Request 和一个Reply Topic

##### [SWS_CM_11030] Assigning a DDS Topic and a DDS DataWriter to every Field in the ServiceInterface with its hasNotifier attribute equal to true

对于Field也是Topic，但是有一个Notifer Attribute

##### [SWS_CM_11031] Assigning a DDS Request and Reply Topic, and DataWriters and DataReaders, to the Field Getters/Setters in the ServiceInterface

对于Field的Getters/Setters，也是按照Method的设计方式来，所以是Request/Reply Topic

##### [SWS_CM_09004] Adding Service IDs, Service Instance IDs, and ServiceInterface Contract Versions to the DDS DomainParticipant’s USER_DATA QoS Policy

对于基于participant USER_DATA_QOS 的服务发现来讲，需要将这些信息添加到服务发现信息中

##### [SWS_CM_11005] Mapping of StopOfferService method

不删除Participant，删除SI对应的Events，Triggers, Methods, Fields

##### [SWS_CM_11006] Mapping of FindService method

##### [SWS_CM_11007] Finding a DDS DomainParticipant suitable for performing client-side operations

在Client端，也要创建一个DomainParticipant

##### [SWS_CM_11008] Creating a DDS DomainParticipant suitable for performing client-side operations

##### [SWS_CM_11009] Discovering remote Service Instances through DDS DomainParticipants

##### [SWS_CM_11010] Mapping of StartFindService method

##### [SWS_CM_11011] Defining a DDS BuiltinParticipantListener

The DDS Binding implementation shall define a BuiltinParticipantListener class to handle notifications whenever a remote DomainParticipant is discovered<br />这个listener是用来监测Participant状态变化的 domainparticipantlistener<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671519491827-d72438ca-5d7e-4d58-9c49-83e748cdcc71.png#averageHue=%23faf8f7&clientId=ue8a9adb6-8116-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=151&id=u91826046&margin=%5Bobject%20Object%5D&name=image.png&originHeight=151&originWidth=960&originalType=binary&ratio=1&rotation=0&showTitle=false&size=22680&status=done&style=none&taskId=u419fd787-a180-437c-bc66-41b21e341a0&title=&width=960)

##### [SWS_CM_11012] Binding a BuiltinParticipantListener to a DDS DomainParticipant

##### [SWS_CM_11014] Unbinding a BuiltinParticipantListener from a DDS DomainParticipant

#### 7.8.3.2-Service Discovery via Topic

SI的Event, Method和Fields与DDS实体的对应是一样的，不同的是服务发现的方式

##### [SWS_CM_90508]{DRAFT} Advertising Service IDs, Service Instance IDs, and ServiceInterface Contract Versions over the ara.com://services/discovery topic

这样，服务发现可以避免使用DDS规范使用的SPDP，简化服务的发现

##### [SWS_CM_90513]{DRAFT} Discovering remote Service Instances through the ara.com://services/discovery topic

每一个SI将自己的SI发现信息发送到指定的Topic，这样服务发现变成了搜索

#### 7.8.3.3-Handling Events

##### [SWS_CM_11015] Mapping Events to DDS Topics

ara.com://services/<InterfaceID>/<Major>.<Minor>/<TopicName>

##### [SWS_CM_11016] DDS Topic data type definition

```cpp
struct <eventTypeName>EventType {
    @key uint16 instance_id;
    <eventTypeName> data;
};
```

##### [SWS_CM_11017] Mapping of Send method dWhen instructed to send an event message, the DDS Binding shall construct a new sample of the equivalent DDS Topic data type

通过publisher往Topic发送数据

##### [SWS_CM_11018] Mapping of Subscribe method dWhen instructed to subscribe to an event, the DDS binding shall create a DDS DataReader using the DDS Subscriber created for the proxy in

##### [SWS_CM_11019] Creating a DDS DataReader for event subscription

##### [SWS_CM_11020] Defining a DDS DataReaderListener

Listener是用来触发回调的

##### [SWS_CM_11021] Mapping of Unsubscribe method dWhen instructed to unsubscribe from a service event, the DDS binding shall delete the DataReader associated with the event.c

SOME/IP的unsubscriber语义在DDS中直接delete析构对象

##### [SWS_CM_11022] Mapping of GetSubscriptionState method dWhen instructed to provide the subscription state, the DDS binding shall check if the DataReader associated with the subscription exists

- If it does exist, the binding shall call the DataReader’s get_subscription_matched_status() method next.
- If the total_count attribute of the resulting SubscriptionMatchedStatus is greater than zero, GetSubscriptionState() shall return SubscriptionState = kSubscribed.
- Otherwise, it shall return SubscriptionState = kSubscriptionPending
- Else, if it does not exist—which indicates that either Subscribe() has never invoked or Unsubscribe() has been called before—GetSubscriptionState () shall return SubscriptionState = kNotSubscribed.

##### [SWS_CM_11023] Mapping of GetNewSamples method  dWhen instructed to get new samples, the DDS binding shall perform a take() on the DataReader as follows

- If a maxNumberOfSamples is specified, the binding implementation shall invoke take() with max_samples = maxNumberOfSamples.
- Else, if no maxNumberOfSamples is specified (i.e., if maxNumberOfSamples is equal to the default value std::numeric_limits<std::size_t> ::max()), the binding implementation shall invoke take() without specifying a max_samples limit.

##### [SWS_CM_11024] Mapping of GetFreeSampleCount method dWhen instructed to provide the number of free sample slots, the binding implementation shall return the number free sample slots in the DDS DataReader’s cache.c(

##### [SWS_CM_11025] Mapping of SetReceiveHandler method dWhen instructed to register an EventReceiveHandler, the binding implementation shall perform the following operations

使用data-reader-listener的回调

##### [SWS_CM_11026] Mapping of UnsetReceiveHandler method dWhen instructed to unregister an EventReceiveHandler, the binding implementation shall perform the following operations

使用data-reader-listener的回调

##### [SWS_CM_11027] Mapping of SetSubscriptionStateHandler method dWhen instructed to register a SubscriptionStateChangeHandler, the binding implementation shall perform the following operation

使用data-reader-listener的回调

##### [SWS_CM_11028] Mapping of UnsetSubscriptionStateHandler method dWhen instructed to unregister a SubscriptionStateChangeHandler, the binding implementation shall perform the following operations

#### 7.8.3.4-Handling Triggers

与Event一致，此时传递的数据是触发信息，Trigger，所以数据类型会简单很多

#### 7.8.3.5-Handling Method Calls

注意：FastDDS和CycloneDDS都不支持DDS-RPC，只提供基于Topic的数据发布订阅，而ROS的Service是在RCL层基于DDS Topic来实现的<br />在AP中，SOME/IP的Method与DDS-RPC实体做对应，DDS-RPC的规范与RCL的实现一致，使用两个Topic来模拟一个Service<br />但是Topic数据类型使用了DDS-RPC规定的数据类型如RequestHeader，所以目前应该是不支持的

##### [SWS_CM_11101] DDS Service Request Topic data type definition

```cpp
struct <svcId>Method_Request {
 dds::rpc::RequestHeader header;
 <svcId>Method_Call data;
 };
```

##### [SWS_CM_11102] DDS Service Reply Topic data type definition

##### [SWS_CM_10431] Mapping of ara::core::ErrorCode

##### [SWS_CM_11109] Processing a method call on the server side (event driven)

##### [SWS_CM_11110] Creating a DataReaderListener to process asynchronous requests on the server side d

通过DataReaderListener来做异步

##### [SWS_CM_11111] Processing a method call on the server side (polling)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671523868063-10c6e438-2e0a-4a0d-832d-af2371e704d3.png#averageHue=%23f2eeea&clientId=u7ca6137b-3ecf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=281&id=udfd3a039&margin=%5Bobject%20Object%5D&name=image.png&originHeight=281&originWidth=803&originalType=binary&ratio=1&rotation=0&showTitle=false&size=136761&status=done&style=none&taskId=u27a0ede2-6a5a-46f5-8d10-f30bc1bcb5e&title=&width=803)

#### 7.8.3.6-Handling Fields

##### [SWS_CM_11130] Mapping Fields with hasNotifier attribute to DDS Topics

#### 7.8.3.7-Serialization of Payloads

遵循DDS的规范，DDS-RTPS文档<br />类型映射<br />结构体映射<br />字符串映射<br />数据分片

## 7.9-Security

略

## 7.10-ComAPI

对于Chapter8的一个概述

# 8-ComAPI-Spec

全部都是代码细节，略过<br />根据需求查询即可

# 9-ServiceInterface

略

# 参考

[https://github.com/COVESA](https://github.com/COVESA)<br />[https://some-ip.com/](https://some-ip.com/)<br />[https://github.com/COVESA/capicxx-someip-tools/wiki/CommonAPI-C---SomeIP-in-10-minutes](https://github.com/COVESA/capicxx-someip-tools/wiki/CommonAPI-C---SomeIP-in-10-minutes)<br />[https://www.cnblogs.com/polelink/p/16043599.html](https://www.cnblogs.com/polelink/p/16043599.html)<br />[https://stackoverflow.com/questions/51182471/whats-the-difference-between-dds-and-some-ip](https://stackoverflow.com/questions/51182471/whats-the-difference-between-dds-and-some-ip)<br />[https://cdn.vector.com/cms/content/know-how/_technical-articles/PREEvision/PREEvision_MiddlewareProtocols_ElektronikAutomotive_202003_PressArticle_EN.pdf](https://cdn.vector.com/cms/content/know-how/_technical-articles/PREEvision/PREEvision_MiddlewareProtocols_ElektronikAutomotive_202003_PressArticle_EN.pdf)<br />[https://blog.csdn.net/xiaoxiaozengz/article/details/115531135](https://blog.csdn.net/xiaoxiaozengz/article/details/115531135)
