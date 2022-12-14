# 架构

The goal is to make DDS an implementation detail of ROS 2. This means that all DDS specific APIs and message definitions would need to be hidden. DDS provides discovery, message definition, message serialization, and publish-subscribe transport. Therefore, DDS would provide discovery, publish-subscribe transport, and at least the underlying message serialization for ROS. ROS 2 would provide a ROS 1 like interface on top of DDS which hides much of the complexity of DDS for the majority of ROS users, but then separately provides access to the underlying DDS implementation for users that have extreme use cases or need to integrate with other, existing DDS systems.

![](https://tcs.teambition.net/storage/312gb42d182196fcc633588799db95d27cdf?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjM5NSwiaWF0IjoxNjcxMDE3NTk1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdiNDJkMTgyMTk2ZmNjNjMzNTg4Nzk5ZGI5NWQyN2NkZiJ9.C1DimOl_fdXtiF-yQvl4TtEWFOS7wOVrfgG4MQmb1zY&download=image.png "")

# 发布订阅中的概念映射

The DDSI-RTPS (DDS-Interoperability Real Time Publish Subscribe) protocol would replace ROS’s TCPROS and UDPROS wire protocols for publish/subscribe. The DDS API provides a few more actors to the typical publish-subscribe pattern of ROS 1. In ROS the concept of a node is most clearly paralleled to a graph participant in DDS. A graph participant can have zero to many topics, which are very similar to the concept of topics in ROS, but are represented as separate code objects in DDS, and is neither a subscriber nor a publisher. Then, from a DDS topic, DDS subscribers and publishers can be created, but again these are used to represent the subscriber and publisher concepts in DDS, and not to directly read data from or write data to the topic. DDS has, in addition to the topics, subscribers, and publishers, the concept of DataReaders and DataWriters which are created with a subscriber or publisher and then specialized to a particular message type before being used to read and write data for a topic. These additional layers of abstraction allow DDS to have a high level of configuration, because you can set QoS settings at each level of the publish-subscribe stack, providing the highest granularity of configuration possible. Most of these levels of abstractions are not necessary to meet the current needs of ROS. Therefore, packaging common workflows under the simpler ROS-like interface (Node, Publisher, and Subscriber) will be one way ROS 2 can hide the complexity of DDS, while exposing some of its features.

DDSI-RTPS协议用来替代TCPROS，UDPROS。但是DDS 的API比ROS的要复杂一些。

在ROS中node的概念和DDS中的参与者是最接近的

DDS计算图参与者可以有很多个话题，这个概念与ROS中的Topic很像，但是DDS中的参与者分解成了多个对象，不是简单的发布者和订阅者，而是publishers，subscribers，DataReaders，DataWriters，这个额外的层次的好处在于提供更大细粒度的解耦，从而可进行更细化的QoS配置，不过这些配置中有些过于复杂，不符合ROS的需求。所以ROS2会在这一块梳理出合适的流程，并且提供良好的封装。

# 更有效率的数据传输

In ROS 1 there was never a standard shared-memory transport because it is negligibly faster than localhost TCP loop-back connections. It is possible to get non-trivial performance improvements from carefully doing zero-copy style shared-memory between processes, but anytime a task required faster than localhost TCP in ROS 1, nodelets were used. Nodelets allow publishers and subscribers to share data by passing around `boost::shared_ptr`s to messages. This intraprocess communication is almost certainly faster than any interprocess communication options and is orthogonal to the discussion of the network publish-subscribe implementation.

在ROS1中没有共享内存的通信，这个比TCP loopback方式数据传输要快。想要在进程间做共享内存，很麻烦（tzc），一般是用nodelet机制，通过shared_ptr放在消息中，在子节点间共享。这种进程内通信方式已经比任何进程间通信方式要快。

In the context of DDS, most vendors will optimize message traffic (even between processes) using shared-memory in a transparent way, only using the wire protocol and UDP sockets when leaving the localhost. This provides a considerable performance increase for DDS, whereas it did not for ROS 1, because the localhost networking optimization happens at the call to `send`. For ROS 1 the process was: serialize the message into one large buffer, call TCP’s `send` on the buffer once. For DDS the process would be more like: serialize the message, break the message into potentially many UDP packets, call UDP’s `send` many times. In this way sending many UDP datagrams does not benefit from the same speed up as one large TCP `send`. Therefore, many DDS vendors will short circuit this process for localhost messages and use a blackboard style shared-memory mechanism to communicate efficiently between processes.

在DDS中，大量厂商已经使得共享内存传输功能得以透明化使用，只需要使用UDP就可以了，这样带来了性能大提升。在ROS1做不到这点，因为localhost的loopback的性能只在send层次有局部优化。对于ROS1的流程来讲：serialize the message into one large buffer, call TCP’s `send` on the buffer once.串行化，然后send。但是DDS进程来讲：serialize the message, break the message into potentially many UDP packets, call UDP’s `send` many times. 串行化，消息分片，使用udp send发送多次。这种情况下，性能比tcp send一次不好，所以大多数厂商会使用黑盒功能共享内存。

However, not all DDS vendors are the same in this respect, so ROS would not rely on this “intelligent” behavior for efficient **intra**process communication. Additionally, if the ROS message format is kept, which is discussed in the next section, it would not be possible to prevent a conversion to the DDS message type for intraprocess topics. Therefore a custom intraprocess communication system would need to be developed for ROS which would never serialize nor convert messages, but instead would pass pointers (to shared in-process memory) between publishers and subscribers using DDS topics. This same intraprocess communication mechanism would be needed for a custom middleware built on ZeroMQ, for example.

对于进程内通信来讲，ROS不能指导DDS厂商帮我们做好。而且，只要ROS消息一直有定义，则ROS消息到DDS消息的转换开销一直存在。所以还是需要一个自定义的进程内通信，不需要消息转换和串行化环节，而且同样可以通过dds topics转发指针。

The point to take away here is that efficient **intra**process communication will be addressed regardless of the network/interprocess implementation of the middleware.

需要指出的是，高效率的进程内通信，与中间件的网络和进程间通信实现没有关系。

# ros_dds原型

![](https://tcs.teambition.net/storage/312g1f37d2ae7865d1597adcbe22739eeb51?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjM5NSwiaWF0IjoxNjcxMDE3NTk1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmcxZjM3ZDJhZTc4NjVkMTU5N2FkY2JlMjI3MzllZWI1MSJ9.bsGcyEndgxRO4uzEmM972qrTanMIBO-r4FtCeuzW3t0&download=image.png "")

总的逻辑是：

- 将msg转换为idl

- 将idl转换为cpp头文件，包含消息对象

- 将数据发送出去

  - 将node，topic，publisher，subscriber等rcl概念映射到dds概念

# 参考

[https://design.ros2.org/articles/ros_on_dds.html](https://design.ros2.org/articles/ros_on_dds.html)
