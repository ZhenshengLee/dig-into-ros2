本文来自ROS2社区论文，有删改

[https://docs.google.com/document/d/1O53xOVlk4zwsfNukLaDbSWfT64wnoD9xh2EjWwX43qo/edit#](https://docs.google.com/document/d/1O53xOVlk4zwsfNukLaDbSWfT64wnoD9xh2EjWwX43qo/edit#)

# 动机

将执行实体（比如node的概念，其实是业务）与计算资源（进程和线程）分离是通行的做法。在特定的系统上，需要使用内存池，也可能只需要一个线程，上下文切换开销可能很大。如果需要把安全性和性能考虑在内，我们需要一个灵活的executor设计。

在ROS2中有很多executor的设计，不过这些设计都没有很好地同步。

本文目的：

- executor设计的概念解析

- 社区已有的executor设计综述

- 将executor的设计目标聚焦到两种，一种是面向性能，一种是面向确定性

- 上述两种设计方法的实现方式

- 为上述的两种设计目标提供代表性的用例

![](https://tcs.teambition.net/storage/3127dfe67f836c1bc27b6a88eacb4667aa9e?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NjgxMywiaWF0IjoxNjcxMTgyMDEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdkZmU2N2Y4MzZjMWJjMjdiNmE4OGVhY2I0NjY3YWE5ZSJ9.jXmZrbmNiGEgCPB3HwUpVupHUR2d3bSWnWGp-zfJLQ8&download=image.png "")

![](https://tcs.teambition.net/storage/3127e950203af88c7dae6ee57e1d0e18ef61?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NjgxMywiaWF0IjoxNjcxMTgyMDEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdlOTUwMjAzYWY4OGM3ZGFlNmVlNTdlMWQwZTE4ZWY2MSJ9.VwckEMEcoV0eclqxY9lVKjVKXZIpiKwA167AqD4iicc&download=image.png "")

# Event Handling Theory

事件处理理论

当设计一个响应事件的系统时，有很多中方式，各有优缺点。总的来说，有两种事件处理模型，ovserver model观察者模型，以及reactor model反应器模型

## **观察者模式：**

　　也可以称为为 发布-订阅 模式，主要适用于多个对象依赖某一个对象的状态并，当某对象状态发生改变时，要通知其他依赖对象做出更新。是一种一对多的关系。当然，如果依赖的对象只有一个时，也是一种特殊的一对一关系。通常，观察者模式适用于消息事件处理，监听者监听到事件时通知事件处理者对事件进行处理（这一点上面有点像是回调，容易与反应器模式和前摄器模式的回调搞混淆）。

In the observer model (also known as the **listener model**), users register callbacks with the entity that generates events. When an event occurs, the entity **invokes all registered callbacks, allowing them to process and handle the event**. Since the entity that generated the event invokes the registered callbacks directly, this approach usually results in the **lowest latency**. However, since the callback is invoked every time the event occurs, this can become inefficient if there are a large number of events or events that occur at a high rate.

这就是ROS默认的callback机制

## **Reactor模式：**

　　reactor模式，即反应器模式，是一种高效的异步IO模式，特征是回调，当IO完成时，回调对应的函数进行处理。这种模式并非是真正的异步，而是运用了异步的思想，当IO事件触发时，通知应用程序作出IO处理。模式本身并不调用系统的异步IO函数。

![](https://tcs.teambition.net/storage/312767a9bd801b63161bb7c5eac6f812d77d?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NjgxMywiaWF0IjoxNjcxMTgyMDEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc2N2E5YmQ4MDFiNjMxNjFiYjdjNWVhYzZmODEyZDc3ZCJ9.RRu9DhanTeMdhKD3vCORncb9e028eg88dMQZXJ-paA8&download=image.png "")

反应器的特点：具体的处理程序不调用反应器，而是由反应器通知处理程序处理就绪的事件

缺点：不能支持大量的客户请求与耗时过长的请求，由于select串行化了所有的事件处理过程。

In the reactor model (also known as the**waitset model**), one component handles or dispatches calls for multiple events. Generally there is **an event multiplexor like the DDS Waitset or Linux's epoll** that is configured to be notified when any of a number of events occur. The **event handling thread will block and wait until events occur**. When an event occurs, the thread is unblocked and woken up. It can then check to see which events have been triggered. Depending on the triggered events, the event handler can then invoke the appropriate handler. Since an event is either triggered or it is not, it does not matter if an event has occurred 1 or 100 times since the last time it was handled as the handler will still only be invoked once. **This generally results in more efficient use of CPU resources and a higher total throughput. However, since events are aggregated and may not be handled immediately, this will result in higher latency.**

这就是waitset机制

## **Proactor模式：**

　　Proactor模式，即前摄器模式，也是一种高效的异步IO模式，特征也是回调，当IO事件完成时，回调对应的函数对完成事件作出处理。这种模式是真正意义上的异步，属于系统级的异步，通常要调用系统提供的异步IO函数进行IO处理。

![](https://tcs.teambition.net/storage/3127b6c5e7e54a25bfe5f5d54738aa477a82?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NjgxMywiaWF0IjoxNjcxMTgyMDEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdiNmM1ZTdlNTRhMjViZmU1ZjVkNTQ3MzhhYTQ3N2E4MiJ9.uAQ85yhEOuw5xU-KhjZqj_mXdG-WldQH3rnOEdso-Q8&download=image.png "")

## **event queue**

There are also some less common models that sit in between these two. One model that is being investigated by the ROS community is an event queue. In this model, as events occur they are added to a shared queue by the entities that are generating them. At the same time, the event handler/dispatcher is waiting for events to be added to the queue. As events are added, the event handler pops them from the queue and invokes the appropriate handler. The performance characteristics of this model are somewhere in between the observer and reactor models above. However, this model also has some drawbacks. Like the observer model, it can have problems with large numbers of events or when events are occurring rapidly. Since there is a queue in between the event producers and event consumers, it can be difficult to ensure that the queue is large enough to hold all events.

生产者消费者模式，有危险，不能保证队列足够大足以包含所有事件。

## 对比

**Reactor模式和Proactor模式之间的区别：**

　　Reacor模式不调用系统异步IO函数，是一种仿异步。而Proactor是系统层面上的真正的异步，调用系统提供的异步IO函数。

举个例子，以网络IO为例：当我们从套接字读取数据

　　1.如果是Reactor模式，那么，反应器会通知我们 “可以读取数据了”，然后调用回调函数，利用recv函数从套接字读取数据，类似于MFC中的CSocket，在我们重写OnRecieve时，内部要调用Recv函数从套接字读取数据。

　　2.如果是Proactor模式，那么会先调用WSARecv函数注册读事件，反应器会通知我们 “数据已经读取了”，回调函数触发时，数据已经被接收到事先提供的缓冲区中，整个IO过程是由操作系统完成的，而不需要我们自己调用recv函数来读取数据，直接在事先提供的缓冲区取数据就可以了。

**观察者模式和Reactor模式，Proactor模式的主要区别：**

　　观察者模式，也叫发布-订阅模式，主要是适用于对象间一对多的依赖关系，通常用作消息分发和处理。而Reactor模式和Proactor模式主要用于高效的IO模式，明显的特征是“回调”思想的运用，提高效率，避免没有必要的耗时的等待，与对象间的依赖关系无关。

These are two fundamental different API approaches. Working with separate callbacks for different inputs like samples of a subscription or having a trigger somewhere that leads to executing a node which then does polling access to the inputs. The callbacks approach is widespread in Infotainment and also what ROS users are used to. Executing a whole node that polls for data is what is normally used in a real-time system such as Driver Assistance (DA) systems and Autonomous Driving (AD) systems. It is also what Apex.AI is doing with a [**WaitSet and a PollingSubscription**](#_kl7h5dpykn61). It is like having one callback for all the processing.

主要有两种API方法，一个是callbacks方法，被广泛接受并且在ROS中使用。

一种是poll方法，在实时系统中广泛使用，尤其是DA，APEX.AI公司在开发一种基于waitset和pollingsubscription的机制，这种机制就像一个节点只有一个回调，处理所有事务。

## 业务分析

The callback approach is good when you want to process every sample and as fast as possible. The “execute a whole node and poll for samples” is the better choice if you

1. Want to have more execution control in the overall system

相对执行过程有更完整的控制

1. You want to control which node is executed when, across thread and process borders

选择执行哪一个节点，跨过进程和线程边界

2. You want to have defined execution orders and reduce context switching

定义执行的顺序，减少上下文切换

3. You want to support a system wide time schedule for time-triggered nodes

支持全局时间调度节点

4. You want to control the input data that is used for an execution

控制输入的数据

2. Want to use the history caching of the middleware, that allows to drop samples that will not be needed. E.g. lidar scan with 10 Hz and Odometry with 100 Hz. The latest odometry is needed when processing the lidar scan. All others can be dropped and need not to be processes with individual callbacks

丢弃不需要的数据

3. Want to configure a specific trigger condition. This could be time, a sample on a specific subscription or even more complex things. The trigger can be related to but even independent from the entities of your node

定义具体的触发条件，时间，样本内容或者更复杂的条件。

4. Want to process samples from different subscriptions together and not individually

从不同的订阅获取数据，汇聚统一处理

Some may argue that you can put the samples in the sample callbacks aside and then do the major processing when a timer or “main trigger” callback is coming. But then you pay with a lot of context switches for building something the polling approach is the better fit for.

你也许可以说，可以把多个样本放到同一个回调中，不过这造成了更大的上下文切换开销

There are nodes with entities like subscriptions and services. A good flexibility would be to be able to choose what shall be processed with individual callbacks and what is processed with polling in the node’s run() function (a function where work gets done). Additionally it would be good to be able to configure which of the entities shall trigger the node’s run() call. This is normally only a small subset of the entities. Maybe the trigger is even coming from outside the node as there is a global schedule or a strand of nodes and one triggers the other.

DA and AD applications can often be described with graphs of nodes. Bigger graphs can be separated in sub-graphs that shall be executed on a specific trigger and run in a single process. So we see that besides strands it would be cool to execute directed acyclic graphs of nodes with a configurable number of threads.

## 结论

Conclusions for the ROS2 executor

1. In AD and DA a major use case is: execute a node based on a specific trigger condition with the use of polling subscriptions.

2. For this use case using a waitset is more natural than using an event queue that collects all events of all entities

3. A user can directly wait for this waitset and then poll the subscription. This operation can also be hidden in a spin() call

4. The waitset can be a wrapped waitset of the middleware that is configured according to the desired trigger condition

5. For some middleware it makes much more sense to use an already existing waitset than to build a waitset in ROS layers with callbacks from the middleware.

We would strongly vote for an optimized waitset based executor. It probably makes sense to have different executors for different use cases but in combination with polling subscriptions the waitset makes most sense.

# 定义

**Executor**Coordinates the execution of all callbacks issued by these nodes by checking for available work (timers, services, messages, subscriptions, etc.).

协调回调的执行，回调由检查一系列的工作产生（定时器，服务，消息，订阅等等）

**Determinism**

确定性：优先级，可抢占性，时间顺序，屏蔽

**Scheduler**

调度器：调整执行的顺序

**Callback / Callback Groups**

回调/回调组：回调是事件处理器，不包含线程，只有逻辑

**WaitSet**

用于同步的资源

# 用例：典型场景

1. Lidar-based object detection formed as DAG (sub)graph. In this use case we take the lidar data from multiple sources and run the data through the processing pipeline as shown in the below Figure. The requirements imposed by the below system are as follows:

a. We need to be able to assign threads and nodes to the CPU cores

b. We need to be able to assign scheduling policies to the nodes and threads

c. We need to have no memory allocations in the runtime phase

d. Be able to build sequential pipelines and partially sequential pipelines

e. Be able to filter and fuse the data

f. Be able to do message communication intra and inter process

g. Be able to pre-empt or suspend nodes and threads

![](https://tcs.teambition.net/storage/31271f5aecfcc13fa218c3ae5adc8970d470?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NjgxMywiaWF0IjoxNjcxMTgyMDEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcxZjVhZWNmY2MxM2ZhMjE4YzNhZTVhZGM4OTcwZDQ3MCJ9.fxl7oAfANGTYqacPK_xl_iMiKyUEgmJLhJPua46TNGA&download=image.png "")

2. Navigation based on IMU and Laser data with the need to filter and fuse the data

![](https://tcs.teambition.net/storage/31279bd68d679f06498cb0f40ff084c71a7c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NjgxMywiaWF0IjoxNjcxMTgyMDEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc5YmQ2OGQ2NzlmMDY0OThjYjBmNDBmZjA4NGM3MWE3YyJ9.BIieO_BoY22rPWyFziYj-KlcumBPhRnW0kHKVdg-fHI&download=image.png "")

**我们的要求，是在满足数据基本速率的基础上尽量减少资源占用**

实时性的要求都要低一些。

# 当前进展综述

## 默认的executor

The design pattern behind the default ROS 2 executor is described in [**Ingo’s talk**](https://ec2a4d36-bac8-4759-b25e-bb1f794177f4.filesusr.com/ugd/984e93_ab196a02893e4fb583a06fea20b8a112.pdf), slides 8-12. This figure describes the main idea:

![](https://tcs.teambition.net/storage/3127c21fc4f8f824b97d38a20e31fc51885d?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NjgxMywiaWF0IjoxNjcxMTgyMDEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdjMjFmYzRmOGY4MjRiOTdkMzhhMjBlMzFmYzUxODg1ZCJ9.T50Sypp3xHM7Ts7gi_kl69UkXFvOT3jbrwWwZjwvN2c&download=image.png "")

Pros:

1. 1st executor implementation in ROS 2

2. It works as good as possible given the TLC it received

3. It supports callback groups

4. It comes in SingleThreaded and MultiThreaded variant

a. Though it would be nicer if the threading was decoupled from the execution

5. It  does not maintain an explicit callback queue

Cons:

1. It does not provide  guarantees on latencies

1. Performance is not the same as in ROS 1

2. It does not support setting of priorities for the nodes and topics

3. If there are multiple pending callbacks, the ROS 2 Executor executes them in an order as they were registered at the Executor (not possible to sequentially execute callbacks, which have no data-dependency)

4. It does not leverage the real-time characteristics of the underlying operating-system scheduler to have finer control on the order of executions

5. Since it executes callbacks to completion, it is a non-preemptive scheduler

6. Timers have the highest priority. The Executor processes *timers* always first. This can lead to the intrinsic effect, that in overload situations messages from the DDS queue are not processed.

7. Non-preemptive round-robin scheduling of non-timer handles. Messages arriving during the processing of the readySet are not considered until the next update, which depends on the execution time of all remaining callbacks. This leads to priority inversion, as lower-priority callbacks may implicitly block higher-priority callbacks by prolonging the current processing of the readySet.

8. Only one message per handle is considered. The readySet contains only one task instance, For example, even if multiple messages of the same topic are available, only one instance is processed until the Executor is idle again and the readySet is updated from the DDS layer. This aggravates priority inversion, as a backlogged callback might have to wait for multiple processing of readySets until it is considered for scheduling. This means that non-timer callback instances might be blocked by multiple instances of the same lower-priority callback.

9. The executor is tightly coupled to both threading (in an opaque manner) and nodes

10. No real preemptive scheduler in user-space

11. [**High CPU and Memory usage**](https://discourse.ros.org/t/singlethreadedexecutor-creates-a-high-cpu-overhead-in-ros-2/10077)

![](https://tcs.teambition.net/storage/3127da1b8f9b3ebb905b7c17ae197627969b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NjgxMywiaWF0IjoxNjcxMTgyMDEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdkYTFiOGY5YjNlYmI5MDViN2MxN2FlMTk3NjI3OTY5YiJ9.nNNR8_YsLSLcIkmoxVmUBMrBqczl-Vcyw10vPoJ7cHU&download=image.png "")

## Static Executor

[nobleo/rclcpp-static-executor](https://github.com/nobleo/rclcpp-static-executor#changes-in-the-staticexecutor.)

与默认机制一致，但是在spin之前所有资源初始化一次，然后开始spin

![](https://tcs.teambition.net/storage/31270dbe0174e7f1685bb914262c5683ea66?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc4NjgxMywiaWF0IjoxNjcxMTgyMDEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcwZGJlMDE3NGU3ZjE2ODViYjkxNDI2MmM1NjgzZWE2NiJ9.gyy3fF5Ja73Jp5bd1K8nSrwys1PJFX9qdWvkZSAsrXA&download=image.png "")

CPU降低一半以上。

这个机制可以用上，马上验证

Pros:

1. Reduced CPU and memory consumption

2. Otherwise the same as the default ROS 2 executor

Cons:

1. No MultiThreaded variant

2. Otherwise the same as the default ROS 2 executor

## RCLC-executor

The main concept of this executor is shown in Figure 3 in [**this report**](https://github.com/micro-ROS/micro-ROS.github.io/blob/master/_docs/concepts/client_library/real-time_executor/index.md#real-time-embedded-application-used-case) and it is based on the logical-execution time (LET) which was introduced in [HHK2001](https://github.com/micro-ROS/micro-ROS.github.io/blob/master/_docs/concepts/client_library/real-time_executor/index.md#HHK2001).  Here the communication of data occurs only at predefined periodic time instances: Reading data only at the beginning of the period and writing data only at the end of the period. The cost of an additional latency delay is traded for data consistency and reduced jitter. This concept has also recently been applied to automotive applications [NSP2018](https://github.com/micro-ROS/micro-ROS.github.io/blob/master/_docs/concepts/client_library/real-time_executor/index.md#NSP2018).

Pros:

● trigger the execution of multiple callbacks

● sequential processing of callbacks

● data synchronization with LET semantics

● possible to allocate new tasks in runtime

Cons:

1. Only implemented in *rclc*

2. The executor is tightly coupled to both threading (in an opaque manner) and nodes? (JS) What do you mean by this?

a. **Subscriptions are directly added to the executor, the node-concept does not play a role for the rclc-executor**

b. This version of the rclc-executor is single-threaded. That is, all callbacks will be executed in the process, which started the ROS application including the rclc-executor.

c. There is currently work-in-progress a multi-threaded executor: each callback is executed in its own thread, the executor manages accesses to rcl library single-threaded.

3. No real preemptive scheduler in user-space

a. Yes, the rclc-executor is single-threaded.

## WaitSet in ROS 2

Two basic things that WaitSet does: waiting and taking the data (the latter is normally done in the Executor). The main power of WaitSet is that users can explicitly decide in what order to do things and whether to wait again. Executor is the encapsulation of those decisions. The two main reasons why WaitSet was implemented in ROS 2:

1. Users can use things that they wait on, without an executor and decide what to do with these things

2. To move the executor forward => simplify it and make it more efficient by avoiding building of WaitSets, …

A longer walk through of the WaitSet implementation in ROS 2 is available in here:

1. [**https://github.com/ros2/rclcpp/pull/1047#issue-397007017**](https://github.com/ros2/rclcpp/pull/1047#issue-397007017)

2. [**https://github.com/rosobject2/rclcpp/pull/1047#issuecomment-731806325**](https://github.com/ros2/rclcpp/pull/1047#issuecomment-731806325)

Pros:

1. TBD

Cons:

1. From [**https://discourse.ros.org/t/ros2-middleware-change-proposal/15863/27**](https://discourse.ros.org/t/ros2-middleware-change-proposal/15863/27):

2. **Waitset performance**

3. The waitset has to be re-created every time

4. You have to iterate over all communication objects

5. Timer performance

● Timer handling is not efficient. It can be improved.

● The current approach makes it easier to combine timers with message handling. No need to bother about concurrency issues or message processing/timer ordering.

3. Message processing ordering

## WaitSet and PollingSubscription（APEX闭源）

In the current version of Apex.OS (Apex.AI fork of ROS 2), the event handling and execution model is implemented using the ApexNode along with the Waitset and PollingSubscription in a polling approach.

Each ApexNode contains one or more Publishers or PollingSubscriptions, as well as a thread which runs a user provided function in a loop. Within this function, users should set up a Waitset and wait until data becomes available. When this happens, the waitset will unblock and data can be read from any PollingSubscriptions that the waitset has indicated are readable. Once it has read all the messages that it needs, the thread function can then continue to algorithmic or processing steps and publish any messages. Once the end of the thread function is reached, the ApexNode will do a check to verify that the execution time was not too long before invoking the thread function again. This will continue until the node is shut down.

Pros:

1. ApexNode is single threaded

2. User is in complete control over the incoming data

3. This generally results in more efficient use of CPU resources and a higher total throughput.

4. This concept is suitable for determinism

Cons:

1. The node abstraction aims to be simultaneously a unit of logic and a unit of execution introducing an arbitrary limitation for the system design and being a bad example of a strong coupling between unrelated system aspects.

2. One thread per node is an unscalable model. The number of threads higher than a number of actual cores by orders of magnitude will lead to thrashing and significant waste of resources, looking system-wise.

3. A-thread-per-node model also renders reasoning about real-time applications with several 100s of nodes very hard. Even if some of them are scheduled as real-time, the context switch between those is not really predictable. The whole process of RT application planning often involves careful consideration of run-time for specific tasks with hand-crafted assignment of execution order, thread priorities and affinities. In those circumstance a-thread-per-node model can become a very limiting constraint which users will be forced to write around.

4. Since events are aggregated and may not be handled immediately, this will result in higher latency.

## Event Queue

In this model, as events occur they are added to a shared queue by the entities that are generating them. At the same time, the event handler/dispatcher is waiting for events to be added to the queue. As events are added, the event handler pops them from the queue and invokes the appropriate handler.

For details see [**https://discourse.ros.org/t/ros2-middleware-change-proposal/15863**](https://discourse.ros.org/t/ros2-middleware-change-proposal/15863).

Pros

1. Improved the performance of system load (compared to Default ROS 2 Executor)

2. Improved message **latency** (compared to Default ROS 2 Executor)

Cons

1. The queue is unbounded.

2. How to deal with overload situations or when events get out of date

3. How to handle timers. Is it okay to handle them in a separate user-thread?

4. With the history queue and the event queue there are now two queues that are accessed at different times and that behave in a different way. When the history queue in DDS overflows and drops samples the number of available samples in DDS and the number of events in the event queue diverge. This destroys the temporal order and leads to “empty” takes from rmw.

# 对ROS2社区的建议

支持实时性的工业级应用

To support ROS 2 industrial applications we propose to enable the future ROS 2 Executor to:

- Enable scheduling parameters in ROS 2 Executor

- Pass, e..g ‘sched_param’ the default POSIX scheduling parameter of the OS to the subscription, when it it added to the executor

- For each subscription callback there could be one thread spawned that is then created with pthread_create with these scheduling parameters (priorities or other availalbe scheduling options of the operating system)

- Benefit: real-time scheduling is configured as ROS 2 API and enforcement/management is done by the underlying operating system

- We have developed a proof-of-concept implementation for rclc-executor for micro-ROS with NuttX RTOS on STM32 microcontroller (paper is under submission, notification in mid March) When accepted, we can provide implementation.

- Have an interface to the ROS 2 Executor that allows to enable scheduling and deterministic execution

# 难题

1. What kind of cooperation do we want with the OS?

a. E.g. CPU pinning, memory locking, using the OS scheduling policies

b. How could OS scheduling parameters be assigned to ROS 2 Executor and enforced by the OS (see note in rclc-executor)?

2. Can we suspend or pre-empt the nodes?

3. What will schedule the Internal DDS implementation threads

4. How will we account for the change of constraints (e.g. change of transport, QoS, …)

5. In your perfectly predictable and scheduled system a task takes more time than it was allowed to (for some reason like CPU overheating). What happens now? You preempt it. But what if it never finished its work and now the whole pipeline is stuck?

6. How is ROS 2 Executor different from a C++ planned to be Executor: [**https://www.modernescpp.com/index.php/a-short-detour-executors**](https://www.modernescpp.com/index.php/a-short-detour-executors)

7. How will we do the scheduling and priorities in the[__ network stack__](https://arxiv.org/pdf/1808.10821.pdf)?

8. How will we combine executor with the [**parallel algorithms**](https://devblogs.microsoft.com/cppblog/using-c17-parallel-algorithms-for-better-performance/)?

a. Then we also have a different category for e.g. GPU processing

9. What is possible in the intra-proces and what in the inter-process case?

a. E.g. cooperative multitasking assumes that runners share the address space whereas multiprocess dispatch is a different level. Basically you need a process that spawns other processes, sets their priorities and monitors their exit status

# 工作项

# 参考资料

[boost asio学习笔记1--Reactor与Proactor模式_辛明辉的专栏-CSDN博客_asio reactor](https://blog.csdn.net/u013246898/article/details/52044091)

[对于观察者模式、Reactor模式、Proactor模式的一点理解_草上爬的博客-CSDN博客_reactor模式 与 观察者模式](https://blog.csdn.net/caoshangpa/article/details/79420547?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)

[Observer Pattern vs Reactor Pattern](https://stackoverflow.com/questions/26454642/observer-pattern-vs-reactor-pattern/26470697)

[https://github.com/micro-ROS/micro-ROS.github.io/blob/master/_docs/concepts/client_library/execution_management/index.md](https://github.com/micro-ROS/micro-ROS.github.io/blob/master/_docs/concepts/client_library/execution_management/index.md)
