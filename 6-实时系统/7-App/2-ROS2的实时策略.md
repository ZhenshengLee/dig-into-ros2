# 术语

可预见性，确定性，顺序，

更新周期，截止日期，DEADLINE

# 参考

[Proposal for Implementation of Real-time Systems in ROS 2](https://design.ros2.org/articles/realtime_proposal.html)

# 实时性需求

## 系统需求

实时系统需求根据不同用例有不同的需求，核心的需求在两个方面

- 延迟

  - 更新周期（截止日期）

  - 可预见性（顺序和抖动）(确定性)

- 失效模式

  - 当截止日期失效时的行为设计

不同的失效模式对应着硬实时，准实时和软实时。硬实时系统中过期意味着系统失效，软实时系统中则放弃计算结果，降低系统性能。

（上面是单机系统的需求，还有一个是网络通信的需求，由DDS来把控）

## 操作系统架构

操作系统有非常多的指标：内核数量，内存大小，指令集架构，有一些指标有利于实时系统的搭建。

这些指标的选择取决于成本，尺寸，能源经济性，这些是约束性条件。

一个机器人系统可能具备多个计算机，多种处理器架构。

除了硬件，操作系统的选择也是需要考虑的。硬实时操作系统是绝对的确定性系统，软实时操作系统的设计目标只是满足大部分程序的截止日期，抖动较低，但是不是确定性的。大部分传统的用户接触到的操作系统的设计都不是以确定性为第一目标，这些系统为了人机交互的方便，舍弃了确定性。

我们选择实时系统，是为了下列目标：

- 任务调度

  - 线程优先级和确定性的调度

  - 允许线程抢占

- 并发和同步原语

  - Floored semaphores (allow a higher priority thread to take control of locked semaphore)

- 中断处理

  - 去使能中断

  - 屏蔽中断

- 内存分配

  - 避免页故障

  - 避免不确定的堆内存分配算法

一些实时系统有非实时的组件，需要实时组件和非实时组件之间的通信。

在ROS1中的例子是orocos的组件和ROS组件通信，另一个例子是实时性代码和GUI组件。

非实时的线程和实时线程在一起调度会造成系统的非确定性。非实时线程必须比实时线程更低的优先级，这样才不会抢占实时线程的截止日期。实时线程不能被非实时线程阻塞，不能被非实时线程抢占。

实时操作系统有多种措施来做到这些约束。比如，a Linux RT_PREEMPT system可以使用 [SCHED_IDLE](http://man7.org/linux/man-pages/man2/sched_setscheduler.2.html) scheduling policy应用到非实时线程。

Systems like QNX and Xenomai give the option to partition real-time and non-real-time components onto different cores, or even different kernels.将实时线程和非实时线程分配到不同的逻辑核或CPU核心。

资源分割是最安全的做法，将实时代码和非实时代码分开，不过也增加了实时代码和非实时代码通信的开销。

另一个约束条件是操作系统支持那种处理架构，以及哪种RTOS可以满足性能要求。例如不支持SIMD就不可以更好利用并行机制，如果OS不支持线程，则不支持多核，这些限制也就不存在了。

# 实现策略

With judicious application of the performance patterns and benchmarking tests proposed in this document, implementing real-time code in C/C++ is feasible. The question of how ROS 2 will achieve real-time compatibility remains.

允许ROS节点在启动和结束阶段不是实时安全，但是ROS节点交互，特别是进程内通信，则需要是实时安全的。

- 创建一个配置选项，允许进入实时友好模式。

  - 优点：

    - 允许用户动态地从实时模式切换到非实时模式

  - 缺点：

    - 重构代码需要成本，将现有代码集成入

- 实现一个实时的计算客户端rclrt, rmwrt

  - 优点：

    - 设计和实现简单一些

    - 可以充分进行优化

  - 缺点：

    - 开发成本大

    - 对用户不友好，需要切换包

- 在软件栈的某个层级上允许设置实时性，实现一个实时的语言客户端，rclrt, rclc

  - 优点：

    - 当前的代码可以复用

    - 用户可以提供内存分配策略来确保操作的确定性

    - 同步原语在编程语言层面，所以重构rcl/rmw更加简单

    - 通过不同的wrappers可以方便支持更多平台

  - 缺点：

    - 更多的测试代码

    - 灵活性更大，对用户来讲更为复杂

第三个策略最具性价比。

# ROS2的策略和目标

将上述需求分解到ROS2上，构成具体的技术需求。

我们的策略是迭代策略，分析出API的性能瓶颈，将代码分为实时的和非实时的行为，解决掉性能瓶颈

1. 选择两种用例

- STM32微控板，运行一个实时操作系统，1 kHZ update frequency with 3% allowable jitter.

- x86计算机，打上实时补丁，通过以太网或者USB接受传感器信息，在硬实时或者软实时的约束下处理传感器数据，但是也和一些非实时的进程一起运行，比如GUI进程

1. ROS2的实现

1. 度量系统性能

1. 寻找提升系统性能的解决方案，例如从ROS2抽象中暴露出更多的并发原语

1. 调整ROS2API来实现这些解决方案，增强实时特性

1. 重构API，迭代

# 软件架构

当前的API暴露了很多不确定性，应该暴露可选的API。这一部分评估当前rclcpp的问题，提出实时需求的挑战，给出可能的解决方案

## 内存分配

## 调度和同步

并发非常有用，因为能降低延迟

就算只有一个核心，并发也是有用的，因为异步模式在ROS中非常普遍

并发同步上，通用做法是通过复杂的同步来降低系统的可预测性。例如mutex如果不恰当使用可能造成死锁。所以实时系统通常提供其他原语来保证实时安全

在ROS2中遵循以下原则：

- 尽可能用atomics来替代互斥量

  - 很多原子类型并不是无锁设计，使用`std::atomic_is_lock_free`

- 充分测试多线程代码，避免死锁

  - 使用 tests with the [Clang ThreadSanitizer](http://clang.llvm.org/docs/ThreadSanitizer.html).

- 使用同步库，避免直接使用同步原语

  - 使用操作系统或者第三方提供的同步库

  - 比如ecl_threads

当前，Executor并没有遵循上述原则，同步原语的屏蔽工作也没做好。所以上述工作要做到需要强力的编码规范。

## 中间件和QoS

如果ros2下层使用的中间件不满足实时的性能，那么整个系统将无法满足实时性要求。

网络环境下，网络通信，网络发现，异步通信等都带来了不确定性。尽管一些实时操作系统部分解决了这个问题，仔细选择一款中间件依然是非常重要的。

dds的实时性性能，如何测试？

qos也是dds的一个重要指标，一些qos选项包含了实时性要求，比如DEADLINE策略。可以将这些策略集合成一个实时性的QoS策略。

## 组件生命周期

值得重视的是，节点声明周期对于实时性编程非常有用

组件有四个主要的状态：

- unconfigured

- inactive

- active

- shutdown

如果一个节点能够区分四个状态，并且能够接受外部指令进行切换，就OK了

![](https://tcs.teambition.net/storage/3129c93c430e54be4089399cb71bc7b48a58?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMzQwMywiaWF0IjoxNjcxNDE4NjAzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjljOTNjNDMwZTU0YmU0MDg5Mzk5Y2I3MWJjN2I0OGE1OCJ9.s8A1Wn4agcyuvAh7jSJpTy4HOjGgkmxG7fQJM7t2Ybk&download=image.png "")

[demos/README.rst at master · ros2/demos](https://github.com/ros2/demos/blob/master/lifecycle/README.rst)

# 测试

设计出的系统必须经过正确性测试，延迟参数必须进行基准测试。

还必须写单元测试，开发性能测试工具。

## 基准测试

可能的度量包括：

- 延迟的最小值，最大值，平均值

  - 循环更新的延迟

  - 消息经过dds双向传输时延

- 延迟的标准差

- 最后期限超过的次数

- 发送消息和接受消息个数之差

rttest是一个好用的工具

## 度量工具

Two well-established open source command line profiling tools with orthogonal methodologies are:

- `perf`: counts system-wide hardware events, e.g. instructions executed, cache misses, etc. (see [perf wiki](https://perf.wiki.kernel.org/index.php/Main_Page))

  - Linux only.

- `gprof`: recompiles source code with instrumentation to count time spent in each function for a particular program, with option to format output in a call graph (see [GNU gprof documentation](https://sourceware.org/binutils/docs/gprof/)).

  - GNU and C/C++ only.

然后这些工具对于RTOS或者裸机系统不友好，gperf只对Linux可用，gprof要求使用gcc编译的代码。

# 验证

自动化测试

回归测试
