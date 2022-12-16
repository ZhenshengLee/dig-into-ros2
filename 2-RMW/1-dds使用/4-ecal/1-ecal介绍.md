# eCAL大陆集团出的DDS

2021.06已经推出1.0版本，包含大量自有工具，全部开源

## 参考

<https://www.reddit.com/r/ROS/comments/imznn1/new_ros2_middleware_layer_rmw_ecal/>

<https://github.com/continental/ecal>

[Welcome to eCAL’s documentation! — eCAL Documentation  documentation](https://continental.github.io/ecal/)

## 概述

包括了iceoryx的特性，可以做到单机零拷贝，多机UDP通信

拥有自己的一套消息录制和播放工具

## 安装

```text
sudo add-apt-repository ppa:ecal/ecal-latest
sudo apt-get update
sudo apt-get install ecal
```

### 源代码编译

使用iceoryx layer功能

```bash
sudo apt-get install git cmake doxygen graphviz build-essential \
                    zlib1g-dev qt5-default libhdf5-dev libprotobuf-dev
                    libprotoc-dev protobuf-compiler libcurl4-openssl-dev

mkdir _build
cd _build
cmake .. -DCMAKE_BUILD_TYPE=Release -DECAL_THIRDPARTY_BUILD_PROTOBUF=OFF -DECAL_THIRDPARTY_BUILD_CURL=OFF -DECAL_THIRDPARTY_BUILD_HDF5=OFF -DECAL_LAYER_ICEORYX=ON -DHAS_ROS2=ON


# -DBUILD_SHARED_LIBS=ON -Dprotobuf_BUILD_SHARED_LIBS=ON

make -j10

# package and install
cpack -G DEB
sudo dpkg -i _deploy/eCAL-*

```

### 验证

```bash
# 执行ecal的 sample即可
# t1
iox-roudi
# t2
ecal_sample_person_snd
# t3
ecal_sample_person_rec
```

## 使用

There are currently two rmw implementations

- rmw_ecal_dynamic_cpp

- rmw_ecal_proto_cpp

dynamic_cpp

Pros:

- Faster than rmw_ecal_proto_cpp

- Plug&Play doesn't require any additional steps/ros packages to work

Cons:

- Doesn't integrate well into eCAL ecosystem (monitor will only show binary data for messages and native eCAL nodes won't be able to deserialize its data)

还是使用第一个，`export RMW_IMPLEMENTATION=rmw_ecal_dynamic_cpp`

```text
export RMW_IMPLEMENTATION=rmw_ecal_dynamic_cpp
ros2 run demo_nodes_cpp talker

ros2 run demo_nodes_cpp listener

```

## 功能

### 版本支持

- Dashing Diademata

- Eloquent Elusor

- Foxy Fitzroy

### QoS支持

完整

### 对SHM支持

通过iceoryx支持，自带支持

### 安全支持

没有

### 特色：其他工具支持

GUI VIEW

## 性能

![](https://tcs.teambition.net/storage/3126aa90ae9ee407e2cbc8f1ad19af8c396e?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk0MSwiaWF0IjoxNjcxMTkwMTQxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZhYTkwYWU5ZWU0MDdlMmNiYzhmMWFkMTlhZjhjMzk2ZSJ9.WFhYV_5Ezqk33TJrvvE2z0Z1jd9-Z7HKMakT4TlSpz8&download=image.png "")

| Payload Size (kB) | Win10 AMD64 (µs)eCAL SHM | Ubuntu18 AMD64 (µs)eCAL SHM | Ubuntu18 AMD64 (µs)Iceoryx SHM |
| ----------------- | ------------------------ | --------------------------- | ------------------------------ |
| 1                 | 10                       | 4                           | 6                              |
| 2                 | 10                       | 4                           | 6                              |
| 4                 | 10                       | 5                           | 6                              |
| 8                 | 11                       | 5                           | 6                              |
| 16                | 12                       | 6                           | 6                              |
| 32                | 13                       | 7                           | 8                              |
| 64                | 16                       | 10                          | 10                             |
| 128               | 21                       | 15                          | 13                             |
| 256               | 32                       | 33                          | 19                             |
| 512               | 56                       | 50                          | 28                             |
| 1024              | 103                      | 154                         | 82                             |
| 2048              | 363                      | 392                         | 177                            |
| 4096              | 867                      | 877                         | 420                            |
| 8192              | 1814                     | 1119                        | 534                            |
| 16384             | 3956                     | 2252                        | 1060                           |

## 问题和后续工作

### foxy版本proto报错

![](https://tcs.teambition.net/storage/31278d73bbf4011bae835ad559ffab650ef5?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk0MSwiaWF0IjoxNjcxMTkwMTQxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc4ZDczYmJmNDAxMWJhZTgzNWFkNTU5ZmZhYjY1MGVmNSJ9.-tRXBe9dibtSKR2OaRKG8Fel-vys7LWJxo8fxCnZkFk&download=image.png "")

### 不支持ros2cli和rqt

![](https://tcs.teambition.net/storage/3126749b800057567b37634bd4df72c7cd5c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk0MSwiaWF0IjoxNjcxMTkwMTQxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY3NDliODAwMDU3NTY3YjM3NjM0YmQ0ZGY3MmM3Y2Q1YyJ9.BkyX1-BRjwsDmzlhZghZjf678gVFof_WiLBQxo4g7r8&download=image.png "")

# 结论

- 如果使用基于protobuf版本的

  - 那么新增消息必须重新编译生成protobuf定义

  - 无法使用 rviz和 rosbag，这才是最大的问题

  - 可以使用 ecal_mon_gui查看数据的节点，但是无法查看点云图像，问题很大

  - 根本原因还是在于 ros2官方不支持 protobuf

- 如果使用默认版本的

  - 则需要扩充rmw功能开发

    - ecal的 api仔细查看允许通过 rmw获取 introspection信息

    - 参考 iceoryx和 rmw_ecal

  - ecal提供工具可以查看节点信息，但是无法查看数据定义

    - 软件架构提供插件机制进行 data reflection

    - ecal_mon基本可以代替 rqt_graph和ros topic

  - rviz和 rqt都可以使用，这个是最重要的

个人建议选择默认版本并扩充rmw功能

# 实践

## 查看ros2消息

必须事先知道类型

[How to print a message to stdout in ROS2? - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/365776/how-to-print-a-message-to-stdout-in-ros2/)

[What's the best way to convert a ros message to a string or xml? - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/10330/whats-the-best-way-to-convert-a-ros-message-to-a-string-or-xml/)

In C++ you don't have introspection for messages, i.e. you need to know the message types at compile time. In theory, it would be possible to deserialize messages into something dynamic such as a stl map over strings and boost::any or something but I don't know of any implementation.

Python is much easier because it's a dynamic language. You can import messages at run-time and you can introspect instances of any class at run-time. Consider the following example:

```text
import roslib; roslib.load_manifest('geometry_msgs')
import geometry_msgs.msg

p = geometry_msgs.msg.Vector3(1, 2, 3)
print p.__slots__
print p._slot_types
print p.__getattribute__('x')
```

It shows how to list all slots, i.e. member variables of a ros message, how to get the types of all slots and how to get the value of a slot. With that it should be pretty easy to write something generic to create an xml representation of messages.

正确方法是这个，to_yaml(msg)

[add to_yaml() function for C++ messages (take 2) by dirk-thomas · Pull Request #527 · ros2/rosidl](https://github.com/ros2/rosidl/pull/527)

该特性没有包含在foxy版本中，2020.06版本，合并到了galactic中

rosidl不能单独升级，必须统一构建发布。

拷贝字节到结构体的方法

[Put data from a QByteArray in a Struct](https://stackoverflow.com/questions/35972301/put-data-from-a-qbytearray-in-a-struct)

![](https://tcs.teambition.net/storage/31281d6e5c48fbd01e1f6ae27565b16c7c24?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk0MSwiaWF0IjoxNjcxMTkwMTQxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjgxZDZlNWM0OGZiZDAxZTFmNmFlMjc1NjViMTZjN2MyNCJ9.KLD3CguJk1uBAYwawPF-MhgKj7yNNNTLDNE6JwUPwgA&download=image.png "")

## 动态查看数据

![](https://tcs.teambition.net/storage/312b2584ada7620cab6bd316e200438264ec?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk0MSwiaWF0IjoxNjcxMTkwMTQxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmIyNTg0YWRhNzYyMGNhYjZiZDMxNmUyMDA0MzgyNjRlYyJ9.fWJrIoV_3RtwrMod2SE8KlOcOZiHDfqYGz_FyR_EdBY&download=image.png "")

[dynamic_message_introspection/dynmsg_demo at add-cpp-support-and-refactor · christophebedard/dynamic_message_introspection](https://github.com/christophebedard/dynamic_message_introspection/tree/add-cpp-support-and-refactor/dynmsg_demo)

必须通过rmw获取type实现动态查看消息数据

# 关于ecal数据类型

Hi,

thank you for your nice feedback. Simplicity is one of the major API design decision of eCAL. Regarding the messages type support eCAL is quite different to ROS/ROS2. It's not providing any kind of message collection at all. It's middleware only, not focusing on a specific use case like robotics or AD.

To create a google protobuf message file based on a ROS message file should be an easy task. And using google protobuf is not the only option, there are other message protocols (CapnProto, Flatbuffers) that you can utilize for your use case too. But protobuf is a good choice for many use cases, it's like a swiss army knife ;-).

If there is a general interest to have a standardized collection of messages for the robotic use case, maybe it would be a good idea to collect them in a separate repository.

Hi,

yes this is completely independent. You can choose different kind of publisher / subscriber (raw data blobs, protobuf, capnproto, flattbuffers, strings) and how the data is transferred from a publisher to a matching subscriber is by default shared memory on the same machine and udp multicast if they are running on different hosts,

So if you publish in protobuf format then

- the message will be serialized into a binary representation by the protobuf publisher

- a binary publisher will check if there is a local or network connection to a matching subscriber

- the data transfer starts by shared memory and / or udp multicast to one or more matching subscriber

- a matching binary subscriber will receive the binary payload

- the payload is serialized back into a protobuf message object

- the message object is forwarded to the protobuf subscriber

As you can see .. on the lower communication levels data will always transferred as binary representation. The kind of transport is independent from the chosen message protocol.

ReX

![eCAL-Architecture](https://user-images.githubusercontent.com/49162693/97865961-543c0f00-1d0b-11eb-96ef-89b8d4881ce0.png "")
