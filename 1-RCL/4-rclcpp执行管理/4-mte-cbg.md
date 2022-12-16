**create_callback_group api，由node调用，创建回调组**

你没有办法控制mte中每个thread的实时性配置

# 概念

当在mt executor中运行节点，ROS2提供cbg作为工具来控制不同回调的执行。

本想使用多线程executor执行node，没想到简单的创建Subscription之后callback funcs竟然是顺序执行的，所以仔细看了一下，原来node中还有callback_group这个东西。

类型有

- MutuallyExclusive；互斥，即这个组别中每时刻只允许1个线程，一个callback在执行时，其他只能等待

- Reentrant；可重入，这个组别中每时刻允许多个线程，一个Callback在执行时，其他callback可开启新的线程

默认情况

- 每个node中有一个默认的callback_group，默认是MutuallyExclusive类型，即便使用了multiThreadedExecutor，也依然默认MutuallyExclusive类型

- 当node创建一个subscription时，可以设置选项

# 基础

两种cbg类型

- Mutually Exclusive Callback Group 互斥cbg

- Reentrant Callback Group 可重入cbg

互斥类型的cbg，不允许回调并行执行，在这个cbg中的回调行为像在ste中一样

可重入cbg中的回调允许进行调度，并行执行，在这种情况下，同一个话题的多个回调执行实例，也会执行，可能会导致算法计算结果**时序的乱序**

不同的cbg之间的的回调总是可以并行调度和执行的

同一个实体创建的回调都在一个cbg内，比如action client创建的所有回调都在同一个cbg内

cbg的创建使用node的接口create_callback_group，The callback group can then be passed as argument/option when creating a subscription, timer,

```cpp
my_callback_group = create_callback_group(rclcpp::CallbackGroupType::MutuallyExclusive);

rclcpp::SubscriptionOptions options;
options.callback_group = my_callback_group;

my_subscription = create_subscription<Int32>("/topic", rclcpp::SensorDataQoS(),
                                              callback, options);
```

# 回调的概念

前摄器模式

In the context of ROS 2 and executors, **a callback means a function whose scheduling and execution is handled by an executor.**Examples of callbacks in this context are

- subscription callbacks (receiving and handling data from a topic),

- timer callbacks,

- service callbacks (for executing service requests in a server),

- different callbacks in action servers and clients,

- done-callbacks of Futures.

Below are a couple important points about callbacks that should be kept in mind when working with callback groups.

- **Almost everything in ROS 2** is a callback! Every function that is run by an executor is, by definition, a callback. The non-callback functions in a ROS 2 system are found mainly at the **edge of the system (user and sensor inputs etc).**

- Sometimes the callbacks are hidden and their presence may not be obvious from the user/developer API. This is the case especially with any kind of “synchronous” call to a service or an action (in rclpy). For example, the synchronous call `Client.call(request)` to a service adds a Future’s done-callback that needs to be executed during the execution of the function call, but this callback is not directly visible to the user.

# cbg作用-合理调度

更好的控制执行

In order to control execution with callback groups, one can consider the following guidelines.

- Register callbacks that should never be executed in parallel to the same Mutually Exclusive Callback Group. An example case might be that the callbacks are accessing shared critical and non-thread-safe resources.

  - 将永远不会并发执行的回调放入互斥cbg中，比如将线程不安全的资源访问放在一个组中

- If you have a callback whose execution instances need to be able to overlap with each other, register it to a Reentrant Callback Group. An example case could be an action server that needs to be able to process several action calls in parallel to each other.

  - 如果回调执行需要覆盖掉之前的那个，那么放在可重入cbg中

- If you have different callbacks that require to be potentially executed in parallel to one another, register them to

  - a Reentrant Callback Group, or

  - different Mutually Exclusive Callback Groups (this option is good if you want the callbacks to not overlap themselves or also need thread safety with respect to some other callbacks) or different callback groups of any type (choose the types according to other criteria).

  - 如果你需要不同的回调必须并发执行，则可以放入同一个可重入的cbg中，或者放入不同的互斥cbg中

  - 不同的cbg中的好处在于，多个实例可以并发运行，并且不会被overlap。

Note that the option in the list is a valid way of allowing parallel execution for different callbacks, and can even be more desirable than simply registering everything into one Reentrant Callback Group.

# cbg作用-避免死锁

Setting up callback groups of a node incorrectly can lead to deadlocks (or other unwanted behavior), especially if one desires to use **synchronous calls to services or actions**. Indeed, even the API documentation of ROS 2 mentions that **synchronous calls to actions or services should not be done in callbacks, because it can lead to deadlocks.** While using asynchronous calls is indeed safer in this regard, synchronous calls can also be made to work. On the other hand, synchronous calls also have their advantages, such as making the code simpler and easier to understand. Hence, this section provides some guidelines on how to set up a node’s callback groups correctly in order to avoid deadlocks.

当在回调中调用同步服务或action的情况下，容易发生死锁。

First thing to note here is that every node’s default callback group is a Mutually Exclusive Callback Group. If the user does not specify any other callback group when creating a timer, subscription, client etc., **any callbacks created then or later by these entities will use the node’s default callback group**. Furthermore, if everything in a node uses the same Mutually Exclusive Callback Group, that node essentially acts as if it was**handled by a Single-Threaded Executor, even if a multi-threaded one is specified!** Thus, whenever one decides to use a Multi-Threaded Executor, some callback group(s) should always be specified in order for the executor choice to make sense.

如果使用了mte，那么最好总是定义多个cbg，以使得多线程回调执行成功。

With the above in mind, here are a couple guidelines to help avoid deadlocks:

- If you make a synchronous call in any type of a callback, this callback and the client making the call need to belong to

  - different callback groups (of any type), or

  - a Reentrant Callback Group.

- If the above configuration is not possible due to other requirements - such as thread-safety and/or blocking of other callbacks while waiting for the result (or if you want to make absolutely sure that there is never a possibility of a deadlock), use asynchronous calls.

Failing the first point will always cause a deadlock. An example of such a case would be making a synchronous service call in a timer callback (see the next section for an example).

# 参考

[ROS2(四)- MultiThreadedExecutor多线程调用callback funcs_虎哥的铲屎员-CSDN博客](https://blog.csdn.net/liuerin/article/details/108749596)

[ROS2探索（二）executor_灰灰h的博客-CSDN博客](https://blog.csdn.net/qq_16893195/article/details/113123386)

[Using Callback Groups — ROS 2 Documentation: Galactic  documentation](https://docs.ros.org/en/galactic/How-To-Guides/Using-callback-groups.html)
