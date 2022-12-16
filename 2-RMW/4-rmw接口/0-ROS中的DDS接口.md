# ros on dds

## 参考

<https://design.ros2.org/articles/ros_on_dds.html>

## Why consider DDS

分布式的机制

ROS1自定义了串行化格式，自定义的传输协议，自定义的集中发现机制。

当探寻下一代ROS的通信系统时，初步的选项是要么增强ROS1的传输协议，使用zeroMQ，Protocol Buffer，zeroconf等实时队列，但是这些都意味着要从底层自定义一个中间件。而我们在调研的过程中发现了一个端到端的中间件，就是DDS

zs: 这是一个有众多实现，在IOT业界广泛使用的分布式通信机制，也符合ROS的需求，ROS就直接复用了。如果有实力，像百度，就自己做了。

关于DDS有一个中文文献 数据分发服务以数据为中心的发布订阅式通信 [任昊利，张少扬 编著] 2014年版.pdf

讲的是opendds，但是ROS使用的是fast-RTPS，这个更加简洁，清凉化。

## 端到端的中间件

直接复用的好处就是维护更少的代码，而且有明确的规范保证稳定的API。

缺点也是有的，那就是ROS2必须以已有的设计为基础来设计，如果一些功能DDS满足不了，那ROS也就无法提供对应的功能，也许需要绕过DDS的一些设计，不过不容易。

## 什么是DDS

DDS provides a publish-subscribe transport which is very similar to ROS’s publish-subscribe transport.

DDS提供发布订阅机制，和ROS的发布订阅机制很像。

DDS使用IDL来描述消息和串行化。DDS也可以实现请求响应机制。叫做DDS-RPC

DDS默认的发现系统，需要使用DDS的订阅发布机制，并且是分布式的，去中心化的。这样就不要master了，容错性较好，灵活性好。

## DDS从哪里来

公司希望软件系统与软件系统有好的互操作性，打破信息孤岛，构建企业信息化应用建设。DDS规范由OMG编写，OMG还提供了UML，CORBA，SysML。

报上OMG的大腿有好处也有坏处，好处就是有了一个国际性的标准组织可以规范，坏处就是OMG太庞大，要考虑的因素太多，无法快速适应变化的时代。

对DDS的担忧，DDS规范太老了，用在过时的系统上？这种印象是因为OMG的产品，CORBA和UML过时了。不过DDS貌似还正直壮年，有活跃的更新，甚至有C++11的接口定义。

## 技术上的信赖

DDS已经用在非常多的场合：

战场；大型装备安装；金融系统；宇航系统；飞行系统；火车调度系统。DDS的用户包括美国政府和NASA，他们都赞赏DDS的灵活性和可靠性。不过这些特性来自于它的复杂性，所以开发难度会很大。这些问题在ROS的包装下也许会得到改善。

DDS的连接规范（DDSI-RTPS）非常灵活，允许构建可靠的，高层系统集成，实时应用。不同的DDS厂家，对不同的应用场景实现了不同的DDS。

DDS默认是走在UDP上，不过它并不依赖可靠的通信，所以DDS重新设计了一套可靠通信机制（基于TCP，不过裁剪了）。

对于可靠性，DDS支持QOS，控制通信的优先级。如果你担心延迟，比如需要软实时，那么你把DDS的QOS调城UDP Blaster，在其他场合你也许需要像TCP一样的服务，只需要修改QOS的参数就可以了。

两个厂家已经实现了DDS OVER TCP（RTI and ADLINK）

### 主要的厂商

RTI-装机数量最多

ADLINK Technologies- OpenSplice glibc和zeromq，qt用的是这个

Twin Oaks Software

[http://dds-directory.omg.org/vendor/list.htm](http://dds-directory.omg.org/vendor/list.htm)

eProsima Fast RTPS -- 比较新，轻量化，实现RTPS是DDS的一个子集

### 层次化结构

![](https://tcs.teambition.net/storage/3126883affa1e0018ddf07b319c4e0fe8cf1?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODcyMiwiaWF0IjoxNjcxMTkzOTIyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY4ODNhZmZhMWUwMDE4ZGRmMDdiMzE5YzRlMGZlOGNmMSJ9.OyyseQ3WbLdtvXo5Sjsb_zyABFGFu_QfpMmpH_WIX_I&download=image.png "")

不推荐用户代码直接操纵DDS 的API，因为会损失移植性

### 消息

ros1的消息类型用了很多年，也很好用，被继承下来。

### Services and Actions

目前DDS没有RPC实现，现在已经实现了

## 结论

After working with DDS and having a healthy amount of skepticism about the ethos, community, and licensing, it is hard to come up with any real technical criticisms. While it is true that the community surrounding DDS is very different from the ROS community or the ZeroMQ community, it appears that DDS is just solid technology on which ROS could safely depend. There are still many questions about exactly how ROS would utilize DDS, but they all seem like engineering exercises at this point and not potential deal breakers for ROS.

## 当前中间件

### iceoryx-来自bosch

iceoryx - an IPC middleware for POSIX-based systems

It features shared memory capabilities which allow a true zero-copy data transfer.

### Vortex OpenSplice Community Edition

### eProsima's Fast-RTPS

西班牙

默认实现，不过有两种连接方式，静态和动态

Robotics: ROS (Robotic Operating System) as their default middleware for ROS2.

EU R&D: FIWARE Incubated GE.

### RTI's Connext DDS

Real-Time Innovations (RTI) is the Industrial Internet of Things (IIoT) connectivity company.

### Intel's Distributed Publish & Subscribe for IoT (DPS)

[https://intel.github.io/dps-for-iot/](https://intel.github.io/dps-for-iot/)

### gurumdds

guru networks一个网络厂商
