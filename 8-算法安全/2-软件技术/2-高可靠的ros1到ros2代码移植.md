# ROS1代码到ROS2的移植

## 参考

[https://www.apex.ai/post/porting-algorithms-from-ros-1-to-ros-2](https://www.apex.ai/post/porting-algorithms-from-ros-1-to-ros-2)

https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-struct

## 概述

![](https://tcs.teambition.net/storage/3126bb966fa942ec1ad2d1636554b1f544d7?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA0NjkyMiwiaWF0IjoxNjcxNDQyMTIyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZiYjk2NmZhOTQyZWMxYWQyZDE2MzY1NTRiMWY1NDRkNyJ9.keHpDCvWz1SRJuLQXXow_-Z_EeQGUZza-FLC-V3j0D8&download=image.png "")

从ros1到ros2的代码移植也许很简单，就是切换一下库加改一些头文件类型。但是要成功将一个应用切换到ROS2，并且成为产品级代码，需要做以下工作

1-确保算法实现遵循软件架构和软件工程的最佳实现

2-确保关注点分离

3-遵循safety-critical编码规范

4-使用warnning和静态检查器（编译器的warnning要去掉）

5-确保算法经过完整测试

6-针对目标平台进行优化

本文以velodyne driver的移植为例解释上述原则

## 简介

由于ROS2继承了ROS1的大部分概念，所以ROS1的代码可以轻松移植到ROS2，修改构建工程，链接到库，修改下头文件，这样应用程序就运行在ROS2的节点上了。

但是不能称之为ROS2程序，因为不是实时安全的，也不是鲁棒的。需要做好以下事情

## 了解驱动的行为

移植程序之前必须了解应用程序的高层功能

分析驱动的高层功能

Velodyne雷达驱动的行为：

启动两个nodelets，线程

第一个线程

- 轮询socket接口，收到数据包

-当数据包个数足够，将数据包发送到第二个线程

第二个线程

- 收到一些列包

- 解析包中信息，发布点云话题

## 改进-多线程编程

到底是标准库的线程，还是ROS2中的节点呢？

架构上来讲，当前V的驱动有两个工作线程（当然ROS1自动启动了很多线程），首先要确认线程工作机制

![](https://tcs.teambition.net/storage/312648bc049cd6ab9784100e05ea8de188ab?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA0NjkyMiwiaWF0IjoxNjcxNDQyMTIyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY0OGJjMDQ5Y2Q2YWI5Nzg0MTAwZTA1ZWE4ZGUxODhhYiJ9.iATnh9A7Q2WeAquCFWEJbrTAPXh3L1Cl4Y4HJq3tvzA&download=image.png "")

多线程编程的最佳实践是，不要用多线程

根据数据包的属性，如果下一个包已经到来，而前一个包还没有处理完，则这个包可以丢弃了。所以整个应用程序可以重构为单线程应用程序

## 改进-使用事件而不是轮询

为了防止阻塞，ROS1的驱动使用poll，而poll一般来讲会增加系统抖动

为了避免抖动，使用select接口，使用事件机制，降低CPU占用率

### 改进-关注点分离

![](https://tcs.teambition.net/storage/312667451a8fe85fb876a02e4fdff064ccbe?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA0NjkyMiwiaWF0IjoxNjcxNDQyMTIyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY2NzQ1MWE4ZmU4NWZiODc2YTAyZTRmZGZmMDY0Y2NiZSJ9.JOhulla7l3PAgRaH2FtAx8t6oSSDFsYSgfeVhOsoJv4&download=image.png "")

ROS1和ROS2都把2-5涵盖了，不过第一个是需要开发者 考虑的，比如算法，逻辑。

由于ROS1的快速迭代，关注点分离比较难做到。V的驱动就没有做到

## V的ROS1驱动的缺点

由于采用了nodelet，composition这个关注点就要移植

由于采用了pub和sub，Communication这个关注点就要移植

由于采用了ROS1参数，Configuration这个关注点就要移植

Computation和业务强绑定，如果算法原理变化，则需要重新开发

Coordination

![](https://tcs.teambition.net/storage/3126989b47b577fd7325c5b94103592cba40?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA0NjkyMiwiaWF0IjoxNjcxNDQyMTIyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY5ODliNDdiNTc3ZmQ3MzI1YzViOTQxMDM1OTJjYmE0MCJ9.hKgDVTz-44D75XlrsnpY4yr8_RBb-lux07gKR9FOlCQ&download=image.png "")

由于和ROS1强绑定，需要重新开发

## 动作-重新设计关注点分离

在下面这种类设计下，就可以较好隔离ROS1和ROS2的特点，将Computation关注点解耦出来

需要学一点设计模式的知识

![](https://tcs.teambition.net/storage/3126b113b385fc103063d65f35974b479b52?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA0NjkyMiwiaWF0IjoxNjcxNDQyMTIyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZiMTEzYjM4NWZjMTAzMDYzZDY1ZjM1OTc0YjQ3OWI1MiJ9.hgYLgQw4e1pPIjrbSydP6KIEWzux4oAo1FdfEYsDdhY&download=image.png "")

![](https://tcs.teambition.net/storage/31267333c3f675adbcba0b4c9423586c7591?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA0NjkyMiwiaWF0IjoxNjcxNDQyMTIyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY3MzMzYzNmNjc1YWRiY2JhMGI0Yzk0MjM1ODZjNzU5MSJ9.8tTsLifuVzYzRZSG_o8mxJb8Prg2booXYZKZlgmU4g0&download=image.png "")

## 动作-遵循严格的编码规范

![](https://tcs.teambition.net/storage/3126ffa146af72680502068b05c611a772a0?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA0NjkyMiwiaWF0IjoxNjcxNDQyMTIyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZmZmExNDZhZjcyNjgwNTAyMDY4YjA1YzYxMWE3NzJhMCJ9.KvVIbcJs-CP39_96i15RmWxvpl-q3op170cr2ghe9Os&download=image.png "")

- 避免阻塞式调用

- 在循环内避免动态内存分配

- 避免未定义行为

- 避免隐式转换

- 检查返回值

- 适当捕获异常

### 避免动态内存分配

这样就避免了内存泄露

在对象初始化时候进行内存分配

这就要求点云数据的最大尺寸是可预见的。

ros-foxy-osrf-testing-tools-cpp可以帮助测试该问题

### 避免阻塞式调用

阻塞式调用使得执行时间变得不确定

还有其他措施，比如有限的循环次数，避免递归

ROS1的驱动使用poll轮询来避免阻塞，但是产生了CPU抖动的问题。

ROS2的驱动使用udpdriver通过select来获取数据，通过事件的方式，允许超时，在运行事件上是有上限的。

【原则】必须有执行时间上限，超过一定时间则必须结束。必须通过事件方式执行。

### 异常和错误处理

【原则】必须报错

ROS1驱动poll不报错

ROS2驱动超时会抛出异常

## 质量保证

传统的软件工程需要包括测试，代码检查，文档。

遵循cpp开发标准

https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-struct

### 静态检查

turn on all warnings，打开所有警告

使用高级的检查器，如cppcheck等，cpplint

### 测试

单元测试，集成测试

硬件在环HIL，软件在环测试SIL

在环测试

### 检查器

sanitizer

valgrind memcheck

### 文档

## 优化和实时性支持

### 优化

优化的第一步一定是算法形式的优化，包括选择正确的算法，选择正确的数据结构，算法逻辑（排序还是动态规划）。

上述优化统称算法优化。

算法优化经过证实后，当输入数据比较固定，更加小细粒度的优化可以开始

#### 使用性能测试工具

使用perf或者valgrind's callgrind，intel's VTune，包括GPU的perf工具，

#### 细粒度性能优化

将sin, cos替换为查找表，减少计算时间

### 实时性支持

当应用软件经过优化之后，就可以调教为实时支持

应该允许

- 设置CPU亲和性

- 设置进程优先级

- 锁定内存

这些是在最后部署节点，针对特定硬件做的最后一个步骤

## 结论

构建安全的软件需要大量的软件工程实践

![](https://tcs.teambition.net/storage/31267f3e9a927471d67ecca805174312934a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA0NjkyMiwiaWF0IjoxNjcxNDQyMTIyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY3ZjNlOWE5Mjc0NzFkNjdlY2NhODA1MTc0MzEyOTM0YSJ9.T6wm-is6go2Nx1pHkCk-p9wkorPUD7V_vaF4RmME_mQ&download=image.png "")

