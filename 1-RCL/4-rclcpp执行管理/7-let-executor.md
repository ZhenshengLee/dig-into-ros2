logic execution time

博世的解决方案-rclc

# 概述

ros提供了一个基于SOA的软件框架，但是没有提供高效的回调执行控制。例如节点不能控制回调执行的顺序，默认是谁先到谁先执行，基于FIFO。但是机器人业务的感知，验证，执行是有明显顺序要求的，在ros中没有显式定义的方法。而且，当输入数据一定，比如在bag包中，每次冲播放，结果都不一样，这都是因为执行过程的不确定性。

根据主题的优先级来手动设置回调的执行顺序，通过设置Linux进程优先级来实现的，也是可能的，但是这个方法容易导致错误，且扩展性很差，需要对部署在当前系统中的ros2节点深入了解。

所以real-time executor提供一个简单的机制来提供以下特性：

- 回调执行的顺序

- 实时性的支持

- 在一个平台中集成实时的和非实时的特性

- 支持RTOS和microcontrollers

# 标准executor的问题

标准的executor有一些违反实时性的特点：

在网络IO中，收到数据本质上都是事件

- timers事件具有最大的优先级

- 非定时器事件，不具备抢占性的时间片轮转调度

- 对于每一个handle只考虑一个输入数据

这种rclcpp executor不是确定性的，所以不具备强实时性的特性。

- 在dashing中，executor占用很高的CPU占用率

- 没有针对实时性能进行优化

- 没有抢占优先级

- 使用轮转模式，空转，spin带来高cpuz占用率

- 在流量拥塞时没有队列FIFO机制

对延迟有需求

# 设计目标

实时性设计目标：

- 为ros2设计一种调度机制

- 友好使用的api

- 定义关于实时性的需求模型

- 将ROS的概念和OS调度器的概念映射

![](https://tcs.teambition.net/storage/3126754661672c724989369f21af0a0f0508?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA5NiwiaWF0IjoxNjcxMTgyMjk2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY3NTQ2NjE2NzJjNzI0OTg5MzY5ZjIxYWYwYTBmMDUwOCJ9.z7yKaIXUkReAw_TCboJ83kD7lQFB7T6amc7TDE87dU0&download=image.png "")

# 方法

提供一种实时的执行器。基于rcl的和rclcpp的

我们首先提出了基于LET的调度机制，基于logic execution time机制的静态调度策略。在这种策略中，所有的回调函数都在预定义的顺序下执行。

Logical execution time refers to the concept, that first input data is read before tasks are executed.

LET指的是，第一个输入数据在任务执行前读取完毕。

我们还提供了一个call-back-group的executor，以组为单位为回调函数定义优先级，这些方法基于executor的概念，在ros2中引入。

# let executor

定义实时性的调度

# 嵌入式特性

rclc包含了实时性的考虑

![](https://tcs.teambition.net/storage/31263903dc87479ec522426bbeb50f0b4aad?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA5NiwiaWF0IjoxNjcxMTgyMjk2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjYzOTAzZGM4NzQ3OWVjNTIyNDI2YmJlYjUwZjBiNGFhZCJ9.yAWOcI4kVcJTkzRO2O0qRlEmbEqT1n8RpS_9ug9DRAo&download=image.png "")

![](https://tcs.teambition.net/storage/3126378a31cd62936464e97d29cb37667315?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA5NiwiaWF0IjoxNjcxMTgyMjk2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjYzNzhhMzFjZDYyOTM2NDY0ZTk3ZDI5Y2IzNzY2NzMxNSJ9.DvgqFKoBuEcb1JezMRPy9-Vslp7gdxA-8QhcEmhp89w&download=image.png "")

这个属于上述方法的第二种，添加额外的队列和排序方法。

# 参考

[rclc/executor.h at master · ros2/rclc](https://github.com/ros2/rclc/blob/master/rclc/include/rclc/executor.h)
