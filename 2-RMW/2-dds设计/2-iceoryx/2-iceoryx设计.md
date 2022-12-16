## 参考

[Eclipse iceoryx | The Eclipse Foundation](https://www.eclipse.org/community/eclipse_newsletter/2019/december/4.php)

## 概述

在汽车领域，对于IPC，有大数据量传输的需求，iceoryx就是使得零拷贝的中间件，由博世研发，用在汽车领域。

![](https://tcs.teambition.net/storage/312750a0684d73ef1c8cd7071eb6eb044939?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODAzNiwiaWF0IjoxNjcxMTkzMjM2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc1MGEwNjg0ZDczZWYxYzhjZDcwNzFlYjZlYjA0NDkzOSJ9.7g6rgh__iUBAVkHEoM-BwgIkY240EXeswCZOPtvRVPA&download=image.png "")

一个激光雷达，10hz，1M点云，那么就是10M/s的速率。

整体数据传输速率很可能达到1G/s，蔚来汽车甚至声称10G/s（出处不明）

相似的领域有机器人领域和IoT领域，数据传输的方式为订阅和发布。

典型情况下，middleware需要对每个订阅者做拷贝。

对于大数据，拷贝线性增加时延，而且增加CPU占用率。这样使得资源没有用在真正的业务上。

![](https://tcs.teambition.net/storage/31274977bce897687f41c0b88474372a2926?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODAzNiwiaWF0IjoxNjcxMTkzMjM2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc0OTc3YmNlODk3Njg3ZjQxYzBiODg0NzQzNzJhMjkyNiJ9.gu2TEPqsBQxL3HSh4wThX6EK5ju8L_o5qsgLTLMBmNE&download=image.png "")

## iceoryx的零拷贝

零拷贝的概念在1970s就有了，iceoryx在此基础上加上了publish/subscribe architecture, service discovery, modern C++, and lock-free algorithms.

真正零拷贝的意思是：发布者和订阅者之间传输数据不产生数据拷贝

在iceoryx中，发布者将数据直接写在内存池中，订阅者接受内存块的引用，维护一个队列。

每一个订阅者对消息的视图都是独立的，iceoryx计算引用计数，在计数为0时候释放内存

![](https://tcs.teambition.net/storage/31271c1a9f6e9b1d0e2ec0ea02a03a8a8111?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODAzNiwiaWF0IjoxNjcxMTkzMjM2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcxYzFhOWY2ZTliMWQwZTJlYzBlYTAyYTAzYThhODExMSJ9.YBcBTr7JGRPkcl7wwqP1c5afnqP0DhJuogF2uvPHnMI&download=image.png "")

iceoryx支持pooling轮询和callback事件方式。

内存池可以做访问控制。

## 技术细节

订阅者读消息时候，发布者可以再次写入。

如果订阅者在polling，middle会回收旧的内存块。

因为传输的是指针，所以传输数据的事件是恒定的。用户需要写入一次。

因为消息没有串化，所以消息发送和接受必须具有相同的内存布局。所以进程必须要使用相同的编译器，使用相同的编译选项。

消息中不能包含指向另一块内存的指针。

如果这些限制不能满足，iceoryx也能够在顶层处理串行化和解串行化，这一种情况下iceoryx处理低层次的零拷贝流程。

底层接口不建议用户直接使用，而是通过ROS使用。

已经和cyclonedds结合，可以构造一个同时实现网络通信和本地通信的中间件。

## rmw_iceoryx实现原理

The code above has one problem though: How can the middleware allocate enough memory for the string message? The middleware can't possibly know the size of the string the user wants to put into that message.

That being said, in order to enable a true zero copy data transport we have to limit ourselves to fixed size data structures. The picture below tries to illustrate the difference between a fixed size message and a dynamically resizable message.

![](https://github.com/ros2/rmw_iceoryx/raw/foxy/docs/fixed_size_messages.png "")

固定大小数据的处理

不固定大小数据的处理

就算串行化，串行化之后也是固定大小的数据，可以做到零拷贝

对于大型点云数据，如果不是固定大小数据，那么开销会多一个串行化和解串行

```cpp
 // if messages have a fixed size, we can just memcpy
  if (iceoryx_publisher->is_fixed_size_) {
    return details::send_payload(iceoryx_sender, ros_message, iceoryx_publisher->message_size_);
  }

  // this should never happen if checked already at rmw_create_publisher
  if (!rmw_iceoryx_cpp::iceoryx_is_valid_type_support(&iceoryx_publisher->type_supports_)) {
    RMW_SET_ERROR_MSG("Use either C typesupport or CPP typesupport");
    return RMW_RET_ERROR;
  }

  // message is neither loaned nor fixed size, so we have to serialize
  std::vector<char> payload_vector{};

  rmw_iceoryx_cpp::serialize(ros_message, &iceoryx_publisher->type_supports_, payload_vector);

  // send composed payload
  return details::send_payload(iceoryx_sender, payload_vector.data(), payload_vector.size());

```

## 消息的改进

消息串行化的基本原理见

对于大型数据的串行化开销

使用autoware.auto消息定义和类型定义

autoware.auto中的消息定义全部是有界的?

也有string和sequence<uint8>

# 线程模型
