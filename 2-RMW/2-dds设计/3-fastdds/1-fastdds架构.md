# 层次化模型

- **Two API Layers.**_eProsima Fast DDS_ comprises a high-level DDS compliant layer focused on usability and a lower-level RTPS compliant layer that provides finer access to the RTPS protocol.
- **Easy to use.** The project comes with an out-of-the-box example, the _DDSHelloWorld_ (see Getting Started) that puts into communication a publisher and a subscriber, showcasing how _eProsima Fast DDS_ is deployed. Additionally, the interactive demo _ShapesDemo_ is available for the user to dive into the DDS world. The DDS and the RTPS layers are thoroughly explained in the DDS Layer and RTPS Layer sections.

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565320963-ad8cc1de-879d-4560-883f-6284b547dfb0.png#clientId=uac293f21-a464-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ubeb2c04d&margin=%5Bobject%20Object%5D&originHeight=942&originWidth=1112&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u382d8f4e-c759-4fbe-b603-a305de2ea23&title=)

- **Application layer**. The user application that makes use of the _Fast DDS_ API for the implementation of communications in distributed systems.
- **Fast DDS layer**. Robust implementation of the DDS communications middleware. It allows the deployment of one or more DDS domains in which DomainParticipants within the same domain exchange messages by publishing/subscribing under a domain topic.
- **RTPS layer**. Implementation of the Real-Time Publish-Subscribe (RTPS) protocol for interoperability with DDS applications. This layer acts an abstraction layer of the transport layer.
- **Transport Layer**. _Fast DDS_ can be used over various transport protocols such as unreliable transport protocols (UDP), reliable transport protocols (TCP), or shared memory transport protocols (SHM).

# 线程和执行模型

FastDDS是并行的，基于事件的。

## 线程模型

DomainParticipant启动了一系列线程来处理一系列的后台任务，包括logging，message reception, asynchronous communication。这些后台线程不应该应用用户对库的使用，例如FastDDS API是线程安全的，所以你可以在不同线程下访问DomainParticipant，但是，当外部函数和库同时访问某资源的时候，这种多线程的实现方式必须考虑了，一个例子是在entity listener callbacks。<br />The following is a brief overview of how _Fast DDS_ multithreading schedule work:

- Main thread: Managed by the application.
- 主线程
- Event thread: Each DomainParticipant owns one of these. It processes periodic and triggered time events.
- 事件处理线程
- Asynchronous writer thread: This thread manages asynchronous writes for all DomainParticipants. Even for synchronous writers, some forms of communication must be initiated in the background.
- 异步写线程
- Reception threads: DomainParticipants spawn a thread for each reception channel, where the concept of a channel depends on the transport layer (e.g. a UDP port).
- 读线程

## 事件驱动架构

There is a time-event system that enables _Fast DDS_ to respond to certain conditions, as well as schedule periodic operations. Few of them are visible to the user since most are related to DDS and RTPS metadata. However, the user can define in their application periodic time-events by inheriting from the TimedEvent class.<br />有一个时间事件系统，FastDDS通过这个时间事件系统来做操作，大部分操作会处理DDS和RTPS的元数据，所以用户一般不用考虑这个，但是用户依然可以通过继承TimedEvent的方式来创建时间事件<br />（时间事件应该就是timer定时器事件吧？）

# 支持的传输方式

- **Transport layers.**_eProsima Fast DDS_ implements an architecture of pluggable transports. The current version implements five transports: UDPv4, UDPv6, TCPv4, TCPv6 and SHM (shared memory).

# 功能

## 发现协议

发现协议指的是，DataWriter和DataReader共享数据的机制。

- **Simple Discovery**. This is the default discovery mechanism, which is defined in the RTPS standard and provides compatibility with other DDS implementations Here the DomainParticipants are discovered individually at an early stage to subsequently match the DataWriter and DataReader they implement.
- 这是默认机制，RTPS规范定义的
- **Discovery Server**. This discovery mechanism uses a centralized discovery architecture, where servers act as a hubs for discovery meta traffic.
- 集中式的发现服务器
- **Static Discovery**. This implements the discovery of DomainParticipants to each other but it is possible to skip the discovery of the entities contained in each DomainParticipant (DataReader/DataWriter) if these entities are known in advance by the remote DomainParticipants.
- 避免广播发现，静态配置
- **Manual Discovery**. This mechanism is only compatible with the RTPS layer. It allows the user to manually match and unmatch RTPSParticipants, RTPSWriters, and RTPSReaders using whatever external meta-information channel of its choice.
- 只是兼容性功能，用于在RTPS层手动匹配读者和写者

## Security

_Fast DDS_ can be configured to provide secure communications by implementing pluggable security at three levels:

- Authentication of remote DomainParticipants. The **DDS:Auth:PKI-DH** plugin provides authentication using a trusted Certificate Authority (CA) and ECDSA Digital Signature Algorithms to perform the mutual authentication. It also establishes a shared secret using Elliptic Curve Diffie-Hellman (ECDH) Key Agreement protocol.
- Access control of entities. The **DDS:Access:Permissions** plugin provides access control to DomainParticipants at the DDS Domain and Topic level.
- Encryption of data. The **DDS:Crypto:AES-GCM-GMAC** plugin provides authenticated encryption using Advanced Encryption Standard (AES) in Galois Counter Mode (AES-GCM).

## Logging

_Fast DDS_ provides an extensible Logging system. Log class is the entry point of the Logging system. It exposes three macro definitions to ease its usage: logInfo, logWarning and logError. Moreover, it allows the definition of new categories, in addition to those already available (INFO_MSG, WARN_MSG and ERROR_MSG). It provides filtering by category using regular expressions, as well as control of the verbosity of the Logging system. Details of the possible Logging system configurations can be found in the Logging section.
