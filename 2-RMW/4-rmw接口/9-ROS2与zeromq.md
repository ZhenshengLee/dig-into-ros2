# 从零到一搭建中间件

## 参考

[https://design.ros2.org/articles/ros_with_zeromq.html](https://design.ros2.org/articles/ros_with_zeromq.html)

## 简介

这篇文章讲解了使用zeromq和其他库实现一个新的，现代的ROS中间件

将已有的一些库组织起来构建一个新中间件，这是一个策略

不过OSRF采用了另一个策略，就是寻找一个端到端的中间件，然后在上面做wrapper，满足ROS的接口需要。

## 从0到1搭建中间件

ROS的需求，一个中间件需要提供一些关键服务。

第一个就是运行时刻的软件组件动态发现和连接服务。

然后，中间件需要提供软件组件间交换信息的多种方式。典型的就是订阅发布机制，请求响应机制（能够在订阅发布机制之上实现）。

最后，中间件提供消息定义和发送机制，如串行化和解串化（因为网络通信都是流式传输），这个已经有很多组件可以完成。

### 服务发现

第一个想到的方案就是Zeroconf with avahi/bonjour, The core technology here is mDNSresponder, which is provided by Apple as free software, and is used by both Bonjour (OS X and Windows) and Avahi (Linux, specifically avahi-compat)。苹果用的就是这个发现协议。

但是，zeroconf方案被证明是不可靠的，当设备数量增加到20的时候就不稳定了。

[https://bitbucket.org/osrf/disc_zmq/src](https://bitbucket.org/osrf/disc_zmq/src)

基于zmq的发现系统，是比较简单的

这个系统，使用组播UDP包来发布通知，比如“Node started”, “Advertise a publisher”, and “Advertise a subscription”。

For discovery the first solution that was investigated was [Zeroconf](http://en.wikipedia.org/wiki/Zero_configuration_networking) with Avahi/Bonjour. Some simple experiments were conducted which used [pybonjour](https://code.google.com/p/pybonjour/) to try out using the Zeroconf system for discovery. The core technology here is `mDNSresponder`, which is provided by Apple as free software, and is used by both Bonjour (OS X and Windows) and Avahi (Linux, specifically avahi-compat).

These Zeroconf implementations, however, proved to not be so reliable with respect to keeping a consistent graph between machines. Adding and removing more than about twenty items at a time from subprocesses typically resulted in inconsistent state on at least one of the computers on the network. One particularly bad case was the experiment of removing items from Zeroconf, where in several “nodes” were registered on machine A and then after a few seconds shutdown cleanly. The observed behavior on remote machines B and C was that the Zeroconf browser would show all “nodes” as registered, but then after being shutdown only some would be removed from the list, resulting in “zombie nodes”. Worse still is that the list of “zombie nodes” were different on B and C. This problem was only observed between machines using avahi as a compatibility layer, which lead into a closer look into avahi and its viability as a core dependency. This closer look at avahi raised some concerns about the quality of the implementation with respect to the [Multicast DNS](http://en.wikipedia.org/wiki/Multicast_DNS) and [DNS Service Discovery (DNS-SD)](http://en.wikipedia.org/wiki/Zero_configuration_networking#Service_discovery) technology.

Furthermore, DNS-SD seems to prefer the trade-off of light networking load for eventual consistency. This works reasonably well for something like service name look up, but it did not work well for quickly and reliably discovering the proto-ROS graph in the experiments. This led to the development of a custom discovery system which is implemented in a few languages as part of the prototype here:

[https://bitbucket.org/osrf/disc_zmq/src](https://bitbucket.org/osrf/disc_zmq/src)

The custom discovery system used multicast UDP packets to post notifications like “Node started”, “Advertise a publisher”, and “Advertise a subscription”, along with any meta data required to act, like for publishers, an address to connect to using ZeroMQ. The details of this simple discovery system can be found at the above URL.

This system, though simple, was quite effective and was sufficient to prove that implementing such a custom discovery system, even in multiple languages, is a tractable problem.

### 数据传输

or transporting bytes between processes, a popular library is ZeroMQ, but there are also libraries like nanomsg and RabbitMQ.

但是，zmq，依赖TCP或PGM，所以对于软实时场景不是很适合。

For transporting bytes between processes, a popular library is [ZeroMQ](http://zeromq.org/), but there are also libraries like [nanomsg](http://nanomsg.org/) and [RabbitMQ](http://www.rabbitmq.com/). In all of those cases the goal of the library is to allow you to establish connections, explicitly, to other participants and then send strings or bytes according to some communication pattern. ZeroMQ is an LGPL licensed library which has recently become very popular, is written in C++ with a C API, and has bindings to many languages. nanomsg is a new MIT licensed library which was created by one of the original authors of ZeroMQ, is written in C with a C API, but is far less mature than ZeroMQ. RabbitMQ is a broker that implements several messaging protocols, mainly AMQP, but also provides gateways for ZeroMQ, STOMP and MQTT. By being a broker, it meets some of the discovery needs as well as the transport needs for ROS. Although ZeroMQ is usually used in brokerless deployments, it can also be used in conjunction with RabbitMQ to provide persitence and durability of messages. RabbitMQ is licensed under the Mozilla Public License. All of these libraries could probably be used to replace the ROSTCP transport, but for the purposes of this article we will use ZeroMQ in a brokerless deployment.

In this prototype:

[https://bitbucket.org/osrf/disc_zmq/src](https://bitbucket.org/osrf/disc_zmq/src)

ZeroMQ was used as the transport, which conveniently has bindings in C, C++, and Python. After making discoveries using the above described simple discovery system, connections were made using ZeroMQ’s `ZMQ_PUB` and `ZMQ_SUB` sockets. This worked quite well, allowing for communication between processes in an efficient and simple way. However, in order to get more advanced features, like for instance latching, ZeroMQ takes the convention approach, meaning that it must be implemented by users with a well known pattern. This is a good approach which keeps ZeroMQ lean and simple, but does mean more code which must be implemented and maintained for the prototype.

Additionally, ZeroMQ, in particular, relies on reliable transports like TCP or [PGM (Pragmatic General Multicast)](http://en.wikipedia.org/wiki/Pragmatic_General_Multicast), so it makes it unsuitable for soft real-time scenarios.

### 消息串行化

google protobuf

In ROS 1, messages are defined in `.msg` files and code is generated at build time for each of the supported languages. ROS 1 generated code can instantiate and then later serialize the data in a message as a mechanism for exchanging information. Since ROS was created, several popular libraries which take care of this responsibility have come about. Google’s [Protocol Buffers (Protobuf)](https://code.google.com/p/protobuf/), [MessagePack](http://msgpack.org/), [BSON](http://bsonspec.org/), and [Cap’n Proto](http://kentonv.github.io/capnproto/) are all examples of serialization libraries which have come to popularity since ROS was originally written. An entire article could be devoted to the pros and cons of different message definition formats, serialization libraries, and their wire formats, but for the purposes of this prototype we worked with either plain strings or Protobuf.

## 结论

首先，没有一个专门满足ROS需求的中间件。自己实现一个发现系统，可能，但是十分费时

然后，有一系列的好软件，可以集成到发现系统中去，包括数据传输和串行工具。另外zmq还提供了很好的进程内通信机制。
