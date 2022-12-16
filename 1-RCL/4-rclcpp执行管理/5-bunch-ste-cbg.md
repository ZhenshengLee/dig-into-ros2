**add_callback_group api, 由executor调用，将不同的cbg添加到executor中**

*Callback-group-level Executor* concept

这个特性没有增加新的executor，而是利用cbg来细化executor的api.

这个行为允许单个节点拥有多个回调，不同回调间的RT特性不同，并且驻留到不同的executor实例(线程)上运行，在单进程下.

这样的好处在于，一个节点的多个回调可以分开在不同的executor中执行，而executor的实时特性是可以配置的，则不同的cbg的实时特性也是可以配置的。比如关键回调可以由高RT配置的executor实例执行，配置executor的RT，使用装载它的thread的优先级配置就可以了。

```cpp
rclcpp::SubscriptionOptionsWithAllocator<std::allocator<void>> options;
  options.callback_group = low_prio_callback_group_;
  low_ping_subscription_ = this->create_subscription<Int32>(
    "low_ping", rclcpp::SensorDataQoS(),
    std::bind(&PongNode::low_ping_received, this, _1), options);

//////

// Create two executors within this process.
  rclcpp::executors::SingleThreadedExecutor high_prio_executor;
  rclcpp::executors::SingleThreadedExecutor low_prio_executor;

  // Create Ping node instance and add it to high-prio executor.
  auto ping_node = std::make_shared<PingNode>();
  high_prio_executor.add_node(ping_node);

  // Create Pong node instance and add it the one of its callback groups
  // to the high-prio executor and the other to the low-prio executor.
  auto pong_node = std::make_shared<PongNode>();
  high_prio_executor.add_callback_group(
    pong_node->get_high_prio_callback_group(), pong_node->get_node_base_interface());
  low_prio_executor.add_callback_group(
    pong_node->get_low_prio_callback_group(), pong_node->get_node_base_interface());

  rclcpp::Logger logger = pong_node->get_logger();

  // Create a thread for each of the two executors ...
  auto high_prio_thread = std::thread(
    [&]() {
      high_prio_executor.spin();
    });
  auto low_prio_thread = std::thread(
    [&]() {
      low_prio_executor.spin();
    });

```

# cbg 特性

该特性已经合并到foxy主分支

cbg的目的是为了区分不同回调的实时特性要求。

使得不同类型的回调按类别分配到指定线程中执行。

cb有两个类型：

- MutuallyExclusive

- Reentrant

![](https://tcs.teambition.net/storage/3126820f71c701cd8004649e1de560452f7b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA1OSwiaWF0IjoxNjcxMTgyMjU5LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY4MjBmNzFjNzAxY2Q4MDA0NjQ5ZTFkZTU2MDQ1MmY3YiJ9.dhhn2VN9xjugj48xwsdz12JVJiC_JBr75Xyef-DWC9o&download=image.png "")

Thus, an Executor instance can be dedicated to specific callback group(s) and the Executor’s thread(s) can be prioritized according to the real-time requirements of these groups. For example, all time-critical callbacks are handled by an “RT-CRITICAL” Executor instance running at the highest scheduler priority.

The following figure illustrates this approach with two nodes served by three Callback-group-level Executors in one process:

![](https://tcs.teambition.net/storage/312757725f21cba0adcff40bf04f5cd970cb?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA1OSwiaWF0IjoxNjcxMTgyMjU5LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc1NzcyNWYyMWNiYTBhZGNmZjQwYmYwNGY1Y2Q5NzBjYiJ9.5qAtHj4nBOkV2syiCL-h40AN3w6e5EIu0ksS-FFbIaY&download=image.png "")

The callback-group-level executor has been merged into ROS 2 rclcpp in [pull request 1218](https://github.com/ros2/rclcpp/pull/1218/commits).

# 参考

[ros2/examples](https://github.com/ros2/examples/tree/master/rclcpp/executors/cbg_executor)

[boschresearch/ros2_rclcpp](https://github.com/boschresearch/ros2_rclcpp/blob/cbg-executor-foxy/README.md)

[ROS Index](https://index.ros.org/p/rclc/)

# 使用

```bash
sudo apt install ros-foxy-rclc ros-foxy-rclc-lifecycle ros-foxy-rclc-examples
```
