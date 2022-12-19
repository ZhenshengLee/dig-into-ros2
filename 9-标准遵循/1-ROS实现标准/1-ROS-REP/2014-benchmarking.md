# 概述

本篇REP描述了ROS2的性能基准测试的原则和指南

# 动机

基准测试是在确定工作负载的计算机中运行程序来确定程序的相对性能的过程。在ROS2的场景下，性能信息能够帮助工程师设计更加高效的系统，选择合适的硬件。基准测试也可以帮助理解不同算法的开销，使得工程师可以为具体场景选择不同的实现。性能测试数据也可以用来对比不同版本的ROS2，进行回归测试。最后，性能数据能够帮助对未来工作评定优先级。<br />由于机器人系统中硬件和软件是高度耦合的，这使得通用的基准性能测试变得异常困难，因为要考虑架构无关，可表示性和可复现性等种种因素。该REP尝试提供多种指南来帮助机器人工作者以定量，可复现，可重复的方式进行性能基准测试。该REP同时提供了一系列工具和样例。<br />对相关人员的价值：

- Package maintainers can use these guidelines to integrate performance benchmarking tools (e.g. instrumentation) and data (e.g. results, plots and datasets) in their packages.
- Consumers can use the guidelines in the REP to benchmark ROS Nodes and Graphs in an architecture-neutral, representative, and reproducible manner, as well as the corresponding performance data offered in ROS packages to set expectations on the capabilities of each.
- Hardware vendors and robot manufacturers can use these guidelines to show evidence of the performance of their systems solutions with ROS in an architecture-neutral, representative, and reproducible manner.

# 概念

## 定量方法

Performance must be studied with real examples and measurements on real robotic computations, rather than simply as a collection of definitions, designs and/or marketing actions. When creating benchmarks, prefer to use realistic data and situations rather than data designed to test one edge case. Follow the quantitative approach [1]_ when designing your architecture.<br />在真实机器上测试得到数据，而不是通过市场调查<br />使用真实数据，使用实际的算法测试数据，在设计架构时候遵循定量方法

## 基准性能测试

有很多种不同的基准测试方案，特别是在性能测试上。下面给出不同的定义：

- `Functional performance testing`: Functional performance is the measurable performance of the system’s functions which the user can experience. For example, in a motion planning algorithm, measures could include items like the ratio the algorithm gets the motion plan right.
  - 功能性能测试，涉及到算法的功能指标
- `Non-functional performance testing`: The measurable performance of those aspects that don't belong to the system's functions. In the motion planning example, the latency the planner, the memory consumption, the CPU usage, etc.
  - 非功能性能测试，部涉及到系统的功能，比如说程序的执行时间，cpu资源占用率
- `Black-box performance testing`: Measures performance by eliminating the layers above the *layer-of-interest* and replacing those with a specific test application that stimulates the layer-of-interest in the way you need it. This allows to gather the measurement data right inside your specific test application. The acquisition of the data (the probe) resides inside the test application. A major design constraint in such a test is that you have to eliminate the “application” in order to test the system. Otherwise, the real application and the test application would likely interfere.
  - 黑盒性能测试：推测待测软件层的上层软件，然后用一个特定的测试应用来模拟对layer-of-interest的使用。这允许在你的应用中直接收集测试数据。在测试应用中装在有数据收集装置（探针）。一个主要的约束条件在于你需要推导出那个特定的应用程序，不然的话真实应用和测试应用会有冲突
- `Grey-box performance testing`: More application-specific measure which is capable of watching internal states of the system and can measure (probe) certain points in the system, thus generate the measurement data with minimal interference. Requires to instrument the complete application.
  - 灰度性能测试：更多的测试是基于某个具体的application的，尝试检测系统内部状态，能够使用探针检测到系统中具体的某个操作，进一步生成测试数据，同时尽可能减少对系统的中断。这种策略需要给完成应用程序打桩。

```

             Probe      Probe
             +            +
             |            |
    +--------|------------|-------+     +-----------------------------+
    |        |            |       |     |                             |
    |     +--|------------|-+     |     |                             |
    |     |  v            v |     |     |        - latency   <--------------+ Probe
    |     |                 |     |     |        - throughput<--------------+ Probe
    |     |     Function    |     |     |        - memory    <--------------+ Probe
    |     |                 |     |     |        - power     <--------------+ Probe
    |     +-----------------+     |     |                             |
    |      System under test      |     |       System under test     |
    +-----------------------------+     +-----------------------------+


              Functional                            Non-functional


    +-------------+                     +----------------------------+
    | Test App.   |                     |  +-----------------------+ |
    |  + +  +  +  |                     |  |    Application        | |
    +--|-|--|--|--+---------------+     |  |                   <------------+ Probe
    |  | |  |  |                  |     |  +-----------------------+ |
    |  v v  v  v                  |     |                            |
    |     Probes                  |     |                      <------------+ Probe
    |                             |     |                            |
    |       System under test     |     |   System under test        |
    |                             |     |                      <------------+ Probe
    |                             |     |                            |
    |                             |     |                            |
    +-----------------------------+     +----------------------------+


             Black-Box                            Grey-box
```

有没有白盒性能测试呢，没有

## Tracing跟踪

Tracing and benchmarking can be defined as follows:

- `tracing`: logging (partial) execution information while the system is running. `tracing` is used to understand what goes on in a running software system.
- `benchmarking`: a method of comparing the performance of various systems by running a common test.

From these definitions, inherently one can determine that both benchmarking and tracing are connected in the sense that the test/benchmark will use a series of measurements for comparison. These measurements will come from tracing probes or other logging mechanisms. In other words, tracing will collect data that will then be fed into a benchmark program for comparison.

## 相关工作

There're various past efforts in the robotics community to benchmark ROS robotic systems. The following are some of the most representative ones:

- `ros2_benchmarking <https://github.com/piappl/ros2_benchmarking/>`_ : First implementation available for ROS 2, aimed to provide a framework to compare ROS and ROS 2 communications.
- `performance_test <https://gitlab.com/ApexAI/performance_test/>`_: Tool is designed to measure inter and intra-process communications. Runs at least one publisher and at least one subscriber, each one in one independent thread or process and records different performance metrics. It also provides a way to generate a report with the results through a different package.
- `reference_system <https://github.com/ros-realtime/reference-system/>`_: Tool designed to provide a framework for creating reference systems that can represent real-world distributed systems in order to more fairly compare various configurations of each system (e.g. measuring performance of different ROS 2 executors). It also provides a way to generate reports as well.
- `ros2-performance <https://github.com/irobot-ros/ros2-performance/>`_: Another framework to evaluate ROS communications and inspired on `performance_test`. There's a decent rationale in the form of a proposal, a good evaluation of prior work and a well documented set of experiments.
- `system_metrics_collector <https://github.com/ros-tooling/system_metrics_collector/>`*: A lightweight and *real-time* metrics collector for ROS 2. Automatically collects and aggregates *CPU* % used and *memory* % performance metrics used by both system and ROS 2 processes. Data is aggregated in order to provide constant time average, min, max, sample count, and standard deviation values for each collected metric. _**Deprecated***.
- `ros2_latency_evaluation <https://github.com/Barkhausen-Institut/ros2_latency_evaluation/>`_: A tool to benchmarking performance of a ROS 2 Node system in separate processses (initially focused on both inter-process and intra-process interactions, later focused). Forked from `ros2-performance`.
- `ros2_timer_latency_measurement <https://github.com/hsgwa/ros2_timer_latency_measurement/>`_:  A minimal *real-time safe* testing utility for measuring jitter and latency.  Measures nanosleep latency between ROS child threads and latency of timer callbacks (also within ROS) across two different Linux kernel setups (`vanilla` and a `RT_PREEMPT`` patched kernel).
- `buildfarm_perf_tests <https://github.com/ros2/buildfarm_perf_tests/>`_: Tests which run regularly on the official ROS 2 buildfarm. Formally, extends `performance_test` with additional tests that measure additional metrics including CPU usage, memory, resident anonymous memory or virtual memory.
- `ros2_tracing <https://github.com/ros2/ros2_tracing>`*: Tracing tools for ROS 2 built upon LTTng which allow collecting runtime execution information on real-time distributed systems, using the low-overhead LTTng tracer. Performance evaluation can be scripted out of the data collected from all these trace points. The ROS 2 core layers (*`_rmw_`_,_`_rcl_`_,_`_rclcpp_`_) have been instrumented with LTTng probes which allow collecting information of ROS 2 targets without the need to modify the ROS 2 core code (*system under test)*. There are various publications available about_`_ros2_tracing_`_[3]_ [4]_ and it is used actively to benchmark ROS 2 **in real scenarios**, including perception and mapping [5]_, hardware acceleration [6]_[7]_ or self-driving mobility [8]_.

## 工业标准

There are no globally accepted industry standards for benchmarking robotic systems. The closest initiative to a standardization effort in robotics is the European H2020 Project **EUROBENCH** which aimed at creating the first benchmarking framework for robotic systems in Europe focusing on bipedal locomotion. The project has been completed in 2022 and the results are available in [9]_. The project has **been a great success** and has been used to benchmark a wide range of bidepal robotic systems throughout experiments however there're no public plans to escalate the project to other types of robots, nor the tools have been used elsewhere.<br />When looking at other related areas to robotics we find the MLPerf Inference and MLCommons initiatives which are the closest to what we are trying to achieve in ROS 2. The **MLPerf Inference** is an open source project that aims to define a common set of benchmarks for evaluating the performance of machine learning inference engines. The **MLCommons** is an open source project that aims to define a common set of benchmarks for evaluating the performance of machine learning models. Both projects have been very successful and are widely used in the industry. The MLPerf Inference project has been completed in 2021 and the results inference benchmarks available in [10]_. The MLCommons project has become an industry standard in Machine Learning and the results publicly disclosed in [11]_.

## 测试指标

Robots are deterministic machines and their performance should be understood by considering metrics such as the following:

- **latency**: time between the start and the completion of a task.
  - 延迟：任务执行开始到结束的时间间隔
- **system reaction time**: time between receipt of an external stimulus and the beginning of the system's actions (for example, time between an obstacle sensor firing and the first velocity command taking this into account)
  - 系统响应时间，系统从收到激励到开始执行动作的时间间隔
- **software system reaction time**: time between when an external stimulus is received by the robot's software and when the corresponding action has been executed by the software. This is usually the more directly measurable version of system reaction time.
  - 软件系统响应时间，软件系统从收到激励到开始执行动作的时间间隔
- **message latency**: Time between publishing a message and invocation of the corresponding callback on the receiver side
  - 消息时延，消息发布到callback调用之间的时间间隔
  - 包括消息传输时延和callback调度时延（execution latency）
- **execution latency**: Time between when an event leading to an execution (such as a timer firing, or a message being received) occurs, and when the corresponding callback is called
  - 执行时延，也就是消息收到到callback执行到的时间间隔
- **bandwidth or throughput**: the total amount of work done in a given time for a task. When measuring bandwidth or throughput in a ROS 2 system Messages per second is interesting, and the total number of bytes per second is interesting too.
  - 单位时间内处理的数据量
- **power**: the electrical energy per unit of time consumed while executing a given task.
  - 功耗
- **performance-per-watt**: total amount of work (generally *bandwidth* or *throughput*) that can be delivered for every watt of power consumed.
  - 能效，单位功耗的性能
- **memory**: the amount of short-term data (not to be confused with storage) required while executing a given task.
  - 内存

These metrics can help determine performance characteristics of a robotic system. Of most relevance for robotic systems we often encounter the *real-time* and *determinism* characteristics defined as follows:

- **real-time**: ability of completing a task's computations while meeting time deadlines
- **determinism**: that the same external or internal event leads to the same system behavior, with executions in the same order, each time.

For example, a robotic system may be able to perform a task in a short amount of time (*low latency*), but it may not be able to do it in *real-time*. In this case, the system would be considered to be *non-real-time* given the time deadlines imposed. On the other hand, a robotic system may be able to perform a task in *real-time*, but it may not be able to do it in a short amount of time. In this case, the system would be considered to be *non-interactive*. Finally, a robotic system may be able to perform a task in real-time and in a short amount of time, but it may consume a lot of *power*. In this case, the system would be considered to be *non-energy-efficient*.<br />In another example, a robotic system that can perform a task in 1 second with a power consumption of `2W` is twice as fast (*latency*) as another robotic system that can perform the same task in 2 seconds with a power consumption of `0.5W`. However, the second robotic system is twice as efficient as the first one. In this case, the solution that requires less power would be the best option from an energy efficiency perspective (with a higher *performance-per-watt*). Similarly, a robotic system that has a high bandwidth but consumes a lot of energy might not be the best option for a mobile robot that must operate for a long time on a battery.<br />Therefore, it is important to consider different of these metrics when benchmarking a robotic system. The metrics presented in this REP are intended to be used as a guideline, and should be adapted to the specific needs of a robot.

# ROS2的基准测试策略

In this REP, we **recommend adopting a grey-box and non-functional benchmarking approach** to measure performance and allow to evaluate ROS 2 individual nodes as well as complete computational graphs. To realize it in an architecture-neutral, representative, and reproducible manner, we also recommend using the Linux Tracing Toolkit next generation (`LTTng <https://lttng.org/>`_) through the `ros2_tracing` project, which leverages probes already inserted in the ROS 2 core layers and tools to facilitate benchmarking ROS 2 abstractions.<br />The following diagram shows the proposed methodology for benchmarking performance in ROS 2 which consists of 3 steps:

```


                                                +--------------+
                    +----------------+  rebuild |              |
                    |                +---------->              |
  start  +----------> 1. trace graph |          | 2. benchmark +----------> 3. report
                    |                |          |              |
                    +----+------^--^-+          |              |
                         |      |  |            +-------+------+
                         |      |  |                    |
                         +------+  |                    |
                           LTTng   +--------------------+
                                       re-instrument

```

1. instrument both the target ROS 2 abstraction/application using `LTTng <https://lttng.org/>`_. Refer to_`_ros2_tracing <https://github.com/ros2/ros2_tracing>_` for tools, documentation and ROS 2 core layers tracepoints;
2. trace and benchmark the ROS 2 application;
3. create performance reports with the results of the benchmarking.

# 参考实现

The reader is referred to `ros2_tracing <[https://github.com/ros2/ros2_tracing>`_](<https://github.com/ros2/ros2_tracing>>`_) and`LTTng <[https://lttng.org/>`_](<https://lttng.org/>>`_) to familiarize herself with the tools and the methodology of collecting and analyzing performance data. In addition, [3]_and [4]_ present comprehensive descriptions of the `ros2_tracing <[https://github.com/ros2/ros2_tracing>`_](https://github.com/ros2/ros2_tracing>`_) tools and the`LTTng <[https://lttng.org/>`_](https://lttng.org/>`_) infrastructure.<br />Reference implementations complying with the recommendations of this REP can be found in literature for applications like perception and mapping [5]_, hardware acceleration [6]_ [7]_or self-driving mobility [8]_. A particular example of interest for the reader is the instrumentation of the `image_pipeline <[<https://github.com/ros-perception/image_pipeline/tree/humble/>>`_](https://github.com/ros-perception/image_pipeline/tree/humble/>`_) ROS 2 package [12]_, which is a set of nodes for processing image data in ROS 2. The`image_pipeline <[https://github.com/ros-perception/image_pipeline/tree/humble/>`_](https://github.com/ros-perception/image_pipeline/tree/humble/>`_) package has been instrumented with LTTng probes available in the ROS 2 `Humble` release, which results in various perception Components (e.g. `RectifyNode <[https://github.com/ros-perception/image_pipeline/blob/ros2/image_proc/src/rectify.cpp#L82/>`_](<https://github.com/ros-perception/image_pipeline/blob/ros2/image_proc/src/rectify.cpp#L82/>>`_) *Component*) leveraging intrumentation which if enabled, can help trace the computational graph information flow of a ROS 2 application using such Component. The results of benchmarking the performance of`image_pipeline <[https://github.com/ros-perception/image_pipeline/tree/humble/>`_](https://github.com/ros-perception/image_pipeline/tree/humble/>`_) are available in [13]_and launch scripts to both trace and analyze perception graphs available in [14]_.

# 参考

[https://github.com/ros-infrastructure/rep/pull/364/files?w=1](https://github.com/ros-infrastructure/rep/pull/364/files?w=1)<br />.. [1] Hennessy, J. L., & Patterson, D. A. (2011). Computer architecture: a quantitative approach. Elsevier.<br />.. [2] Pemmaiah, A., Pangercic, D., Aggarwal, D., Neumann, K., & Marcey, K. (2019) "Performance Testing in ROS 2".<br />   [https://drive.google.com/file/d/15nX80RK6aS8abZvQAOnMNUEgh7px9V5S/view](https://drive.google.com/file/d/15nX80RK6aS8abZvQAOnMNUEgh7px9V5S/view)<br />.. [3] Bédard, C., Lütkebohle, I., & Dagenais, M. (2022). ros2_tracing: Multipurpose Low-Overhead Framework for Real-Time Tracing of ROS 2. IEEE Robotics and Automation Letters, 7(3), 6511-6518.<br />.. [4] Bédard, C., Lajoie, P. Y., Beltrame, G., & Dagenais, M. (2022). Message Flow Analysis with Complex Causal Links for Distributed ROS 2 Systems. arXiv preprint arXiv:2204.10208.<br />.. [5] Lajoie, P. Y., Bédard, C., & Beltrame, G. (2022). Analyze, Debug, Optimize: Real-Time Tracing for Perception and Mapping Systems in ROS 2. arXiv preprint arXiv:2204.11778.<br />.. [6] Mayoral-Vilches, V., Neuman, S. M., Plancher, B., & Reddi, V. J. (2022). "RobotCore: An Open Architecture for Hardware Acceleration in ROS 2".<br />   [https://arxiv.org/pdf/2205.03929.pdf](https://arxiv.org/pdf/2205.03929.pdf)<br />.. [7] Mayoral-Vilches, V. (2021). "Kria Robotics Stack".<br />   [https://www.xilinx.com/content/dam/xilinx/support/documentation/white_papers/wp540-kria-robotics-stack.pdf](https://www.xilinx.com/content/dam/xilinx/support/documentation/white_papers/wp540-kria-robotics-stack.pdf)<br />.. [8] Li, Z., Hasegawa, A., & Azumi, T. (2022). Autoware_Perf: A tracing and performance analysis framework for ROS 2 applications. Journal of Systems Architecture, 123, 102341.<br />.. [9] European robotic framework for bipedal locomotion benchmarking<br />    [https://eurobench2020.eu/](https://eurobench2020.eu/)<br />.. [10] MLPerf™ inference benchmarks<br />    [https://github.com/mlcommons/inference](https://github.com/mlcommons/inference)<br />.. [11] MLCommons<br />    [https://mlcommons.org/en/](https://mlcommons.org/en/)<br />.. [12] image_pipeline ROS 2 package. An image processing pipeline for ROS. `Humble` branch.<br />    [https://github.com/ros-perception/image_pipeline/tree/humble](https://github.com/ros-perception/image_pipeline/tree/humble)<br />.. [13] Case study: accelerating ROS 2 perception<br />    [https://github.com/ros-acceleration/community/issues/20#issuecomment-1047570391](https://github.com/ros-acceleration/community/issues/20#issuecomment-1047570391)<br />.. [14] acceleration_examples. ROS 2 package examples demonstrating the use of hardware acceleration. <br />    [https://github.com/ros-acceleration/acceleration_examples](https://github.com/ros-acceleration/acceleration_examples)
