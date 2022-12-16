# managed nodes

![](https://tcs.teambition.net/storage/312i76f49bc48b35ba4989d706aeb08b432b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTU4MSwiaWF0IjoxNjcxMTg2NzgxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmk3NmY0OWJjNDhiMzViYTQ5ODlkNzA2YWViMDhiNDMyYiJ9.kJUTRSfCZaATXZa-SJKLoE6V9_OzSCdoDjOAq6JMq44&download=image.png "")

更加灵活

# 参考

[http://design.ros2.org/articles/node_lifecycle.html](http://design.ros2.org/articles/node_lifecycle.html)

[https://github.com/ros2/demos/blob/master/lifecycle/README.rst](https://github.com/ros2/demos/blob/master/lifecycle/README.rst)

[https://github.com/ros2/launch_ros/blob/master/launch_ros/examples/lifecycle_pub_sub_launch.py](https://github.com/ros2/launch_ros/blob/master/launch_ros/examples/lifecycle_pub_sub_launch.py)

<https://index.ros.org/r/cascade_lifecycle/#foxy>

[demos/README.rst at foxy · ros2/demos](https://github.com/ros2/demos/blob/foxy/lifecycle/README.rst)

# 概述

ROS2引入了声明周期管理节点的概念，称作LifecycleNode。主要是借鉴操作系统中进程的状态的思想。ROS离操作系统越来越进了。

本文介绍声明周期节点的目的，与普通节点的区别。管理节点有一系列的状态。

![](https://tcs.teambition.net/storage/31260860b7bf2556fff8dc33a3f2487ebb79?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTU4MSwiaWF0IjoxNjcxMTg2NzgxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjYwODYwYjdiZjI1NTZmZmY4ZGMzM2EzZjI0ODdlYmI3OSJ9.gtY84mC77UX20jcgWtJ1BFYrnGteGmStgoR8OirG15I&download=image.png "")

我们实现了两个种类的状态，Primary States and Transition States，基本状态和迁移状态。基础状态下节点稳定，执行重复的任务。迁移状态下意味着是一个暂时的状态，暂存状态的结果用来表示在两个主状态之间迁移的成功与否。所以，任何一个节点都处于以下状态之一：

![](https://tcs.teambition.net/storage/31269e22a923adf6de68b09f8e4056338aeb?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTU4MSwiaWF0IjoxNjcxMTg2NzgxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY5ZTIyYTkyM2FkZjZkZTY4YjA5ZjhlNDA1NjMzOGFlYiJ9.lO5EagpMhmezeVSz8VaIRxoCTWCi74TxmybrDXhv1T8&download=image.png "")

# 设计思路

管理节点的设计，是为了更好地控制ROS系统的状态，这样roslaunch就可以确保所有的组件实例化正确，然后开始自己的行为。这也允许节点在线重启或替换！

节点的状态管理，遵循一个有限状态机。

## 基础状态

从基础状态跳出需要外部监管进程触发，或者错误异常触发，触发后进入状态迁移状态

有7个外部触发

![](https://tcs.teambition.net/storage/3126ec8732573fcc94467f0c20ba02c0a8a4?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTU4MSwiaWF0IjoxNjcxMTg2NzgxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZlYzg3MzI1NzNmY2M5NDQ2N2YwYzIwYmEwMmMwYThhNCJ9.dIz-w8w6tc1D_LEa6dFwOnnoBcvL9rlRrCF99pAsAKA&download=image.png "")

## 迁移状态

在迁移状态中会执行逻辑，来判断状态是否迁移成功。成功失败状态会通过生命周期管理接口通知到生命周期管理模块软件。

## 每个状态的行为

### 基础状态：未配置

节点实例化后的第一个状态

### 基础状态：未激活

节点不会运行任何程序，不响应任何配置，主题也不会处理

这个状态存在的意义，就是允许节点重配置（改变配置参数，增加删除消息订阅）。不然的化，节点在运行时刻被重配置，会导致未定义行为。

### 基础状态：激活

节点的主状态，节点处理所有信息，主题，请求

如果有错误，会迁移到错误处理状态

### 基础状态：结束

节点销毁之前的最后一个状态。在这个状态下，节点一定会结束。

这个状态存在的意义，是为了方便调试和检测。在这个状态下，节点的状态是可见的，可以调试。

### 迁移状态：配置

在这个状态中，节点的回调会调用，onConfigure callback，来加载配置和其他设置。

一些在节点启动过程中只执行一次的操作，适合放在配置状态下。比如申请内存，创建主题订阅和发布（由于只能执行一次，这个是不能变更的）。

在这个状态中配置的所有资源在整个生命周期都有效！

### 迁移状态：清理

onCleanup，清理所有状态，返回到第一次创建时的状态。

### 迁移状态：激活

onActive，这个方法执行任何执行前的最后准备。

也许包括申请资源，但是这个资源只在激活状态有效！比如访问硬件，在这个状态的操作不适合占用很长时间。

### 迁移状态：去激活

onDeactivate，激活的反操作

### 迁移状态：关闭

onShutdown，清理和析构操作

### 迁移状态：错误处理

处理错误的状态，错误处理完毕后进入未配置状态。

### 迁移状态：销毁

节点释放内存。在面向对象中，调用析构函数。这个过程永远返回成功。

# 管理接口

每一个管理节点都对外暴露一些管理接口，供外部工具调用，这个工具就是roslaunch，这些接口应该跟状态无关。

节点对位提供的服务，应该支持本地管理和远程管理。本地管理通过函数调用实现，远程管理通过topic/services实现。

除了create迁移命令外，所有的迁移命令都通过service服务调用。

# 状态迁移事件

状态迁移的时候，一个topic会发送

The topic must be named lifecycle_state it will carry both the end state and the transition, with result code. It will publish ever time that a transition is triggered, whether successful or not

# 节点管理和状态扩展插件

略

# cascade lifecycle

引入了节点状态的依赖，当A节点进入某状态，其依赖的节点B也自动进入某状态

[ROS Index](https://index.ros.org/r/cascade_lifecycle/)

# demo

略
