# FastDDS架构概述

- **Two API Layers.**_eProsima Fast DDS_ comprises a high-level DDS compliant layer focused on usability and a lower-level RTPS compliant layer that provides finer access to the RTPS protocol.
- **Easy to use.** The project comes with an out-of-the-box example, the _DDSHelloWorld_ (see Getting Started) that puts into communication a publisher and a subscriber, showcasing how _eProsima Fast DDS_ is deployed. Additionally, the interactive demo _ShapesDemo_ is available for the user to dive into the DDS world. The DDS and the RTPS layers are thoroughly explained in the DDS Layer and RTPS Layer sections.

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565320963-ad8cc1de-879d-4560-883f-6284b547dfb0.png#clientId=uac293f21-a464-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ubeb2c04d&margin=%5Bobject%20Object%5D&originHeight=942&originWidth=1112&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u382d8f4e-c759-4fbe-b603-a305de2ea23&title=)

- **Application layer**. The user application that makes use of the _Fast DDS_ API for the implementation of communications in distributed systems.
- **Fast DDS layer**. Robust implementation of the DDS communications middleware. It allows the deployment of one or more DDS domains in which DomainParticipants within the same domain exchange messages by publishing/subscribing under a domain topic.
- **RTPS layer**. Implementation of the Real-Time Publish-Subscribe (RTPS) protocol for interoperability with DDS applications. This layer acts an abstraction layer of the transport layer.
- **Transport Layer**. _Fast DDS_ can be used over various transport protocols such as unreliable transport protocols (UDP), reliable transport protocols (TCP), or shared memory transport protocols (SHM).

# DDS层设计

主要API是DDSI-DCPS API，基于DDS1.4版本定义，

## **Entity**

- **Entity:** An _Entity_ is a DDS communication object that has a _Status_ and can be configured with _Policies_.
  - 实体就是DDS通信的对象，拥有一个状态对象，策略对象和Listener

## Policy

- **Policy:** Each of the configuration objects that govern the behavior of an _Entity_.
  - 管理实体对象行为的对象，叫做策略，在DDS中主要就是QoS Policy
- QoS Policy
- The behavior of each entity can be configured with a set of configuration policies. For each entity type, there is a corresponding Quality of Service (QoS) class that groups all the policies that affect said entity type. Users can create instances of these QoS classes, modify the contained policies to their needs, and use them to configure the entities, either during their creation or at a later time with the set_qos() function that every entity exposes
  - 对于每种类型的实体，都可以设置QoS 策略来改变实体的行为特性
  - 分为很多独立的类型QoS，这些类型可能是互相冲突的，实体在设置QoS的时候通过返回值判断是否发生配置冲突
  - 有些QoS是immutable不可变的，意味着其配置只能在实体创建和使能之前配置
  - QoS策略可以通过扩展机制扩展
  - Topic对象也有QoS策略！

## Listener

- Listener
- A listener is an object with functions that an entity will call in response to events. **Therefore, the listener acts as an asynchronous notification system that allows the entity to notify the application about the Status changes in the entity.**
  - 是一个异步通知机制，用于报告实体的状态变更，与listener绑定的回调在状态变更时会触发
- All entity types define an abstract listener interface, which contains the callback functions that the entity will trigger to communicate the Status changes to the application. Users can implement their own listeners inheriting from these interfaces and implementing the callbacks that are needed on their application. Then they can link these listeners to each entity, either during their creation or at a later time with the set_listener() function that every entity exposes
  - 每个实体都会有一个listener基类，用户可以继承这个基类，定义listener callback
- Only one thread is created to listen for every listener implemented, so it is encouraged to keep listener functions simple, leaving the process of such information to the proper class.

## Status

- **Status:** Each of the objects associated with an _Entity_, whose values represent the _communication status_ of that _Entity_.
  - 状态，用于表示对象在通信活动中的行为状态
  - 状态变更会触发实体的一些事件，比如when new data arrives, a new participant is discovered, or a remote endpoint is lost.
  - 状态会分类分解为多个状态对象，每个状态对象复杂关注通信的某个方面

Status与Listener的关系<br />Changes on a status object trigger the corresponding Listener callbacks that allow the Entity to inform the application about the event. For a given status object with name fooStatus, the entity listener interface defines a callback function on_foo() that will be called when the status changes. Beware that some statuses have data members that are reset every time the corresponding listener is called. The only exception to this rule is when the entity has no listener attached, so the callback cannot be called. See the documentation of each status for details.<br />每一次状态变更，都会调用其listener的回调函数<br />The entities expose functions to access the value of its statuses. For a given status with name fooStatus, the entity exposes a member function get_foo() to access the data in its fooStatus. The only exceptions are DataOnReaders and DataAvailable. These getter functions return **a read-only struct where all data members are public and accessible to the application**. Beware that some statuses have data members that are reset every time the getter function is called by the application. See the documentation of each status for details.<br />暴露一个接口访问其状态。<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565346156-27c94c83-9e17-4796-a793-64693433ee05.png#clientId=u80d56aec-d9dd-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ue52cd245&margin=%5Bobject%20Object%5D&originHeight=510&originWidth=941&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ua886704d-1a50-4f8b-89dc-e89217b328c&title=)

## Publisher-DataWriter-Topic

- **Publisher**: It acts as a factory that can create any number of DataWriters. See Publisher for further details.
- **Subscriber**: It acts as a factory that can create any number of DataReaders. See Subscriber for further details.
- **Topic**: This entity fits between the publication and subscription entities and acts as a channel. See Topic for further details.
- **DataWriter**: Is the object responsible for the data distribution. See DataWriter for further details.
- **DataReader**: Is the object used to access the received data. See DataReader for further details.

Publisher是DataWriter的Factory<br />Subscriber是DataReader的Factory<br />A DataWriter is attached to exactly one Publisher that acts as a factory for it. Additionally, each DataWriter is bound to a single Topic since its creation. This Topic must exist prior to the creation of the DataWriter, and must be bound to the data type that the DataWriter wants to publish.<br />The effect of creating a new DataWriter in a Publisher for a specific Topic is to initiate a new publication with the name and data type described by the Topic.<br />Once the DataWriter is created, the application can inform of changes in the data value using the write() member function on the DataWriter. These changes will be transmitted to all subscriptions matched with this publication.

# RTPS层设计

RTPS层设计实现是依据RTPS规范，该层相比DDS层而言，对通信协议有更多更密集的设计。所以高级用户对库的功能有着更精细的控制。<br />RTPS的实体与DDS层的实体有着一对一的关系

| DDS Layer | RTPS Layer |
| --- | --- |
| Domain | RTPSDomain |
| DomainParticipant | RTPSParticipant |
| DataWriter | RTPSWriter |
| DataReader | RTPSReader |

RTPS层与DDS层的区别在于，用各个属性Attributes代替了QoS Policy<br />RTPSParticipantAttributes<br />HistoryAttributes<br />WriterAttributes<br />ReaderAttributes

## 通过History发送和接受数据

In the RTPS Protocol, Readers and Writers save the data about a topic in their associated Histories. Each piece of data is represented by a Change, which _eprosima Fast DDS_ implements as CacheChange_t. Changes are always managed by the History.<br />RTPS规范中，读者和写者将数据保存在历史中，每一块数据都用Change来表示。Change是History来管理的。<br />发送数据<br />You can add a new CacheChange_t to the History of the Writer to send data. The procedure is as follows:

1. Request a CacheChange_t from the Writer with RTPSWriter::new_change(). In order to allocate enough memory, you need to provide a callback that returns the maximum number bytes in the payload.
2. Fill the CacheChange_t with the data.
3. Add it to the History with WriterHistory::add_change().

接收数据<br />You can receive data from within the ReaderListener::onNewCacheChangeAdded callback, as we did in the DDS Layer:

1. The callback receives a CacheChange_t parameter containing the received data.
2. Process the data within the received CacheChange_t.
3. Inform the Reader’s History that the change is not needed anymore.

## RTPSReaders-RTPSWriters

DataDurability kind<br />The Durability parameter defines the behavior of the Writer regarding samples already sent when a new Reader matches. _eProsima Fast DDS_ offers three Durability options:<br />定义当新的读者匹配上的时候，写者对已经发出数据的行为

- VOLATILE (default): Messages are discarded as they are sent. If a new Reader matches after message _n_, it will start received from message _n+1_.
  - 数据在发送之后就丢弃，如果新的Reader接入，则不会收到历史数据，而是收到最新的数据
- TRANSIENT_LOCAL: The Writer saves a record of the last _k_ messages it has sent. If a new reader matches after message _n_, it will start receiving from message _n-k_
  - Writer保留k个已经发送数据的记录，如果有新的Reader接入，则会接受到过去k个数据
- TRANSIENT: As TRANSIENT_LOCAL, but the record of messages will be saved to persistent storage, so it will be available if the writer is destroyed and recreated, or in case of an application crash.
  - 相比于Local，这个是全局的，会保留在持久存储上，而不是Writer自己的缓存中，这样是防止应用程序崩溃后导致的数据丢失

## History

History是payload的容器

- 可以改变payload的最大size
- 可以改变History容器的size

自定义Payload Pool<br />A _Payload_ is defined as the data the user wants to transmit between a Writer and a Reader. RTPS needs to add some metadata to this Payload in order to manage the communication between the endpoints. Therefore, this Payload is encapsulated inside the SerializedPayload_t field of the CacheChange_t, while the rest of the fields of the CacheChange_t provide the required metadata.<br />Payload 有效载荷，有效载荷在SerializedPayload_t中，除了有效载荷，还有协议头，共同组成CacheChange_t<br />WriterHistory and ReaderHistory provide an interface for the user to interact with these changes: Changes to be transmitted by the Writer are added to its WriterHistory, and changes already processed on the Reader can be removed from the ReaderHistory.<br />Writer先将数据放到History中，然后发送，发送完毕后从History中移除；<br />Reader先收数据，然后放到History中，处理完毕后在History中移除；<br />在处理过程中，遵循先进先出，老的数据逐渐被移除，为了处理这个载荷，使用PayloadPool

### PayloadPool

FastDDS提供了集中PayloadPool，可以配置<br />**PREALLOCATED_MEMORY_MODE**<br />All payloads will have a data buffer of fixed size, equal to the value of payloadMaxSize, regardless of the size requested to IPayloadPool::get_payload. Released Payloads can be reused for another CacheChange_t. This reduces memory allocation operations at the cost of higher memory usage.During the initialization of the History, initialReservedCaches Payloads are preallocated for the initially allocated CacheChange_t.<br />预分配内存模式，这种模式下，数据变化大，则可能出现payload 大小不够的情况<br />**PREALLOCATED_WITH_REALLOC_MEMORY_MODE**<br />Payloads are guaranteed to have a data buffer at least as large as the maximum between the requested size and payloadMaxSize. Released Payloads can be reused for another CacheChange_t. If there is at least one free Payload with a buffer size equal or larger to the requested one, no memory allocation is done.During the initialization of the History, initialReservedCaches Payloads are preallocated for the initially allocated CacheChange_t.<br />预分配加重分配模式，这种性能没那么好<br />**DYNAMIC_RESERVE_MEMORY_MODE**<br />Every time a Payload is requested, a new one is allocated in memory with the appropriate size. payloadMaxSize is ignored. The memory of released Payloads is always deallocated, so there are never free Payloads in the pool. This reduces memory usage at the cost of frequent memory allocations.<br />No preallocation of Payloads is done in the initialization of the History,<br />动态分配模式，没有预分配，这种性能更差一些。<br />**DYNAMIC_REUSABLE_MEMORY_MODE**<br />Payloads are guaranteed to have a data buffer at least as large as the requested size. payloadMaxSize is ignored.Released Payloads can be reused for another CacheChange_t. If there is at least one free Payload with a buffer size equal or larger to the requested one, no memory allocation is done.

# Transport层

DDS层与Transport层的关系：DDS层使用transport层来做数据传输和参与者发现，DDSlayer是传输独立的，定义了与Transport层交互的接口，各种传输方式以插件的方式实现这些API，所以DDS层并不局限于某个特定的传输方式。

- **UDPv4**: UDP Datagram communication over IPv4. This transport is created by default on a new DomainParticipant if no specific transport configuration is given (see UDP Transport).
- **UDPv6**: UDP Datagram communication over IPv6 (see UDP Transport).
- **TCPv4**: TCP communication over IPv4 (see TCP Transport).
- **TCPv6**: TCP communication over IPv6 (see TCP Transport).
- **SHM**: Shared memory communication among entities running on the same host. This transport is created by default on a new DomainParticipant if no specific transport configuration is given (see Shared Memory Transport).

默认创建的是UDP和SHM。当然Intra-process-data-delivery和data-sharing-delivery也是在实体之间传输数据的方式。<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565346153-5b979714-9b69-4c85-873e-d223c90a5cfd.png#clientId=u80d56aec-d9dd-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u5b390cf2&margin=%5Bobject%20Object%5D&originHeight=1555&originWidth=3617&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u3d3dbc2e-0547-46b6-aa80-6945f6ed12d&title=)

## TransportAPI

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565346159-710f123a-3dc1-4de6-9647-994e7c1e69b5.png#clientId=u80d56aec-d9dd-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u20501ffc&margin=%5Bobject%20Object%5D&originHeight=161&originWidth=531&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uab8ff667-94d5-4303-b333-d087e43ccb9&title=)

Transport 类实现了TransportInterface，这是真正实施消息发布的对象。每个Transport会有一个名字，一个类型标识，还会检查Locator的兼容性<br />应用不会直接使用Transport对象，而是使用TransportDescriptor对象来配置Transport，将该示例添加到参与者的Transport List中

```cpp
DomainParticipantQos qos;

// Create a descriptor for the new transport.
auto udp_transport = std::make_shared<UDPv4TransportDescriptor>();
udp_transport->sendBufferSize = 9216;
udp_transport->receiveBufferSize = 9216;
udp_transport->non_blocking_send = true;

// Link the Transport Layer to the Participant.
qos.transport().user_transports.push_back(udp_transport);

// Avoid using the default transport
qos.transport().use_builtin_transports = false;
```

Zero-copy is a combination of Data sharing and loans in both DataWriter and DataReader

### Locator

A Locator_t uniquely identifies a communication channel with a remote peer for a particular transport. For example, on UDP transports, the Locator will contain the information of the IP address and port of the remote peer.<br />Locator定义的是通信信道，具体讲就是对端IP和端口号，port， address<br />不同的Transport有不同的Locator，比如说SHM，address是一个ID，port指向一个共享环形缓冲区

### Listening Locator

Listening Locators are used to receive incoming traffic on the DomainParticipant. These Locators can be classified according to the communication type and to the nature of the data.<br />According to the communication type we have:

- **Multicast locators**: Listen to multicast communications.
- **Unicast locators**: Listen to unicast communications.

According to the nature of the data we have:

- **Metatraffic locators**: Used to receive metatraffic information, usually used by built-in endpoints to perform discovery.
- **User locators**: Used by the endpoints created by the user to receive user Topic data changes.

## SHM

The shared memory (SHM) transport enables fast communications between entities running in the same processing unit/machine, relying on the shared memory mechanisms provided by the host operating system.<br />SHM transport provides better performance than other network transports like UDP / TCP, even when these transports use loopback interface. This is mainly due to the following reasons:

- Large message support: Network protocols need to fragment data in order to comply with the specific protocol and network stacks requirements, increasing communication overhead. SHM transport allows the copy of full messages where the only size limit is the machine’s memory capacity.
  - 网络协议数据传输涉及到数据分片，这样通信开销比较大
- Reduce the number of memory copies: When sending the same message to different endpoints, SHM transport can directly share the same memory buffer with all the destination endpoints. Other protocols require to perform one copy of the message per endpoint.
  - 减少拷贝次数。当一对多通信时，可以直接共享内存缓冲区，其他的协议需要逐个拷贝数据
- Less operating system overhead: Once initial setup is completed, shared memory transfers require much less system calls than the other protocols. Therefore, there is a performance/time consume gain by using SHM.
  - 减少OS开销，一旦初始化完成，共享内存数据传输需要的OS调用比其他协议要少。

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565346142-8acd3547-30c2-4cee-954e-a335ffec29a1.png#clientId=u80d56aec-d9dd-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ueb83c860&margin=%5Bobject%20Object%5D&originHeight=381&originWidth=941&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u168bdd63-2198-4016-b271-f20a24e6e88&title=)

segment，段，是一块共享内存，可供多个进程访问，每一个参与者都配置了SHM传输方式，并且会创建一块内存。参与者将需要发送的数据写入共享内存。每一块内存段有一个segmentID，是一个唯一的标识<br />segment buffer, 端中分配的环形缓冲区，作为内存段中的DDS消息容器。每一个消息都在不同的buffer<br />buffer descriptor, 包括segmentID以及offset of the Segment<br />buffer port，标识buffer descriptor的一个channel，在shm中实现了一个环形缓冲区ring-buffer，

## data-sharing

**data-sharing和SHM transport是不一样的传输方式，其主要机制在于DataWriter和DataReader之间通过共享内存共享History**<br />DataShaing并不能避免数据从app到DataReader以及DataWriter的拷贝，而是避免了DataWriter到DataReader之间的拷贝<br />Although Data-sharing delivery uses shared memory, it differs from Shared Memory Transport in that Shared Memory is a full-compliant transport. **That means that with Shared Memory Transport the data being transmitted must be copied from the DataWriter history to the transport and from the transport to the DataReader. With Data-sharing these copies can be avoided.**

### 机制

DataWriter在创建时预分配一个mm文件，当发布数据时，DataWriter将数据拷贝如His中，通知DataReader到His中接收数据。DataReader会去访问共享内存文件。<br />公用一套History，强耦合会造成很多特殊场景。<br />With traditional Transport Layer delivery, the DataReader and DataWriter keep separate and independent histories, each one with their own copy of the sample. Once the sample is sent through the transport and received by the DataReader, the DataWriter is free to remove the sample from its history without affecting the DataReader.<br />With data-sharing delivery, the DataReader directly accesses the data instance created by the DataWriter. This means that the samples in both the history of the DataReader and the DataWriter refer to the same object in the shared memory. Therefore, there is a strong coupling in the behavior of the DataReader and DataWriter histories. If the DataWriter reuses the same sample to publish new data, the DataReader loses access to the old data sample.

### 限制

根据上述机制，该功能的限制有

- DataWriter和DataReader能够访问相同的共享内存文件
  - 好理解
- Topic具有bounded TopicDataType
  - History是一个payload pool, 共享内存需要预先分配内存
- Topic not keyed
  - 不太理解
- The DataWriter is configured with PREALLOCATED_MEMORY_MODE or PREALLOCATED_WITH_REALLOC_MEMORY_MODE.
  - 必须预先分配内存，所以是预先分配或者预先分配加重分配

### 配置

DataSharing模式

- **AUTO**: If both a DataWriter and DataReader meet the requirements, data-sharing delivery will be used between them. This is the default value.
- **ON**: Like **AUTO**, but the creation of the entity will fail if the requirements are not met.
- **OFF**: No data-sharing delivery will be used on this entity.

所以，对于bounded size topic, 默认走的是DataSharing

## ZeroCopy

### 机制

利用DataSharing功能，可以避免DataWriterHis到DataReaderHis之间的拷贝<br />利用DataWriter的borrow a data buffer功能。避免了从App到DataWriterHis的拷贝（注意，数据构建过程是不能避免的）<br />利用DataReader的loans from the DataReader，<br />在平常DataWriter写入数据，数据是拷贝到DataWriter的History中，也就是拷贝到内存中，对于大尺寸数据，拷贝需要消耗大量资源和cpu时间。在新的机制中，DataWriter从History内存中借一份内存到用户，用户填充进去，在发布的时候，DataWriter不需要拷贝，因为已经拥有了这个缓存了。

1. Get a reference to a loaned sample using loan_sample().
2. Use the reference to build the data sample.
3. Write the sample using write().

根据 [https://github.com/ros2/rmw_fastrtps/pull/568/files](https://github.com/ros2/rmw_fastrtps/pull/568/files)

## loan without data-sharing

loan与data-sharing可以解耦，<br />The only requirement on the Fast DDS API in order for loans to be used is that the type is plain.<br />This PR allows calling the loan APIs when data-sharing is OFF.<br />This enables performance improvements on intra-process and UDP.<br />loaned-msg with inter-machine? 这个会有性能问题吗？<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664278902749-38e7a362-945f-4bc8-bf67-bfc7ce3cf285.png#clientId=uec630e09-3c71-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=185&id=u58f4aa07&margin=%5Bobject%20Object%5D&name=image.png&originHeight=185&originWidth=904&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24893&status=done&style=none&taskId=ud54b56bd-ea9e-4c0f-a33b-9f5f8368715&title=&width=904)<br />Do you mean the use case with rclcpp when using loaned api and udp transport of dds?<br />I don't think there will be such an use case, people will definity use loaned api with zero-copy mechanism because loaned-api in rclcpp needs POD msg, and POD msg will meet

## Intra-process

eProsima Fast DDS allows to speed up communications between entities within the same process by avoiding any of the overhead involved in the transport layer. Instead, the Publisher directly calls the reception functions of the Subscriber. This not only avoids the copy or send operations of the transport, but also ensures the message is received by the Subscriber, avoiding the acknowledgement mechanism.

- 机制，publisher直接调用了Subscriber的接受函数
- 不仅仅去掉了Transport的拷贝，而且确认机制也内置了
- 这个功能是内置开启的

This feature is enabled by default, and can be configured using XML profiles. Currently the following options are available:

- **INTRAPROCESS_OFF**: The feature is disabled.
- **INTRAPROCESS_USER_DATA_ONLY**: Discovery metadata keeps using ordinary transport.
- **INTRAPROCESS_FULL**: Default value. Both user data and discovery metadata using Intra-process delivery.

注意：此处的Intra-process没有被ROS2使用，rclcpp使用自己的intra-process-comm绕过了dds通信

# dds拷贝分析汇总

DataWriterHis中的数据是已经序列化的数据（payload有效载荷）以及RTPS协议头，所以**没有包含序列化和解序列化的拷贝次数**<br />**ROS2没有使用到DDS的intra机制，所以此处不考虑**

| Transport | App-><br />DataWriterHis | DataWriterHis-><br />Transport | Transport->Transport<br />(RTPS layer) | Transport-><br />DataReaderHis | DataReaderHis-><br />App |
| --- | --- | --- | --- | --- | --- |
| UDP | 1 | 1 | 1... | 1 | 1 |
| SHM-default | 1 | 1 | 0 | 1 | 1 |
| DataSharing | 1 | 0 | 0 | 0 | 1 |
| ZeroCopy | 0 | 0 | 0 | 0 | 0 |

# 测试论证

Transport comparison. fastdds. Array2m.<br />注意：由于程序问题，该处udp传输方式实际上为shm<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664450826486-8efe9016-a86c-4168-be93-4a94f9c41a67.png#clientId=u88b80ba8-e671-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=578&id=u59ecdb98&margin=%5Bobject%20Object%5D&name=image.png&originHeight=578&originWidth=938&originalType=binary&ratio=1&rotation=0&showTitle=false&size=57889&status=done&style=none&taskId=u5406d62c-f926-4ea6-89c1-c0150d4335b&title=&width=938)

# 参考

[https://fast-dds.docs.eprosima.com/en/v2.3.5/02-formalia/titlepage.html](https://fast-dds.docs.eprosima.com/en/v2.3.5/02-formalia/titlepage.html)<br />[https://zhenshenglee.github.io/ros2_jetson_benchmarks/benchmark_results/galactic/pc_ros2_galactic_v5.13.0-41-generic_amd64/dds_compare/report_compare_transport__fastdds_Array2m.html](https://zhenshenglee.github.io/ros2_jetson_benchmarks/benchmark_results/galactic/pc_ros2_galactic_v5.13.0-41-generic_amd64/dds_compare/report_compare_transport__fastdds_Array2m.html)
