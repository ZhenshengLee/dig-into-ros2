# Transport Layers

In general the latency of the data transport increases depending on the “distance” of the communication participants. The “closer” the participants, the “quicker” the transport. Interhost communication requires using a network stack to transport said data, which generates an overhead and latency. Transport on the same host is significantly faster, as shared memory techniques may be applied. Intraprocess communication may be fastest, as it may require as little as sharing a pointer.

## 可用的传输机制

| Layer  | ini parameter          | Physical Layer | Comment                                                                                   |
| ------ | ---------------------- | -------------- | ----------------------------------------------------------------------------------------- |
| inproc | [publisher/use_inproc] | inner process  | inner process, zeroy copy communication (pointer forwarding)                              |
| shm    | [publisher/use_shm]    | shared memory  | interprocess, shared memory communication, supports N:M connections, 2 memory copies      |
| udp_mc | [publisher/use_udp_mc] | udp multicast  | interhost, topic name based dynamic multicast grouping to optimize pub/sub socket payload |
| tcp    | [publisher/use_tcp]    | tcp            | Network (interhost), simulates N:M connections. Meant for single large payloads.          |

默认机制如下

| Header 1                   | inproc    | shm     | udp_mc    | tcp       |
| -------------------------- | --------- | ------- | --------- | --------- |
| Intraprocess Communication | available | default | available | available |
| Interprocess Communication |           | default | available | available |
| Interhost Communication    |           |         | default   | available |

## SHM

The eCAL Shared Memory (SHM) Layer is the default transport layer for publishers and subscribers running on the same machine. It is based on a shared memory space (therefore the name) that is used to write the data to and again read it from. This technique ensures both low latency and high throughput - as long as the publisher and subscriber are running on the same machine.

### 机制

By default, the logic is implemented in the following way:

Initialization phase:

- The publisher creates a memory file fitting to the size of the current payload.

  - 创建一个mm file，这个payload已经是序列化后的数据了

  - 我觉得吧，

- The publisher creates a named shared mutex and a named event used as synchronization primitives.

  - 创建同步机制

- The publisher informs all matching subscribers about the name of the memory file, the matching mutex and the update event.

  - publisher 通知所有subscriber mm file的路径，同步机制的标识

- The subscriber starts observing the memory file for updates by listening to the update event.

  - subscriber监听同步事件

Communication phase (default configuration):

- The publisher acquires the memory file access-mutex and opens the memory file.

  - 打开mm file

- The publisher writes the payload content and some header information into the memory file.

  - 写入payload和header

- The publisher fires the update event, closes the memory file and unlocks the mutex.

  - 触发事件，关闭mm file

- The subscribers are signaled by the update event and acquire the access-mutex.

  - subscriber收到事件

- The subscribers open the memory file and copy the payload into their process memory.

- The subscribers close the memory file and release the access-mutex.

To support one to many publisher/subscriber connections the publisher creates in fact one named update event per connection.

这整个机制是有拷贝的，2次拷贝，本质上跟线程间共享数据是一样的

注意：如果数据频率高过拷贝时间，则可能出现subscriber拷贝没有结束，publisher就写入新的数据，这样publisher会覆盖旧数据，而subscriber会收到数据dropped的事件

## zero-copy

The “normal” eCAL Shared memory communication results in the payload being copied at least twice:

1. Into the SHM file by the publisher

1. From the SHM file the private memory of each subscriber

对于特别巨大的消息，比如高分辨率图像，在zero-copy模式下

1. The publisher still has to copy the data into the memory file.

1. The subscriber executes its callback directly on the memory file. The memory file is blocked, while being used.

1. The subscriber releases the memory file, so the publisher can update it again.

The memory file is blocked for new publications as long as the user’s callback is processing its content. It will also block other subscribers from reading the same SHM file.

Even though it is called zero-copy, only the subscribers are zero-copy. Publishers still have to copy the data into the memory file, as they have to also support other layers like UDP or TCP and therefore cannot directly work on the memory file.

# rmw_ecal的拷贝

Yeah, its currently experimental feature (we are currently working on a way to remove disadvantages for small messages), but that shouldn't affect the RMW that much (if at all) mainly because RMW has to copy message to its own layer at the moment

so the current behaviour is.

eCAL copies it into its layer -> RMW copies that copy in its layer

with zero copy enabled it would be

eCAL provides ptr to shm mapped address -> RMW copies it in its layer

with zero copy it would just emulate how eCAL works without RMW.

# 拷贝次数分析

## ROS2拷贝分析汇总

| Transport     | msgadaption | rclcpppublish | rmwserial | rmw_ddspub/sub | rmwdeserial | rclcppsubscribe | msgadaption |
| ------------- | ----------- | ------------- | --------- | -------------- | ----------- | --------------- | ----------- |
| default       | 1           | 1             | 1         | dds_shm        | 1           | 1               | 1           |
| loaned_api    | 1           | 1             | 1         | dds_shm        | 0           | 1               | 1           |
| intra-process |             |               |           | 0              |             |                 |             |

## dds拷贝分析汇总

DataWriterHis中的数据是已经序列化的数据（payload有效载荷）以及RTPS协议头

**没有包含序列化和解序列化的拷贝次数**

| Transport |     |     |     |     |     |
| --------- | --- | --- | --- | --- | --- |
| IP        |     |     |     |     |     |
| SHM       |     |     |     |     |     |
| ZeroCopy  |     |     |     |     |     |
