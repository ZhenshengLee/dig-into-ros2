executor原理

ros2的本质是executor

注意：node是模块，executor是线程，node是代码，executor是资源，两个node不一定是两个进程或者线程。

# executor的概念

Coordinate the order and timing of available communication tasks.

Executor provides spin functions (including spin_node_once and spin_some). It coordinates the nodes and callback groups by looking for available work and completing it, based on the threading or concurrency scheme provided by the subclass implementation. An example of available work is executing a subscription callback, or a timer callback. The executor structure allows for a decoupling of the communication graph and the execution model. See SingleThreadedExecutor and MultiThreadedExecutor for examples of execution paradigms.

执行器的作用是协调通信任务之间的时序，协调节点和回调任务的执行。

在ROS1中，一个单独的网络线程负责接受消息，放入一个FIFO队列中，没有执行管理。

在ROS2中集成了DDS，消息缓存在DDS中。在ROS2中，有一个Executor的概念引入来管理执行。

**ROS允许多个节点在一个系统进程内运行。为了协调进程内多个节点回调的执行顺序，executor概念引入。**

![](https://tcs.teambition.net/storage/312708bba10e1a7e2c931f5091b5d95f4d8a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcwOGJiYTEwZTFhN2UyYzkzMWY1MDkxYjVkOTVmNGQ4YSJ9.SwxgLsZ0WzXUsOgRVDlvteUVNk5mXmPxHNZ98BLj2H4&download=image.png "")

在rcl中，一个wait-set配置好，rcl从dds层获取一个Handle，这个handle包含定时器，订阅，客户端，服务等。

# 概念模型

![](https://tcs.teambition.net/storage/3127af210819549bb0e806b84d68a29083e8?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdhZjIxMDgxOTU0OWJiMGU4MDZiODRkNjhhMjkwODNlOCJ9.9r3Mk8iEsQTVDcGwzYBjPt1uFjYyYjAeAh1Sg3OySEE&download=image.png "")

本质上是一个事件IO模型

- 事件的来源（框架层面的事件，而不是OS事件）

  - 定时器事件

  - 消息接收

- 除了定时器事件，其他事件是平等的，对事件的处理应该按照事件发生的先后顺序

  - 这个是尽力而为的思想best-effort FIFO

  - 实时的思想是按照优先级排序，确保事件处理不过期

- 与ROS1一致的地方

  - 网络线程将消息传入队列中

  - 回调执行顺序按照注册的顺序，先检查最先注册的回调

  - 对于定时器，放入队列首部

# 标准executor

当前Foxy中有的executor包括，multi_threaded_executor，single_threaded_executor，以及对应的static版本，static版本通过减少初始化的方式减少了cpu消耗和内存占用，这种性能优化在订阅者特别多的时候效果较好。

每一个进程一个executor，可以由main函数启动或者由launch system启动，总的逻辑是，检查可用的工作（timers, services, messages, subscriptions等等），从dds队列中提取，派发到一个或者多个线程，由多个线程执行回调函数。

派发机制与ROS1的spin机制一致：

the Executor looks up the wait sets, which notifies it of any pending callback in the DDS queue. If there are multiple pending callbacks, the ROS 2 Executor executes them in an in the order as they were registered at the Executor.

执行顺序以回调在executor中注册的为准。

顺序：添加节点，节点初始化的注册。

注意：executor::spin实现的是异步IO，这个和ROS1是一致的，而在ROS1中这些工作全部作为一个整体给roscpp做，没有executor的概念

![](https://tcs.teambition.net/storage/31274623d3d596e7461fec9478153a26679a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc0NjIzZDNkNTk2ZTc0NjFmZWM5NDc4MTUzYTI2Njc5YSJ9.0jBQFKVHrOIF3h7ApPQrvY1J0sca1bJtj_21-zCx3HM&download=image.png "")

![](https://tcs.teambition.net/storage/31274d1681e3fb45f1370170b20d6612eba0?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc0ZDE2ODFlM2ZiNDVmMTM3MDE3MGIyMGQ2NjEyZWJhMCJ9.7u8EBomqcE3dJsZwYV8ChPdH-PFmmjOlO_HAO1DXfJY&download=image.png "")

# 静态结构

![](https://tcs.teambition.net/storage/3127de9817a95dfc9336c7b747050a9bcbfa?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdkZTk4MTdhOTVkZmM5MzM2YzdiNzQ3MDUwYTliY2JmYSJ9.kUbXj8m1z3Og4dv3_6-JYEVNWCHflXDoaVui_k3FDPQ&download=image.png "")

默认情况下，executor可以不使用，直接rclcpp::spin()，直接启动默认的executor来执行该节点。

# 动态结构

spin的过程

![](https://tcs.teambition.net/storage/3127d1bf94922028b3c27f9dc79cbdef66b5?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdkMWJmOTQ5MjIwMjhiM2MyN2Y5ZGM3OWNiZGVmNjZiNSJ9.Lpa0DE3t_7bw0zvpCPS02dywAvMRgW81U8j4NM7uvE0&download=image.png "")

executor不维护自己的队列，依赖dds的队列

The Executor concept, however, does not provide means for prioritization or categorization of the incoming callback calls. Moreover, it does not leverage the real-time characteristics of the underlying operating-system scheduler to have finer control on the order of executions. The overall implication of this behavior is that time-critical callbacks could suffer possible deadline misses and a degraded performance since they are serviced later than non-critical callbacks. Additionally, due to the FIFO mechanism, it is difficult to determine usable bounds on the worst-case latency that each callback execution may incur.

缺点：不支持实时特性，不支持顺序自定义，无法保证任务延迟。

![](https://tcs.teambition.net/storage/3127bda1513a8c31fd8d66e52b14e2b67836?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdiZGExNTEzYThjMzFmZDhkNjZlNTJiMTRlMmI2NzgzNiJ9.oQ3kwLSOdzCLtSJmM9HhAoBVmCqZy4efF-VPp_Xu5I4&download=image.png "")

![](https://tcs.teambition.net/storage/3127696c1831b5ba1b52593d8467e5b9d332?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc2OTZjMTgzMWI1YmExYjUyNTkzZDg0NjdlNWI5ZDMzMiJ9.73i1KirofoWdhK2jp3GITWdKz8gVKztZIg9lk8zcVAM&download=image.png "")

- 避免了额外的队列

- 具体的执行顺序交给dds和rcl

  - dds根据qos特性自定义顺序

waitset.wait，等待一些条件，这些条件可以是某对象状态变更，也可以是user trigger。

`rmw_wait` is currently doing two "walks" through all rmw entities:

- It's walking through all entities that were added to rmw wait set to get their corresponding DDS conditions, and attach them to the DDS wait set.

- After waiting, it walks **again** through all rmw entities to generate the output, comparing the conditions of each rmw object with **all** conditions that the DDS wait set marked as ready.

![](https://tcs.teambition.net/storage/3127c899e2f050f87c70be77f52dd6c01aa7?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdjODk5ZTJmMDUwZjg3YzcwYmU3N2Y1MmRkNmMwMWFhNyJ9.ha803uICzdy0akdQE9tPT2o-pvWrjL1gpAJIqiP7wys&download=image.png "")

# mt特性

spin的机制继承自ros1

提取，派发到一个或者多个线程，由多个线程执行回调函数。

当回调执行时间长，但是消息频率高时，可能导致线程数量爆炸

多线程Executor，一个wait多个take

![](https://tcs.teambition.net/storage/3126d1cb8e3941aefba94c7b38d189eb4ac8?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZkMWNiOGUzOTQxYWVmYmE5NGM3YjM4ZDE4OWViNGFjOCJ9.U4UyJQJGkIqHW-ovOVqwZj92p0R3O2gIDBcGoZIVaUU&download=image.png "")

而繼承 Executor 的類別更可以提供多執行緒處理 callback 的能力，或是 Mutual Exculsive，限制在多執行緒下單一時間只處理一個 callback。

![](https://tcs.teambition.net/storage/31278c36fea859494e97e05aec151eba5be3?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc4YzM2ZmVhODU5NDk0ZTk3ZTA1YWVjMTUxZWJhNWJlMyJ9.K5p12oFag5Q-ekTmW2sxA2T3-TdOYNHj3MIrr-hxjro&download=image.png "")

# static特性

![](https://tcs.teambition.net/storage/31275186568188d1957062f4150fd0bcce2a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4Njc5NCwiaWF0IjoxNjcxMTgxOTk0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc1MTg2NTY4MTg4ZDE5NTcwNjJmNDE1MGZkMGJjY2UyYSJ9.UGCkqD7qldc6hNX700QWQdr9Pw7YMj3LnmZVcvIptcA&download=image.png "")

# 总结：标准与实时

标准的策略是：BEST-EFFORT

标准策略是，按先后顺序，尽量尽快执行，没有优先级，不能抢占，且采用轮盘赌，机会均等

实时策略是：RT-CRITICAL

按一定规范定义执行顺序，保证执行的确定性

# 参考

[https://hackmd.io/@1IzBzEXXRsmj6-nLXZ9opw/r1zrNKBWU/%2F%401IzBzEXXRsmj6-nLXZ9opw%2Fryteg9D2B#Executor](https://hackmd.io/@1IzBzEXXRsmj6-nLXZ9opw/r1zrNKBWU/%2F%401IzBzEXXRsmj6-nLXZ9opw%2Fryteg9D2B#Executor)
