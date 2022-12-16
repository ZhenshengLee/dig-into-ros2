irobot的解决方案， event based executor

传统的wait set based，由dds掌控

# 动机

ros2 executor的性能问题，拥有比较高的cpu占用率，虽然在foxy版本中已经有优化版本，但是效果不明显

- [Reducing ROS 2 CPU overhead by simplifying the ROS 2 layers](https://discourse.ros.org/t/reducing-ros-2-cpu-overhead-by-simplifying-the-ros-2-layers/13808)

  - 一个context对应一个participant，而不是一个节点

- [SingleThreadedExecutor creates a high CPU overhead in ROS 2](https://discourse.ros.org/t/singlethreadedexecutor-creates-a-high-cpu-overhead-in-ros-2/10077)

  - StaticSingleThreadedExecutor

  - Waitset重新设计

- [ROS 2 Middleware Change Proposal](https://discourse.ros.org/t/ros2-middleware-change-proposal/15863)

  - 新的events queue版本

ROS2的CPU开销使得在低端系统中部署软件变得困难，如rasperry pi这种带ubuntu，但是硬件规格较低的设备

# 需求

以下设计目标用来设计events queue based executor

- 高性能

  - executor不能成为ROS2系统的CPU性能瓶颈

  - 可以与下层的dds性能一较高下

- 可伸缩性

  - 性能不随着计算规模的扩大而扩大

  - 延迟时间是固定的而不是线性增加的

  - "You don't pay for what you don't use"

- 可扩展性

  - 允许直接增加功能，不影响当前功能的性能

- 顺序性

  - 按照事务产生的顺序处理事务（FIFO）

- 高精度和精确定时器

  - 提供最小延迟的定时器

# Events Executor

`EventsExecutor` does not use the concept of a waitset. The `EventsExecutor` is characterized by two main components, i.e. the `EventsQueue` and the `TimersManager`.

没有使用waitset

需要所有实体在有工作要做的时候尽快将其加入到executor的eventQueue中，events是在RMW层生成的，下层的中间件通知ROS2实体有新的工作要做，而events是在rclcpp层消费执行的。events这个数据结构包括了事件类型以及事件对应的对象的句柄。

- `ExecutorEventType::SUBSCRIPTION_EVENT` and an identifier for a `rclcpp::SubscriptionBase` object.

- `ExecutorEventType::SERVICE_EVENT` and an identifier for a `rclcpp::ServiceBase` object.

- `ExecutorEventType::CLIENT_EVENT` and an identifier for a `rclcpp::ClientBase` object.

- `ExecutorEventType::WAITABLE_EVENT` and an identifier for a `rclcpp::Waitable` object.

考虑回调是如何执行的，中间件会通知rmw_subscription对象新消息发送到来，rmw_subscription对象需要将一个

和普通executor一样，需要通过add_node()来与这些实体相关联。

不同实体类型有不同的初始化方式

Client/Server/Subscription

These entities have a 1-to-1 correspondence between objects in the rclcpp and the rmw layer. The initialization is straight forward and consists in having the `EventsExecutor` to pass the aforementioned items to the rclcpp subscription, which then will forward them to rcl and finally to rmw.

Waitable

`Waitables` are a concept that exists only in the rclcpp layer. Each class that inherits from `Waitable` will have to define its own function for forwarding items to the rmw layer. For example, a `SubscriptionIntraProcess` relies on a guard condition in the rmw layer, so it will forward to it the items needed for pushing events. On the other hand, a `QOSEventHandler` will forward the aforementioned items to the underlying rmw QoSEvent implementation.Note that `Waitables` can be used as a way to implement generic custom events, as it will be described in the next sections.

Timer

Timers are not required to push events into the `EventsExecutor`'s `EventsQueue`, so new timer entities are simply redirected to the `TimersManager` object.

# 静态结构

![](https://tcs.teambition.net/storage/3127c9f9a1fa82ffa099a4f2a648be3b87ea?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdjOWY5YTFmYTgyZmZhMDk5YTRmMmE2NDhiZTNiODdlYSJ9.1CcEyTZo_4cls7ewcVQRbk-9WK5bxQVdBu_v9wNTW5k&download=image.png "")

We believe that rather than using user level waitsets, that an executor event queue design will allow events to propagate faster. When the executor thread is waiting on events to arrive, it simply blocks allowing the CPU to perform other work. When awakened, the events are processed in the order they are received. Each event contains the a type enumeration and a unique handle to process the event.

![](https://tcs.teambition.net/storage/3127070d26990f37bd2146e81c992d366c44?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcwNzBkMjY5OTBmMzdiZDIxNDZlODFjOTkyZDM2NmM0NCJ9.zJvW6Cb4MM4m47QHY0bVJfYuA4KhrwXGu6Tvd6PcEI4&download=image.png "")

**IMPLEMENTATION DETAILS :**

**Use a push interface with a queue to signal events to the executor.**

This allows the custom middleware event handlers to notify the executor immediately via a callback pointer. The provided event type and handle allows access to the resource without searching. This callback datatype is (currently) part of the rcutils and does not require the executor data types to pollute the middleware implementation. This design removes the overhead of adding, polling, and checking the waitsets for subcriptions, clients and services when checking for new work.

Slower list maintenance is only performed when the resource is created or deleted.

Currently there is a single queue per executor with events presented to the user in received order with no implied priority of event types.

**Executor blocks waiting on Queued Events**

In this design the executor blocks waiting on a queue event. When signaled, the events are removed and processed. This will work using spin(), spin_some(), or spin_once() if needed by blocking or processing the entire queue, or only processing some of the items. This blocking method will allow the thread to wait without consuming CPU until user work is ready to be executed.

**Offload timers to its own thread**

The current design requires timer maintenance to be performed during the rcl wait operations. In this design we propose to offload the timer operations to its own thread and signal the executor of timer expiry using the event queue. An alternative interface could also execute the callback directly if the user specified it as such, however thread safe practices would apply to the user in that case.

Another advantage of timer offload is that the underlying operating system timer facilities can can be used to better manage the pool. Since these operate closer to the operating system scheduler, more accurate results could be obtained, especially for longer duration intervals.

# 动态结构

![](https://tcs.teambition.net/storage/3127196896d5b8d713f5adb0379f191b9e5b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcxOTY4OTZkNWI4ZDcxM2Y1YWRiMDM3OWYxOTFiOWU1YiJ9.gCKFzM1oq1Ndn43wzMKF2WR9WR7Y201YV8eKLLfbnks&download=image.png "")

# PPT

怎样保证顺序

![](https://tcs.teambition.net/storage/3126d8350b0def53108ae58a818e36952dae?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZkODM1MGIwZGVmNTMxMDhhZTU4YTgxOGUzNjk1MmRhZSJ9.B-TbqQXQ5HqAxzTrhBUCfzcPDnwGumnhfiy6ttLku7c&download=image.png "")

![](https://tcs.teambition.net/storage/3126ec98ed1e001c63bff9d3836aae5249cc?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZlYzk4ZWQxZTAwMWM2M2JmZjlkMzgzNmFhZTUyNDljYyJ9.KOKDWYY_WpapzaWS0EhBAl3dJwTT8G6y4aZYK2md4S4&download=image.png "")

解决思路：

- 离middleware更加近，对于资源受限系统会更加友好，可以减少延迟和抖动

- 额外的队列，阻碍middleware qos

In order to execute generic ROS 2 entities (i.e. everything except timers), the `EventsExecutor` relies on an event queue. Whenever a ROS 2 entity that is associated to the executor has work to do, it will push an event into the executor's queue. Then the executor can process these events in a FIFO manner, without need for expensive entities look-ups. Processing an event results in different operations depending on the entity that generated it.

![](https://tcs.teambition.net/storage/312707c3805efb00429cebe2aef16c288898?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcwN2MzODA1ZWZiMDA0MjljZWJlMmFlZjE2YzI4ODg5OCJ9.dDz88QRJRuST7U-SpD-PVKr2H-Faeg6lc_zOVcitPgw&download=image.png "")

是不是有ROS1似曾相识的感觉？

# 总结

- Problems with the current waitset implementation:

  - Waitset performance

  - The waitset has to be re-created every time

  - You have to iterate over all communication objects

  - Timer performance

  - Timer handling is not efficient. It can be improved.

  - The current approach makes it easier to combine timers with message handling. No need to bother about concurrency issues or message processing/timer ordering.

  - Message processing ordering

- Problems with the event queue proposal:

  - The queue is unbounded.

  - How to deal with overload situations or when events get out of date

  - How to handle timers. Is it okay to handle them in a separate user-thread?

- To solve the unbounded queue issue, Ingo proposes:

  - Use a ready-set, list or fixed size queue/array of communication object handles.

  - Set a flag to signal the data is ready

  - Take all the data and process in order

  - Use a dirty flag to prevent adding events that are already added

- The problem of taking everything is this could lead to counter-intuitive situations where the data handling is overloaded and data is skipped/dropped. This could be acceptable only if a temporary overload is expected.

  - *(Note from William): imo, the counter-intuitiveness of the behavior is not due to dropping or skipping the messages, but instead due to the fact that messages taken in batch may result in processing old data when newer data is available for signification periods of time when the history depth is more than 1*

- On the other hand, taking just one message data is a bad idea.

  - *(Note from William): I don’t think it’s necessarily a bad idea, it’s just got performance problems due to querying the wait set so frequently (after each event). It also contributes to non-determinism when dealing with multiple subscriptions and/or timers due to scheduling race conditions.*

- Another option to solve the unbounded queue issues is to set a maximum queue size.

  - Open questions. What to do when the queue is full? Prune the queue?

- How DDS users typically handle timers

  - Create a user thread to handle the timers

  - timers set guard conditions

# 可行性论证

之前的设计都是“leave it to the middleware”，但是我们并没有关注dds给了我们什么好功能。

离middleware更加近，对于资源受限系统会更加友好，可以减少延迟和抖动

![](https://tcs.teambition.net/storage/31272d397e49696b34738c67d1e527951d73?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcyZDM5N2U0OTY5NmIzNDczOGM2N2QxZTUyNzk1MWQ3MyJ9.4aAvWWERJMFkbQnaJO5JW8v_uIRYOCnyPkIpCacnp_A&download=image.png "")

staticsinglethreaded基础性能是30%，直接访问dds提供的wait_set，使得cpu降低到20%，但是使用events降到了11%

![](https://tcs.teambition.net/storage/3127763de5e1435b0a9d70dbc10b4656410e?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc3NjNkZTVlMTQzNWIwYTlkNzBkYmMxMGI0NjU2NDEwZSJ9.1k-1h-nvkQTKYlZrhwBxi4JGdbBLmr1M3viPHTDjfLM&download=image.png "")

一个是事件结构，一个是生产者消费者结构，事件结构当然没有

每个executor维护一个queue，接受消息，处理回调，如果处理的回调速度一直跟不上接收的速度，就会爆炸。

![](https://tcs.teambition.net/storage/3127e1221633c49333ebb88108f6f4adf62a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdlMTIyMTYzM2M0OTMzM2ViYjg4MTA4ZjZmNGFkZjYyYSJ9.UCpGjzIcwe5LZWspDUEdF6aPCGiM640j415QX6XDnXU&download=image.png "")

将qos队列定义为10，1对queue的影响？

![](https://tcs.teambition.net/storage/3127089dfa65da7f9b4977db892ec999224c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcwODlkZmE2NWRhN2Y5YjQ5NzdkYjg5MmVjOTk5MjI0YyJ9.RTsu8T7I96SPtDWZR-8zJRStGZ5VrhUBOPFplDoxOms&download=image.png "")

这个到底有没有问题？没有问题

只能是单线程的，多线程的正在开发

![](https://tcs.teambition.net/storage/3127b781f47f6bca1ab694a4d6858d4461fd?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdiNzgxZjQ3ZjZiY2ExYWI2OTRhNGQ2ODU4ZDQ0NjFmZCJ9.EM99LPDi6HPi0J8PzPXMCny7abK3rWLWSWOxGeKlCig&download=image.png "")

# 代码pr

[https://github.com/ros2/rmw_connextdds/pull/44](https://github.com/ros2/rmw_connextdds/pull/44)

![](https://tcs.teambition.net/storage/312729a67b2c7b49a2ccdefc96f50c3f61a5?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzA3NiwiaWF0IjoxNjcxMTgyMjc2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcyOWE2N2IyYzdiNDlhMmNjZGVmYzk2ZjUwYzNmNjFhNSJ9.iyRoP8bEnmMxi4fyhx4dG7q4vfbNfN5JpQNsUl1BQFg&download=image.png "")

The new executor uses an events queue and a timers manager as opposed to waitsets, to efficiently execute entities with work to do.

This new executor greatly reduces CPU usage of a ROS 2 application.

See the blog post for more details on the tests that we run.The bulk of the changes for this implementation are in the rclcpp layer, with some minor changes in other repositories (rcl, rmw, rmw_implementation) for forwarding entities, the declaration of some data types in rcutils, and finally some additional changes in the vendor specific rmw implementations..

We currently implemented this only on top of the default ROS middleware fastrtps, while we provided stubs for other middlewares.

The current implementation does not support ROS 2 actions

关键概念在event，listener，callback 之间反复横跳，关键字总是变更，api总是变更。

代码还在不断重构，foxy的版本不是最终版

文档

[https://github.com/ros2/design/pull/305](https://github.com/ros2/design/pull/305)

rclcpp

[https://github.com/ros2/rclcpp/pull/1579](https://github.com/ros2/rclcpp/pull/1579)

rcl

rmw

# 测试

## 官方测试

测试平台为Raspberry Pi1

OS: [Raspbian](https://www.raspberrypi.org/software/) is a Debian-based engineered especially for the Raspberry Pi and it is the perfect general-purpose OS for Raspberry users.

## 实际测试

# 二次开发

默认的rmw_fastrtps已经做好

主要工作是rmw_iceoryx适配，打桩加开发，使用listern api

疑问，direct access to DDS wait set，具体是做了什么？

我看到rcl_wait_set的data成员已经是dds的wait_set了。

# 参考

[Events Executor design by irobot-ros · Pull Request #305 · ros2/design](https://github.com/ros2/design/pull/305/files)

[Events Executor design by irobot-ros · Pull Request #305 · ros2/design](https://github.com/ros2/design/pull/305)

[irobot-ros/rclcpp](https://github.com/irobot-ros/rclcpp/tree/foxy-events-executor)

[https://github.com/ros2/design/pull/305](https://github.com/ros2/design/pull/305)

[SingleThreadedExecutor creates a high CPU overhead in ROS 2](https://discourse.ros.org/t/singlethreadedexecutor-creates-a-high-cpu-overhead-in-ros-2/10077/23)

[ROS2 Middleware Change Proposal](https://discourse.ros.org/t/ros2-middleware-change-proposal/15863)

[design/events_executor.md at gh-pages · irobot-ros/design](https://github.com/irobot-ros/design/blob/gh-pages/articles/events_executor.md)
