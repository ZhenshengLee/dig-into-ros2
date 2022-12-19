# 背景

这是一个原型，用于验证rclc-executor的可行性

可以对每个回调进行优先级，实时性设置。

# 特性

rclcpp executor是在node-level级别工作，不能对所有的callback进行细粒度控制

cbg可以在回调组的细粒度工作，并且应用了一些实时特性，比如RT-CRITICAL and BEST-EFFORT。

在一个节点中定义多个回调组，每一个回调组有一个实时特性，每个回调组分配一个executor。

Thus, an Executor instance can be dedicated to specific callback group(s) and the Executor’s thread(s) can be prioritized according to the real-time requirements of these groups. For example, all time-critical callbacks are handled by an “RT-CRITICAL” Executor instance running at the highest scheduler priority.

The following figure illustrates this approach with two nodes served by three Callback-group-level Executors in one process:

![](https://tcs.teambition.net/storage/3129532e1a877d25be40a004b07dff286f13?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMzIwMywiaWF0IjoxNjcxNDE4NDAzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk1MzJlMWE4NzdkMjViZTQwYTAwNGIwN2RmZjI4NmYxMyJ9.tNQ0hjv47mVPTddkJHyYSIVvC_fWa-C-26a8wmTPYU8&download=image.png "")

The different callbacks of the Drive-Base node are distributed to different Executors (visualized by the color red, yellow and green). For example the onCmdVel and publishWheelTicks callback are scheduled by the same Executor (yellow). Callbacks from different nodes can be serviced by the same Executor.

# API

In this section, we describe the necessary changes to the Executor API:

- [include/rclcpp/callback_group.hpp](https://github.com/boschresearch/ros2_rclcpp/tree/cbg-executor-0.5.1/rclcpp/include/rclcpp/callback_group.hpp):

  - Introduced an enum to distinguish up to three real-time classes (requirements) per node (RealTimeCritical, SoftRealTime, BestEffort)

  - Changed association with Executor instance from nodes to callback groups.

- [include/rclcpp/executor.hpp](https://github.com/boschresearch/ros2_rclcpp/tree/cbg-executor-0.5.1/rclcpp/include/rclcpp/executor.hpp)

  - Added functions to add and remove individual callback groups in addition to whole nodes.

  - Replaced private vector of nodes with a map from callback groups to nodes.

- [include/rclcpp/memory_strategy.hpp](https://github.com/boschresearch/ros2_rclcpp/tree/cbg-executor-0.5.1/rclcpp/include/rclcpp/memory_strategy.hpp)

  - Changed all functions that expect a vector of nodes to the just mentioned map.

- [include/rclcpp/node.hpp](https://github.com/boschresearch/ros2_rclcpp/tree/cbg-executor-0.5.1/rclcpp/include/rclcpp/node.hpp) and [include/rclcpp/node_interfaces/node_base.hpp](https://github.com/boschresearch/ros2_rclcpp/tree/cbg-executor-0.5.1/rclcpp/include/rclcpp/node_interfaces/node_base.hpp)

  - Extended arguments of create_callback_group function for the real-time class.

  - Removed the get_associated_with_executor_atomic function.

The callback-group-level executor has been merged into ROS 2 rclcpp in [pull request 1218](https://github.com/ros2/rclcpp/pull/1218/commits).

# Test

As a proof of concept, we implemented a small test bench in the present package cbg-executor_ping-pong_cpp. The test bench comprises a Ping node and a Pong node which exchange real-time and best-effort messages simultaneously with each other. Each class of messages is handled with a dedicated Executor, as illustrated in the following figure.

![](https://tcs.teambition.net/storage/3129617b82070733444da20e3a98e313f28e?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMzIwMywiaWF0IjoxNjcxNDE4NDAzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk2MTdiODIwNzA3MzM0NDRkYTIwZTNhOThlMzEzZjI4ZSJ9.eaCRiGMua783--UDiG3mFk2B0XthE--uRW_3eAUZTMk&download=image.png "")

![](https://tcs.teambition.net/storage/312925d26b52dcc4a60b1e4965c09bdbf100?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMzIwMywiaWF0IjoxNjcxNDE4NDAzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjkyNWQyNmI1MmRjYzRhNjBiMWU0OTY1YzA5YmRiZjEwMCJ9.spNeXswjp60wJXkTX7QRxdWOIjIoC1TkRnPtl-RAEzU&download=image.png "")

In this example, the callback for the high priority task (red line) consumes 10ms and the low priority task (blue line) 40ms in the Pong Node. With a ping rate of 20 Hz, the CPU saturates (10ms*20+40ms*20=1000ms). With higher frequencies the high priorty task can continue to send its pong message, while the low priority pong task degrades. With a frequency of 100Hz the high priority task requires 100% CPU utilization. With higher ping rates it keeps sending pong messages with 100Hz, while the low priority task does not get any CPU ressources any more and cannot send any messages.

The test bench is provided in the

[https://github.com/ros2/examples/tree/master/rclcpp/executors/cbg_executor](https://github.com/ros2/examples/tree/master/rclcpp/executors/cbg_executor)

# API细节

Callback Groups have a type, either 'Mutually Exclusive' or 'Reentrant' and when creating one the type must be specified.

Callbacks in Reentrant Callback Groups must be able to:

- run at the same time as themselves (reentrant)

- run at the same time as other callbacks in their group

- run at the same time as other callbacks in other groups

Callbacks in Mutually Exclusive Callback Groups:

- will not be run multiple times simultaneously (non-reentrant)

- will not be run at the same time as other callbacks in their group

- but must run at the same time as callbacks in other groups

Additionally, callback groups have a property which determines whether or not they are added to an executor with their associated node automatically. When creating a callback group the automatically_add_to_executor_with_node argument determines this behavior, and if true it will cause the newly created callback group to be added to an executor with the node when the [Executor::add_node](https://docs.ros2.org/galactic/api/rclcpp/classrclcpp_1_1Executor.html#a53f9649e4c2f3593c930f4d191e7e208 "Add a node to the executor.") method is used. If false, this callback group will not be added automatically and would have to be added to an executor manually using the [Executor::add_callback_group](https://docs.ros2.org/galactic/api/rclcpp/classrclcpp_1_1Executor.html#a6bb97eed174e54946d42ea83cf72a5a9 "Add a callback group to an executor.") method.

Whether the node was added to the executor before creating the callback group, or after, is irrelevant; the callback group will be automatically added to the executor in either case.

# 参考

[Execution Management](https://micro.ros.org/docs/concepts/client_library/execution_management/#NSP2018)
