APEX.AI商用DDS! 已经用在的自动驾驶汽车上

广州小鹏汽车的Autosare就已经使用上了！

# 版权

针对ROS2提供ubuntu上的apt二进制安装版本

```bash
sudo apt install rti-connext-dds-5.3.1
# 最新版本6.0.1
```

# 组件

## DataFramework

ConnextDDS Core，是DDS核心框架

## ConnextTools

monitoring, visualization and performance analysis:

- shapesDemo: 学习DDS概念的材料

- AdminConsole:

- DDSSpy:

- Monitor: 整个系统负载

- HeapAnalyzer:

- WiresharkPlugin: 抓包分析工具

system design:

- code generator

- system designer: 图形化系统设计和配置

- micro application generator:

## Infrastructure services

Connext Infrastructure Services enables developers to rapidly scale and integrate real-time, distributed systems.

基础设施服务，用于将DDS系统与外部各种类型系统集成起来。

FastDDS有响应的套件，叫做intergration service

![](https://tcs.teambition.net/storage/312j1321769d0da13af59a7057f53c684f73?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk3OSwiaWF0IjoxNjcxMTkwMTc5LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmoxMzIxNzY5ZDBkYTEzYWY1OWE3MDU3ZjUzYzY4NGY3MyJ9.czVRhsd-QDa1GirDGngB5UEUh7e_tO2goMigd5zTWTY&download=image.png "")

# 对ROS2的支持

其宣称对ROS2有很好的支持，而且其Tools也可以直接用上

比eCAL要好，eCAL内置工具需要在protobuf编码下才能够使用tools

# 直接编写应用程序

[__https://www.rti.com/blog/the-indy-autonomous-challenge-achieving-extreme-performance-with-ros-2__](https://www.rti.com/blog/the-indy-autonomous-challenge-achieving-extreme-performance-with-ros-2)

在这篇文章中介绍去掉ROS2的软件层，直接在connextdds基础上构建

但是缺点特别明显，对开源有害

- 失去了ROS1的生态

- 失去了ROS2的工具

- 失去了ROS2的机器人生态

- 失去了多其他DDS的支持

上述对于一个开源软件组件，损失是巨大的，开源精神要求，人人可参与

对应用开发者有害

- 大量的机器人组件需要重新开发，比如rviz，tf，消息定义等

但是仍然有借鉴意义

由于DDS的互操作性，可以将某进程从ROS2进程替换为CDDS进程，

# 高性能

在上篇文章中采用多种方式，除了去掉ROS2层次外，还使用了两个特性

- RTI FlatData，这是一个编码方式，直接按照其在内存中的存储方式来编码，所以避免了很多数据拷贝

- RTI ZeroCopy，数据不拷贝，只传递一个指针

# 性能

![](https://tcs.teambition.net/storage/3126db6952dcf39b7d845022d851afea7b37?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk3OSwiaWF0IjoxNjcxMTkwMTc5LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZkYjY5NTJkY2YzOWI3ZDg0NTAyMmQ4NTFhZmVhN2IzNyJ9.y3Iz6GXV_7uc3sTF3IhHJMdP-9f-Sq4hbyuj0fVEIfw&download=image.png "")

# 参考

<https://www.rti.com/blog/latest-connext-dds-ros-2-performance-benchmarks>

<https://www.rti.com/blog/ros2-dds-play-it-again>

<https://www.rti.com/blog/ros-2-with-dds-the-right-framework-to-choose-when-building-an-autonomous-vehicle>

[__https://www.rti.com/blog/ros2-dds-when-ecosystems-merge__](https://www.rti.com/blog/ros2-dds-when-ecosystems-merge)

[__https://www.infoq.com/articles/ros2-dds-communication/__](https://www.infoq.com/articles/ros2-dds-communication/)
