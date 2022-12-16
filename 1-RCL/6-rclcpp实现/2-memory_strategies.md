自定义executor使用的分配内存器

# RCL与RCLCPP

RCL包含三个部分：

- RCL接口聚焦ROS到DDS的概念映射

- 不含线程模型，主要是创建数据结构

- 维护对DDS的操作

RCLCPP包含三个部分：

- 语言客户端，cpp实现

- executor线程模型

- 基于RCL创建更复杂的组件，如类型系统，串行化，component，qos

- 基于RCL和类型系统，QOS生成更复杂灵活的应用程序

上述说明，其实rclcpp的核心是线程模型，当然也为其他的机制提供灵活性，比如generic publisher就可以直接发送串行化的信息

# 代码

![](https://tcs.teambition.net/storage/312g4b3e86bbc59bc8a2586357f21af85afb?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ5OCwiaWF0IjoxNjcxMTg2Njk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmc0YjNlODZiYmM1OWJjOGEyNTg2MzU3ZjIxYWY4NWFmYiJ9.K246F_VAc_zHXh5m3TfISqJ7SiO5rLVKIkZxUx9QgOE&download=image.png "")

This application

model, seen in Figure 1.1, organizes executable code into callbacks. Callbacks are then

organised into nodes (ostensibly object-oriented programming (OOP) classes) which share

access to common resources. Finally, nodes are organised into executable entities, forming a

set of independent executables that (when executed concurrently) represents the complete

application run-time.

- 数据缓存在dds参与者的缓存区中

- **wait的时候找dds要wait信号**

- **wait成功后找node要回调函数**

- exe回调的时候找dds要serialized 数据

# 与DDS关系

![](https://tcs.teambition.net/storage/312geb9a2e2136951a8a15202c9041db58e2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ5OCwiaWF0IjoxNjcxMTg2Njk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdlYjlhMmUyMTM2OTUxYThhMTUyMDJjOTA0MWRiNThlMiJ9.sftaEQ4S2RD2Q5U0jAP_UOXH8xH56p3qY110eq-9_EY&download=image.png "")

# memory_strategy

Delegate for handling memory allocations while the [Executor](https://docs.ros2.org/galactic/api/rclcpp/classrclcpp_1_1Executor.html "Coordinate the order and timing of available communication tasks.") is executing.

By default, the memory strategy dynamically allocates memory for structures that come in from the rmw implementation after the executor waits for work, based on the number of entities that come through.

作为executor委托的对象，用于

- **wait的时候找dds要wait信号**

- **wait成功后找node要回调函数**

![](https://tcs.teambition.net/storage/312g01229951344505b83bc278d1b5f6c5d2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ5OCwiaWF0IjoxNjcxMTg2Njk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmcwMTIyOTk1MTM0NDUwNWI4M2JjMjc4ZDFiNWY2YzVkMiJ9.NrJ_ttTQdf3WIQii0f9hiesxGXqWKmypCrtIz0QY50w&download=image.png "")

# 使用

# 参考

[rclcpp: rclcpp::strategies::message_pool_memory_strategy::MessagePoolMemoryStrategy< MessageT, Size, > Class Template Reference](https://docs.ros2.org/galactic/api/rclcpp/classrclcpp_1_1strategies_1_1message__pool__memory__strategy_1_1MessagePoolMemoryStrategy.html)

[https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/strategies/allocator_memory_strategy.hpp](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/strategies/allocator_memory_strategy.hpp)

[demos/allocator_tutorial.cpp at galactic · ros2/demos](https://github.com/ros2/demos/blob/galactic/demo_nodes_cpp/src/topics/allocator_tutorial.cpp)
