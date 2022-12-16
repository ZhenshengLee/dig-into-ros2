# 概述

对于大量机器人系统，都有实时性需求以及确定性执行的需求。ROS作为一个SOA的架构，可以快速集成大量功能，但是对执行管理并没有提供足够的控制。比如，没有机制保证执行的顺序。执行顺序对于移动机器人来讲是必不可少的，但是没有机制来强制生效。所以，当前的ROS对于每一次bag的执行，都会产生不同的结果，其原因就是调度的不确定性。

很多保证确定性执行的办法，包括改变线程优先级，这是有效的。但是这个方法容易发生错误，难以扩展，而且对开发者素质要求较高。

所以，实时的executor的目标是提供易于使用的实时机制：

- 确定性执行

- 实时性保证

- 与平台中其他非实时的组件集成

- 对RTOS和MCU的支持

在ROS1中，roscpp会生成一个网络线程专门接受消息，放入一个FIFO中，所以所有回调都按照FIFO的顺序来执行，没有任何执行管理。ROS2中引入了DDS，消息缓存在DDS中，在ROS2中，executor的概念引申出来，用作执行管理，比如优先级排序。在rcl这一层，一个wait-set配置好，等待完毕后从dds的队列中获取句柄，这个句柄可以是定时器，订阅，客户或者服务等等。

标准执行器其实具备一些不好的特性。

- 定时器事件在所有事件队列之前

- rr调度不可抢占

- 每个事件类型只有一个输入

上述特性导致一个后果，rclcpp executor的行为不是确定的，实时性很难。

挑战在于：

- 开发一个合适的完整定义的调度机制

- 开发一个易于使用的API

- 对实时性需求建模

- 将ROS框架的调度和OS的调度映射起来

我们的方法是提供real-time executor，在rclc和rclcpp中提供。

首先，我们提出LET的执行器，实现静态顺序调度策略，根据执行时间的语义。在这种调度策略中，所有回调在预定义的顺序中执行，LET逻辑执行时间指的是，第一个输入数据已读和任务执行时的时间。

第二，我们开发了CGE，可以对一组回调进行优先级设定。

# 现有架构

The ROS 2 design defines one Executor (instance of [rclcpp::executor::Executor](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/executor.hpp)) per process, which is typically created either in a custom main function or by the launch system. The Executor coordinates the execution of all callbacks issued by these nodes by checking for available work (timers, services, messages, subscriptions, etc.) from the DDS queue and dispatching it to one or more threads, implemented in [SingleThreadedExecutor](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/executors/single_threaded_executor.hpp) and [MultiThreadedExecutor](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/executors/multi_threaded_executor.hpp), respectively.

The dispatching mechanism resembles the ROS 1 spin thread behavior: the Executor looks up the wait sets, which notifies it of any pending callback in the DDS queue. If there are multiple pending callbacks, the ROS 2 Executor executes them in an in the order as they were registered at the Executor.

![](https://tcs.teambition.net/storage/31292a2435a4f0ffb23f3885967cd6030bc4?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzAxNCwiaWF0IjoxNjcxMTgyMjE0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjkyYTI0MzVhNGYwZmZiMjNmMzg4NTk2N2NkNjAzMGJjNCJ9.AVptb2D_F_6isDxgJDXxsNIBOWEMKoQ476IoMfXGR3k&download=image.png "")

Also, the Executor does not maintain an explicit callback queue, but relies on the queue mechanism of the underlying DDS implementation as illustrated in the following sequence diagram:

依赖dds的顺序排布

![](https://tcs.teambition.net/storage/312962dc10ab55fcbd36501452c93a47b561?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzAxNCwiaWF0IjoxNjcxMTgyMjE0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk2MmRjMTBhYjU1ZmNiZDM2NTAxNDUyYzkzYTQ3YjU2MSJ9.fSZ1V9nWjvoavI8MVny1Jpj-OM_8_SniP_t3k5Iuimk&download=image.png "")

The Executor concept, however, does not provide means for prioritization or categorization of the incoming callback calls. Moreover, it does not leverage the real-time characteristics of the underlying operating-system scheduler to have finer control on the order of executions. The overall implication of this behavior is that time-critical callbacks could suffer possible deadline misses and a degraded performance since they are serviced later than non-critical callbacks. Additionally, due to the FIFO mechanism, it is difficult to determine usable bounds on the worst-case latency that each callback execution may incur.

不能对执行顺序进行细粒度的控制，也不能和OS调度器结合起来。

所以不能确保回调执行的最大延迟时间。

# 现有调度策略

The Executor distinguishes four categories of callbacks: *timers*, which are triggered by system-level timers, *subscribers*, which are triggered by new messages on a subscribed topic, *services*, which are triggered by service requests, and *clients*, which are triggered by responses to service requests. The Executor is responsible for taking messages from the input queues of the DDS layer and executing the corresponding callback. Since it executes callbacks to completion, it is a non-preemptive scheduler, However it does not consider all ready tasks for execution, but only a snapshot, called readySet. This readySet is updated when the Executor is idle and in this step it interacts with the DDS layer updating the set of ready tasks. Then for every type of task, there are dedicated queues (timers, subscriptions, services, clients) which are processed sequentially. The following undesired properties were pointed out:

- Timers have the highest priority. The Executor processes *timers* always first. This can lead to the intrinsic effect, that in overload situations messages from the DDS queue are not processed.

- Non-preemptive round-robin scheduling of non-timer handles. Messages arriving during the processing of the readySet are not considered until the next update, which depends on the execution time of all remaining callbacks. This leads to priority inversion, as lower-priority callbacks may implicitly block higher-priority callbacks by prolonging the current processing of the readySet.

- Only one message per handle is considered. The readySet contains only one task instance, For example, even if multiple messages of the same topic are available, only one instance is processed until the Executor is idle again and the readySet is updated from the DDS layer. This aggravates priority inversion, as a backlogged callback might have to wait for multiple processing of readySets until it is considered for scheduling. This means that non-timer callback instances might be blocked by multiple instances of the same lower-priority callback.

Due to these findings, the authors present an alternative approach to provide determinism and to apply well-known schedulability analyses to a ROS 2 systems. A response time analysis is described under reservation-based scheduling.

# 实时需求分析

一个很常用的例子，进程是周期调用回调，进程可以赋予优先级，允许抢占。

![](https://tcs.teambition.net/storage/3129485a664646056bf1df3a61f8c59f62ba?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzAxNCwiaWF0IjoxNjcxMTgyMjE0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk0ODVhNjY0NjQ2MDU2YmYxZGYzYTYxZjhjNTlmNjJiYSJ9.X7hIDl09moQNX9YwTZkvwKLgdI3-OOkPVvEhYgi6kM8&download=image.png "")

默认情况是顺序调用任务，叫做cooperative scheduling.

![](https://tcs.teambition.net/storage/3129faab2801986056dc862d320e0dbee1e4?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzAxNCwiaWF0IjoxNjcxMTgyMjE0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjlmYWFiMjgwMTk4NjA1NmRjODYyZDMyMGUwZGJlZTFlNCJ9.1hzWOU0qbAd1CELv8EAKNg6AN66cVET1pNa4DkkUC1Y&download=image.png "")

研究表明，当cpu占用率低于69%时，使用算法，shorter period have a higher priority

在嵌入式实时系统中广泛采用固定周期抢占的调度算法， fixed-periodic preemptive scheduling and the assignment of priorities, which makes the time-triggered paradigm the dominant design principle in this domain.

然而，当抢占式调度算法使用时，进程间共享的数据，数据持久化问题会发生。这样会导致延迟和更新时间的抖动。有的时候还会导致资源抢占问题。

为了解决上述问题，提出了LET调度算法，n which communication of data occurs only at pre-defined periodic time instances: Reading data only at the beginning of the period and writing data only at the end of the period. The cost of an additional latency delay is traded for data consistency and reduced jitter.

这个概念已经应用到了汽车软件上。

![](https://tcs.teambition.net/storage/3129935ab5e5507c7204308e71f38b11162f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzAxNCwiaWF0IjoxNjcxMTgyMjE0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk5MzVhYjVlNTUwN2M3MjA0MzA4ZTcxZjM4YjExMTYyZiJ9.kzHFdUqsm2BhYhtcPg9XYVVlIbwvxaUExi82As1EauM&download=image.png "")

An Example of the LET concept is shown in Figure 3. Assume that two processes are communicating data via one global variable. The timepoint when this data is written is at the end of the processing time. In the default case (left side), the process p3 and p4 receive the update. At the right side of the figure, the same scenario is shown with LET semantics. Here, the data is communicated only at period boundaries. In this case, the lower process communicates at the end of the period, so that always process p3 and p5 receive the new data.

The described embedded use case relies on the following concepts:

- periodic execution of processes

- assignment of fixed priorities to processes

- preemptive scheduling of processes

- co-operative scheduling of tasks within a process (sequential execution)

- data synchronization with LET-semantics

上述需求在ros2的实现：

periodic activation is possible in ROS2 by using timers

preemptive scheduling is supported by the operating system and assigning priorities on the granularity of threads/processes that correspond to the ROS nodes

it is not possible to sequentially execute callbacks, which have no data-dependency

Furthermore data is read from the DDS queue just before the callback is executed and data is written sometime during the time the application is executed

While the `spin_period` function of the rclcpp-Executor allows to check for data at a fixed period and executing those callbacks for which data is available, however, with this spin-function does not execute all callbacks irrespective wheter data is available or not. So `spin_period` is not helpful to periodically execute a number of callbacks (aka tasks within a process). So we need a mechanism that triggers the execution of multiple callbacks (aka tasks) based on a timer. Data transmission is achieved via DDS which does not allow to implement a LET-semantics. To summarize, we derive the following requirements:

Derived Requirements:

- trigger the execution of multiple callbacks

- sequential processing of callbacks

- data synchronization with LET semantics

# 实际用例分析

## Sense-plan-act pipeline in robotics

我们来考虑实际情况下的软件设计模式，来获得确定性的行为

在不同的实际用例下，我们提出确定性行为的需求

A sensing phase to aquire sensor data, a plan phase for localization and path planning and an actuation-phase to steer the mobile robot. Of course, more phases are possible, here these three phases shall serve as an example. Such a processing pipeline is shown in Figure 4.

![](https://tcs.teambition.net/storage/312921a54cca843c246679dd0c42cf38b164?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzAxNCwiaWF0IjoxNjcxMTgyMjE0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjkyMWE1NGNjYTg0M2MyNDY2NzlkZDBjNDJjZjM4YjE2NCJ9.uUnDAcKPS8RgXRba22biF9MQZooDIg5LmWDRDRHY_kU&download=image.png "")

定位算法的质量高度依赖传感器数据的时间期限，理想情况下，只能处理最近的数据。一种方法是严格按照顺序，先处理传感器数据，然后进行plan。

但是，当前情况下，ROS2的执行器不能保证这种严格的顺序行为。在data-driven pipline下sense 和plan都需要用laser信息，这些节点处理信息的顺序是随机的。

所需，如果定义严格顺序，plan在sense之后才能处理数据。

erived Requirements:

- triggered execution of callbacks

## Synchronization of multiple rates

Often multiple sensors are being used to sense the environment for mobile robotics. While an IMU sensor provides data samples at a very high rate (e.g., 500 Hz), laser scans are availabe at a much slower frequency (e.g. 10Hz) determined by the revolution time. Then the challenge is, how to deterministically fuse sensor data with different frequencies. This problem is depicted in Figure 5.

![](https://tcs.teambition.net/storage/3129f78000756328ee6df90b7419e4b2aaa2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzAxNCwiaWF0IjoxNjcxMTgyMjE0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjlmNzgwMDA3NTYzMjhlZTZkZjkwYjc0MTllNGIyYWFhMiJ9.Y-GAIjxuxDXSvSkuM-zePdkSPZAMEinire6eSIYf61o&download=image.png "")

一种方法是写同步程序，Due to scheduling effects, the callback for evaluating the laser scan might be called just before or just after an IMU data is received. One way to tackle this is to write additional synchronization code inside the application. Obviously, this is a cumbersome and not-portable solution.

还有一种是按照统一频率处理数据，比如每次laser处理时处理50个IMU数据，这样两种数据都有相同的频率。For example by processing always 50 IMU samples with one laser scan. This approach is shown in Figure 6. A pre-processing callback aggregates the IMU samples and sends an aggregated message with 50 samples at 10Hz rate. Now both messages have the same frequency. With a trigger condition, which fires when both messages are available, the sensor fusion algorithm can expect always synchronized input data.

![](https://tcs.teambition.net/storage/3129d72e1ed384f348151cc3af5a796457e1?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzAxNCwiaWF0IjoxNjcxMTgyMjE0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjlkNzJlMWVkMzg0ZjM0ODE1MWNjM2FmNWE3OTY0NTdlMSJ9.QSOQBsgtnOhJ6EdQf-WkwS3l4W0NGOYPdiMXxZhrEeU&download=image.png "")

In ROS 2, this is currently not possible to be modeled because of the lack of a trigger concept in the ROS2 Executor. Message filters could be used to synchronize input data based on the timestamp in the header, but this is only available in rclcpp (and not in rcl). Further more, it would be more efficient to have such a trigger concept directly in the Executor.

在ros2中可以使用msg filter来进行时间戳对齐。

还有一种方法，在处理激光的时刻主动请求最近的IMU数据。

![](https://tcs.teambition.net/storage/31298cd1d251c18b498b85a042d8b037d213?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzAxNCwiaWF0IjoxNjcxMTgyMjE0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk4Y2QxZDI1MWMxOGI0OThiODVhMDQyZDhiMDM3ZDIxMyJ9.Soalp7RvN4dBeKWJ2EWI4hEhSJOkXyDUHsc9DutZdiw&download=image.png "")

Derived Requirements from both concepts:

- triggered execution

- sequential procesing of callbacks

## High-priority processing path

While path following is a permanent activity, obstacle avoidance is trigged by the environment and should be immediately reacted upon. Therefore one would like to specify priorities to activities. This is depicted in Figure 8:

![](https://tcs.teambition.net/storage/3129889a9e4d06e0ef8c623e1f649cb76429?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NzAxNCwiaWF0IjoxNjcxMTgyMjE0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk4ODlhOWU0ZDA2ZTBlZjhjNjIzZTFmNjQ5Y2I3NjQyOSJ9.JLa1AKrhWPV6UNDVJZaGdfrPl_nMNZ4QEC6xY65-tyM&download=image.png "")

Assuming a simplified control loop with the activities sense-plan-act, the obstacle avoidance, which might temporarily stop the robot, should be processed before the planning phase. In this example we assume that these activites are processed in one thread.

Derived requirements:

- sequential processing of callbacks

# 特性

Based on the real-time embedded use-case as well as the software architecture patterns in mobile robotics, we propose an Executor with the following main features:

- user-defined sequential execution of callbacks

- trigger condition to activate processing

- data synchronization: LET-semantics or rclcpp Executor semantics

As stated before, this Executor is based on the RCL library and is written in C to nativly support microcontroller applications written in C. These features are now described in more detail.

The rclc-Executor supports all event types like the ROS 2 rclc executor, which are:

- subscription

- timer

- service

- client

- guard condition

Sequential execution

- At configuration, the user defines the order of handles.

- At configuration, the user defines whether the handle shall be called only when new data is available (ON_NEW_DATA) or whether the callback shall always be called (ALWAYS).

- At runtime, all handles are processed in the user-defined order

  - if the configuration of handle is ON_NEW_DATA, then the corresponding callback is only called if new data is available

  - if the configuration of the handle is ALWAYS, then the corresponding callback is always. If no data is available, then the callback is called with no data (e.g. NULL pointer).

Trigger condition

- Given a set of handles, a trigger condition based on the input data of these handles shall decide when the processing is started.

- Available options:

  - ALL operation: fires when input data is available for all handles

  - ANY operation: fires when input data is available for at least one handle

  - ONE: fires when input data for a user-specified handle is available

  - User-defined function: user can implement more sophisticated logic

LET-Semantics

- Assumption: time-triggered system, the executor is activated periodically

- When the trigger fires, reads all input data and makes a local copy

- Processes all callbacks in sequential order

- Write output data at the end of the executor’s period (Note: this is not implemented yet)

Additionally we have implemented the current rclcpp Executor semantics (“RCLCPP”):

- waiting for new data for all handles (rcl_wait)

- using trigger condition ANY

- if trigger fires, start processing handles in pre-defined sequential order

- request from DDS-queue the new data just before the handle is executed (rcl_take)

The selection of the Executor semantics is optional. The default semantics is “RCLCPP”.

# API

The API of the RCLC-Executor can be divided in two phases: Configuration and Running.

Configuration phase

During the configuration phase, the user shall define:

- the total number of callbacks

- the sequence of the callbacks

- trigger condition (optional, default: ANY)

- data communcation semantics (optional, default ROS2)

As the Executor is intended for embedded controllers, dynamic memory management is crucial. Therefore at initialization of the RCLC-Executor, the user defines the total number of callbacks. The necessary dynamic memory will be allocated only in this phase and no more memory in the running phase. This makes this Executor static in the sense, that during runtime no additional callbacks can be added.

Then, the user adds handles and the corresponding callbacks (e.g. for subscriptions and timers) to the Executor. The order in which this takes place, defines later the sequential processing order during runtime.

For each handle the user can specify, if the callback shall be executed only if new data is available (ON_NEW_DATA) or if the callback shall always be executed (ALWAYS). The second option is useful when the callback is expected to be called at a fixed rate.

The trigger condition defines when the processing of these callbacks shall start. For convenience some default conditions have been defined:

- trigger_any(default) : start executing if any callback has new data

- trigger_all : start executing if all callbacks have new data

- trigger_one(&`data`) : start executing if `data` has been received

- user_defined_function: the user can also define its own function with more complex logic

With ‘trigger_any’ being the default, the current semantics of the rclcpp Executor is selected.

The data communication semantics can be

- ROS2 (default)

- LET

To be compatible with ROS2 rclcpp Executor, the existing rclcpp semantics is implemented as ‘ROS2’. That is, with the spin-function the DDS-queue is constantly monitored for new data (rcl_wait). If new data becomes available, then is fetched from DDS (rcl_take) immediately before the callback is executed. All callbacks are processed in the user-defined order, this is the only difference to the rclcpp Executor, in which no order can be specified.

Secondly, the LET semantics is implemented such that at the beginning of processing all available data is fetched (rcl_take) and buffered and then the callbacks are processed in the pre-defined operating on the buffered copy.

Running phase

As the main functionality, the Executor has a `spin`-function which constantly checks for new data at the DDS-queue, like the rclcpp Executor in ROS2. If the trigger condition is satisfied then all available data from the DDS queue is processed according to the specified semantics (ROS or LET) in the user-defined sequential order. After all callbacks have been processed the DDS is checked for new data again.

Available spin functions are

- `spin_some` - spin one time

- `spin_period` - spin with a period

- `spin` - spin indefinitly

# 评论

对回调执行的顺序要求在一个进程内部定义

对于嵌入式硬件这种直接定义任务的场合是OK的

对于Linux资源丰富的分布式进程，不适合。

# 参考

[Execution Management](https://micro.ros.org/docs/concepts/client_library/execution_management/)
