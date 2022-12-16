cyclonedds内部就是使用iceoryx

# cyclonedds-shm

目前不支持，在ice2.0中支持

# cyclonedds-zc

## cyclonedds对iceoryx的吸收引进

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<CycloneDDS xmlns="https://cdds.io/config" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="https://cdds.io/config https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/iceoryx/etc/cyclonedds.xsd">
    <Domain id="any">
        <SharedMemory>
            <Enable>true</Enable>
            <SubQueueCapacity>256</SubQueueCapacity>
            <SubHistoryRequest>16</SubHistoryRequest>
            <PubHistoryCapacity>16</PubHistoryCapacity>
            <LogLevel>info</LogLevel>
        </SharedMemory>
    </Domain>
</CycloneDDS>

```

shm zero_copy细节

cyclonedds使用iceoryx的场景下，每一个sub都使用内部queen队列，根据qos设置，shm可能不会使用。具体限制如下

1. *SubQueueCapacity* defines the maximum number of samples a subscription can locally store in Shared Memory. If more samples arrive via Shared Memory, the least recent ones will be discarded.

1. *SubHistoryRequest* is the maximum number of samples a late joining subscription will request from a corresponding publisher.

1. *PubHistoryCapacity* is the maximum number of samples a publisher will keep for late joining subscriptions.

1. *Loglevel* controls the output of the iceoryx runtime and can be set to verbose, debug, info, warn, error, fatal and off in order of decreasing output level.

iceoryx最大满足256订阅队列深度

```cpp
export CYCLONEDDS_URI=file://$PWD/cyclonedds.xml
. ~/ros2_ws/install/local_setup.bash
ros2 run demo_nodes_cpp talker
```

在当前情况下，没有使用SHM，因为消息类型中包含string，是一个动态大小的数据类型。

尽管如此，还是需要roudi运行，因为配置文件已经配置了roudi。

## 动态数据结构支持

在iceoryx2.0中增加对动态结构数据支持

但是在galactic中不可用

# rmw支持

目前是支持的

## rclcpp接口

如果要在talker程序中使用shm的话，需要重新设计固定大小的数据类型，比如`std_msgs::msg::Uint32`

```cpp
msg_ = std::make_unique<std_msgs::msg::Uint32>();
    msg_->data = count_++;
    RCLCPP_INFO(this->get_logger(), "Publishing: 'Hello World: %lu'", msg_->data);
    pub_->publish(std::move(msg_));

auto callback =
      [this](const std::shared_ptr<std_msgs::msg::Uint32> msg) -> void
      {
        RCLCPP_INFO(this->get_logger(), "I heard: [Hello World: %lu]", msg->data);
      };

```

# dds局限性

## 额外限制

在这里零拷贝的意思是没有拷贝或者串行化

1. Shared Memory has to be enabled in the configuration.

1. The subscription has to be local on the same machine (i.e. same or different process, but no remote subscription).

1. The QoS settings admit Shared Memory transfer (cf. [QoS settings](https://github.com/ros2/rmw_cyclonedds/blob/galactic/shared_memory_support.md#QoS-settings))

1. The message type has fixed size (i.e. no strings or sequences of dynamic length)

## API限制：使用loan api

### 默认api

We first create a message, populate it before passing it to the publisher.

```text
    msg_ = std::make_unique<std_msgs::msg::Uint32>();
    msg_->data = count_++;
    pub_->publish(std::move(msg_));
```

If the Shared Memory transfer conditions are met, publish will internally loan a Shared Memory chunk from iceoryx and copy the message payload into it without the serialization required for network transfer. Any connected subscription has read-only access to this message data.

While this API will not allow true zero-copy transfer, it still will improve performance for sufficiently large message sizes since it bypasses the loopback interface and does not perform serialization. The actual size where it will outperform the loopback interface primarily depends on the actual hardware and system load.

### loan api

The loan API allows for true zero-copy transfer by directly consctructing the message data in Shared Memory. To do so, we first have to loan this Shared Memory chunk explicitly.

```text
auto loaned_msg = pub_->borrow_loaned_message();
loaned_msg.get().data = count_++;
pub_->publish(std::move(loaned_msg));
```

The publish overload here does not need to copy the data to Shared Memory since it already resides there. Hence no copy is performed and the data is transferred to any subscription in constant time (i.e. independent of the message size). Depending on the data type there serialization into the loaned message may still be required and publish will therefore not be independent of message size. This is not the case for [fixed size](https://github.com/ros2/rmw_cyclonedds/blob/galactic/shared_memory_support.md#Types) data types.

Note that to properly use loaning move semantics are essential, i.e. using `std::move` when publishing is required. This allows efficiently transferring ownership of the internal message data back to the middleware without copy overhead. As a consequence read access of `loaned_msg` after it was published is illegal (undefined behavior).

## 数据类型限制

The message types used must be of a fixed size, i.e. the size can be determined at compile time. This means strings or variable length arrays of the [available message types](http://wiki.ros.org/msg) cannot be used. Nesting types satisfying the fixed size restriction is also possible.

只有固定大小的数据类型才能够用。

First, the data types to be exchanged need to have a fixed size. This precludes the use of strings and sequences at any level in the data type, though this does not prevent the use of arrays, as their size is fixed at compile time. If any of these types of member variables are encountered in the IDL code generating the data types, shared memory exchange is disabled.

A possible workaround for this limitation is using fixed size arrays of chars in stead of strings, and arrays of other types in stead of sequences, and take any overhead for granted.

autoware.auto就是这样做的，可以直接拿来参考。

## QoS限制

iceoryx不完整支持qos，则默认qos为keepall, queen

Only a subset of QoS settings supports Shared Memory. Those are:

1. Liveliness: Automatic

1. Deadline: Infinity (0)

1. Reliability: Reliable or Best Effort

1. Durability: Volatile

1. History: Keep Last

    1. with history depth no larger than PubHistoryCapacity (in the configuration file)

These settings (with Reliability: Reliable) are used by default and applicable to a wide range of applications.

![](https://tcs.teambition.net/storage/3128b231d605c135f792ea0a5ff41b9f3891?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODQ4OCwiaWF0IjoxNjcxMTkzNjg4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjhiMjMxZDYwNWMxMzVmNzkyZWEwYTVmZjQxYjlmMzg5MSJ9.Lp6NssOCgTklyXTcBnFHioNS4VYz_ez5zYMDB65yPsA&download=image.png "")

# 使用注意事项

## iceoryx内部配置

配置好内存块设置

Note that currently the internal loan call is blocking, which means if no memory is available it will not return (this will change in the future). This may happen if the configured memory is not sufficient for the overall system load, i.e. the memory needed was not available in the first place or is used by other samples which are currently read or written.

iceoryx的内存配置是若干个固定大小的内存块，组成内存池。当订阅者请求内存块时，iceoryx会分配一个最小的内存块，如果没有足够内存块满足要求，则进程崩溃。

默认的配置有足够到4M大小的块，如果配置不满足，则fall back to this configuration。

为了最佳性能，用户需要合适的配置。由于header一般和数据一起发送，所以块大小一般大于64 bytes，而且配置需要4bytes对齐。

Below follows an example of an iceoryx configuration file which has a memory pool of 2^15 blocks which can store data types of 16384 bytes (+ 64 byte header = 16448 byte block):

```text
[general]
version = 1

[[segment]]

[[segment.mempool]]
size = 16448
count = 32768
```

## 如何检查是否真正使用了shm

通过iceoryx introspection工具检查

## ros2性能开销分段

rclcpp开销，executor开销。

rmw开销，包括graph开销，qos，security开销等。

dds开销，这个才是dds供应商宣称的性能。

# 示例程序

[GitHub - ApexAI/ros2_shm_demo: Demonstrate how to use zero-copy Shared Memory data transfer in a single independent example. The middleware used is Eclipse CycloneDDS which integrates Eclipse iceoryx for Shared Memory transfer.](https://github.com/ApexAI/ros2_shm_demo)

# 结论

cyclonedds的zcp使用起来局限性较多，如果不满足严格的条件，则自动走原有的loopback。只有loopback和zero_copy两个选项

做到点云的零拷贝尤其困难，要改动的定义代码特别多。

使用rmw_iceoryx，可以在loopback和zero_copy间找到第三个选项，有一次拷贝的minimum_copy，相比loopback而言，仍然省下多次拷贝。

# 参考

[https://github.com/ros2/rmw_cyclonedds/blob/galactic/shared_memory_support.md#Restrictions](https://github.com/ros2/rmw_cyclonedds/blob/galactic/shared_memory_support.md#Restrictions)

[ros2/rmw_cyclonedds](https://github.com/ros2/rmw_cyclonedds/blob/galactic/shared_memory_support.md)

[https://github.com/eclipse-cyclonedds/cyclonedds/pull/1175](https://github.com/eclipse-cyclonedds/cyclonedds/pull/1175)

[https://github.com/ros2/rmw_cyclonedds/pull/379](https://github.com/ros2/rmw_cyclonedds/pull/379)
