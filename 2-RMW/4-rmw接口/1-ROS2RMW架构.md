# 参考

[https://design.ros2.org/articles/ros_middleware_interface.html](https://design.ros2.org/articles/ros_middleware_interface.html)

<http://docs.ros.org/en/rolling/Concepts/About-Middleware-Implementations.html>

<https://design.ros2.org/articles/ros_middleware_interface.html>﻿<http://docs.ros.org/>

# RCL接口

RCL robot client library定义了一系列的API，向用户暴露一些通信概念，比如订阅发布。

在ROS1中，是TCPROS来定义的。

在ROS2中，是在DDS的基础上定义的。明显的优势就是，ROS2可以在一个成熟的中间件上开发。

由于有非常多不同的DDS实现，每种实现都有不同的特点，所以需要在中间隔离一层。

这些差异体现在：支持的平台，编程语言，性能特点，内存分布，依赖，以及许可证

每个DDS的平台API都有不同，所以需要隔离一层。将统一的接口映射到具体DDS平台的API

# RMW接口

因为要向用户掩盖内部的复杂性！

![](https://tcs.teambition.net/storage/3126bb6357ee0f46f0976ae336818fd70d83?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODc1NiwiaWF0IjoxNjcxMTkzOTU2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZiYjYzNTdlZTBmNDZmMDk3NmFlMzM2ODE4ZmQ3MGQ4MyJ9.qmO89BfdwPpTXpCwFu302tRQ-vi5nG7d9EUQKbpfDNQ&download=image.png "")

![](https://tcs.teambition.net/storage/312671e573f37b0b621aecaa7f2c2f1923ab?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODc1NiwiaWF0IjoxNjcxMTkzOTU2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY3MWU1NzNmMzdiMGI2MjFhZWNhYTdmMmMyZjE5MjNhYiJ9.Fe6zdNQRiFAN7NxXkFVMxbzgTcDGt-D8_sU1qx8sKsg&download=image.png "")

RCL只能够操作ROS相关的数据结构

middleware interface的接口设计目的是掩盖DDS的接口

`https://github.com/ros2/rmw/tree/master/rmw`规定了rmw的接口，

`https://github.com/ros2/rmw_implementation/tree/master/rmw_implementation`负责转发rclcpp的接口调用，如果用户设置了具体的rmw，就调用对应的，如果没有指定，则调用默认的

这些是中间件提供给RCL的接口，用户太码调用RCL的接口，RCL的接口调用RMW的接口，RMW的接口调用DDS的接口

所有接口实现为C接口。C接口的好处，能够绑定到其他语言，python, java, and cpp。
