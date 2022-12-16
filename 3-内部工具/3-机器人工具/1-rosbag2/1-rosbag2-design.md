# 参考

[ros2/design](https://github.com/ros2/design/blob/ros2bags/articles/rosbags.md)

# rosbag2设计文档

## 动机

rosbag是非常必要的工具，可以录制和播放机器人系统数据

为了保证信息记录的高效，rosbag避免了串行和解串。

### rosbag in ros1

rosbag和rqt_bag都是一个层次的工具，包含两个基础工具包。rosbag_storage抽象了数据的存储和读取，ros client则定义不同的数据格式，不同数据格式支持压缩格式，提高了性能。

### ros2的场景

ros2引入了dds，引入了cdr的数据表示格式。不过rosbag2的设计同时考虑了更多格式的数据，比如protobuf和zeroMQ

## 技术需求

### 确定性

需要按顺序播放消息，通过加时间戳的方式来保证

### 适应性

能够转化消息的格式

### 可扩展性

支持1T 以上的存储大小

### 随机访问

访问消息中的任何一个元素

### 范围访问

可以访问一段时间内的消息元素

### 分块存储

为了性能可以分块存储

### 后向兼容

兼容ros1

## rosbag2设计

解耦成多个包了

三个层次的api，rosbag storage API, rosbag API, rosbag command line interface

The rosbag storage API is ROS 2 agnostic in a sense that it only reads and writes abstract binary messages, which are well defined given some meta information. The second layer, the rosbag API, is responsible for fetching ROS 2 messages, serialize and sufficiently describe them for the storage API to write them. Inversely, it receives the binary representation from the storage API and converts them into fully defined ROS 2 messages. Finally, the command line tool gives a user friendly entry point for steering the recording and replaying of rosbags.

![](https://tcs.teambition.net/storage/31279460d0470efd0459d0f0368aaedf6ac1?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg0NjY4MSwiaWF0IjoxNjcxMjQxODgxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc5NDYwZDA0NzBlZmQwNDU5ZDBmMDM2OGFhZWRmNmFjMSJ9.1uAsGzMJwPpx-5Mm0ENBRCRlZcO7f7Q9hfKx_6-Z8Xs&download=image.png "")

### rosbag API

ROS 2 is by design independent from its underlying middleware. That means, the incoming binary data can be encoded differently depending on the middleware. In the case of DDS (which is the ROS 2 default), the encoding is [CDR](https://en.wikipedia.org/wiki/Common_Data_Representation). Yet, every other middleware could use protocol buffers or similar to exchange data. Since ROS 2 is independent from its middleware, the stored data within a ROS 2 rosbag shall be independent and interchangeable as well. To achieve this, we utilize a plugin system which allows to convert ROS 2 messages to a specific encoding. This rosbag API shall thus provide a generic function which can (de-)serialize any ROS 2 message form/to an available encoding. By default, we provide such a plugin for CDR. With that, every middleware can use this function to deserialize CDR to ROS 2 and then further serialize to its own encoding, if necessary. We see this - introducing a pluggable serialization format - as the only way to provide generic rosbags. We also clearly see that this introduces a significant overhead. We therefore make this generalization optional to ensure best writing/reading performance when using the same middleware for recording and replaying. When no encoding is enforced, the middleware specific binary data will be stored as received without any further processing. Without this common serialization, no bags can easily be exchanged between multiple middlewares, however the same functionality can be used to convert bags offline. In that case, a non-generic bag can be processed offline and serialized to any encoding without hitting any performance constraints.

rosbag2按照道理讲，应该是middleware无关的，但是序列化和反序列化太耗时，所以就没这么设计了，不执行序列和反序列化，这样只能在一个类型的middleware中执行播放

设计了一个插件可以离线转换序列化方式，这样就可以规避播放时的性能问题。

所以，默认情况下，录制序列化后的数据，当选择特定的格式时，使用插件将当前序列化后的数据转化为指定格式的数据，指定格式的序列化后数据

![](https://tcs.teambition.net/storage/3127a8fd2cfafe0942acbc8816788c8416cd?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg0NjY4MSwiaWF0IjoxNjcxMjQxODgxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdhOGZkMmNmYWZlMDk0MmFjYmM4ODE2Nzg4Yzg0MTZjZCJ9.D6NBzlaDU9_emvfTl0Cj4X1wu5KgIqjJ5x0FesXOT88&download=image.png "")

# rosbag2适配iceoryx

只要串行化方法一致，就可以

所以只有使用iceoryx串行化方法的才行

只有通过iceoryx录制的，才能够使用iceoryx播放

## 录制时指定编码方式

```bash
RMW_IMPLEMENTATION=rmw_<middleware> ros2 bag record --encoding <encoding_identifier>
```

### rosbag2离线convert

插件开发，不需要开发，直接使用rmw中的接口即可

```text
ros2 bag convert <bag_file_a> <bag_file_b> --encoding <encoding_b> --format <format_b>
```

该功能只在rolling才有，foxy都没有这功能
