# 概念

## 参考

[https://github.com/eclipse-iceoryx/iceoryx/blob/master/doc/website/getting-started/overview.md](https://github.com/eclipse-iceoryx/iceoryx/blob/master/doc/website/getting-started/overview.md)

## 概述

![](https://tcs.teambition.net/storage/312651c434c461ff03eed4cd15591a2902ab?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODAxMCwiaWF0IjoxNjcxMTkzMjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY1MWM0MzRjNDYxZmYwM2VlZDRjZDE1NTkxYTI5MDJhYiJ9.iRTfksbKZTx4vpwhwtMcr22YWXWLJ-coGXuUGp9e6cY&download=image.png "")

一个iceoryx系统包括

- RouDi 后台

- 运行在Post Runtime基础上的N个进程

### RouDi

RouDi (''Rou''ting and ''Di''scovery) is the core of the system and is responsible for:

- 服务发现

publisher和subscriber的业务处理中心

- 共享内存管理

初始化共享内存段

- 系统检查

对于现有系统的端口，Port，提供一系列接口来检查自身信息

是iceoryx系统的管理服务器，RouDi拥有一系列的软件组件来扩充其功能

### PoshRuntime

A Posh runtime is a running entity with its own isolated memory space that participates in the iceoryx system.

In a POSIX system, a Posh runtime and a POSIX process have a one-to-one mapping.

进程，作为Ice系统的一个参与者，是一个运行实体

A Posh runtime may offer services to the iceoryx system or discover services offered by other runtimes to interface with.

The services offered by Posh runtimes communicate via events and the event flow is reasoned about using publish-subscribe semantics.

A service must be explicitly registered with RouDi to participate in communication.

### 共享内存机制

An iceoryx system utilizes one "management" segment for administration purposes and any number of "user" segments for event communication between services.

These segments are logically partitioned into "mempools". Mempools contain a number of equally sized "memory chunks".

共享内存管理段分为一系列的内存池，内存池进一步分为等尺寸的内存束

The number of segments used by an iceoryx system, along with the configuration of the mempools they contain, are provided to the system via configuration.

可以在编译时配置，通过头文件，也可以在运行时配置，通过toml配置文件

## 通信机制

### Ports

port是通信数据流的一个实体，有不同的实现

- PublisherPort

- 输出数据

- SubscriberPort

- 接收数据

- InterfacePort

- 被gateways使用，用于远程连接，连接另一个ice系统

A Publisher in an iceoryx system publishes data via a PublisherPort, and likewise, a Subscriber receives data via a SubscriberPort.

### PortWiring(Port现在叫做Topic)

Matching Publishers with Subscribers in iceoryx is achieved by connecting their underlying PublisherPorts and SubscriberPorts.

连接发布者和订阅者的信息是服务Service

- A service id - identifies the type of service-新的叫做Group Name

- A service instance id - identifies an instance of a service，新的叫做Instance Name

- An event id - identifies an output from a service，新的叫做Topic Name

服务描述相关代码在`capro`中

![](https://tcs.teambition.net/storage/3126760ed4a11928742f3165701ae917047b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODAxMCwiaWF0IjoxNjcxMTkzMjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY3NjBlZDRhMTE5Mjg3NDJmMzE2NTcwMWFlOTE3MDQ3YiJ9.5NTSmVpRsjYn1n5OHiMa3ei3QbZ6BnYKb1Qul_kWafU&download=image.png "")

All PublisherPorts and SubscriberPorts are created with a service description. The system will automatically connect ports with matching service descriptions.

The order that ports appear in is not a factor. Existing SubscriberPorts will automatically connect to PublisherPorts that appear at a later time if their service descriptions match (and vice versa).

### 零拷贝通信过程

To output data, a PublisherPort reserves a memory chunk in its assigned memory segment. The iceoryx system will intelligently choose the smallest chunk size that can fit the output data structure. Note that an entire chunk is reserved even if the data type it contains is smaller than its size.

A PublisherPort chooses explicitly when to deliver data written in a memory chunk to all of its attached SubscriberPorts (established via discovery). When this occurs, a pointer to the memory chunk is placed on a receive queue at the SubscriberPort. The SubscriberPort can then access the data at its own convenience by following the pointer.

传递的是指针

### Waitset等待集

The easiest way to receive data is to periodically poll whether data is available. This is sufficient for simple use cases but inefficient in general, as it often leads to unnecessary latency and wake-ups without data.

定期轮询消息是效率低下的

The Waitset can be used to relinquish control (putting the thread to sleep) and wait for user defined events to occur. Here an event is associated with a condition and occurs when this condition becomes true. Usually these events correspond to the availability of data at specific subscribers. This way we can immediately wake up when data is available and will avoid unnecessary wake-ups if no data is available.

线程进入waitset，等待events唤醒

## API

有c和cpp两种API

cpp API遵循函数式编程

typed API and the untyped API，有严格类型限制的接口尽量使用，比较安全

如果是Untyped，则指针类型为void，需要用户来保证数据传输安全，这个在与其他底层API如ROS协作时会用到

Option接口，``

Expected接口，

## 使用API

This involves setting up the runtime in each application, creating publishers in applications that need to send data and subscribers in applications that want to receive said data.

### runtime

```

iox::runtime::PoshRuntime::initRuntime("some_unique_name");

```

Now this application is ready to communicate with the middleware daemon RouDi.

### typed api

```

iox::popo::Publisher<CounterTopic> publisher({"Group", "Instance", "CounterTopic"});



publisher.offer();



## 与ROS2对应概念

![](https://tcs.teambition.net/storage/31278c6239b8e1e3de35e745ec49970a5739?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODAxMCwiaWF0IjoxNjcxMTkzMjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc4YzYyMzliOGUxZTNkZTM1ZTc0NWVjNDk5NzBhNTczOSJ9.fq23V2I8CHZbLxt8Gwoe21we8WTblynzxZ5vftz_rCk&download=image.png "")

service: 消息类型 topic type

instance: 消息名字 topic name

event: 事件类型，ros2的外部全部是data

process: 进程

Node: 节点，ros2中的节点名字，如果是python脚本，则与进程名字一致 node_name
