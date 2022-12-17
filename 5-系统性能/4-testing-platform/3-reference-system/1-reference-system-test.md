# 概述

在buildfarm和performance对ROS2的模组(executor, rmw, dds)进行单独测试之后，我们知道很多因素对ROS2节点的性能有影响

由于整个机器人系统是一个包含多个进程的分布式系统，在系统内外有大量数据在流动，且存在大量的数据依赖和时序依赖

![](https://tcs.teambition.net/storage/312h335cff3baa84e4db5897aa177545c64a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTg0NywiaWF0IjoxNjcxMjQ3MDQ3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgzMzVjZmYzYmFhODRlNGRiNTg5N2FhMTc3NTQ1YzY0YSJ9.rmbAX79spUw48FeSYpyllL-l0Tb-ezEuebnmBrv7WeM&download=image.png "")

而机器人系统的时间性能的关键指标是WCET，这个指标涉及系统整体，所以有必要对节点间的关系进行充分定义。

buildfarm，performance_test，reference-test都是基于基准测试的优化策略，对一个仿照实际系统的原型进行测试，得出结论，为现场系统优化给出启发方向。

e2e测试套件是基于现场测试的优化策略，作用是找到热点，找到优化动机。

# 解决问题

buildfarm和performance解决的是组件选型，以及单一条件下各组件的性能优劣

reference-system-test解决的是在机器人原型系统下，如何对各个节点进行综合配置，以达到性能需求

# 参考系统核心

## 定义

- A [platform](https://github.com/ros2middleware/reference-system/blob/main/reference_system/README.md#supported-platforms) is defined by:

  - Hardware (e.g. an off-the-shelf single-board computer, embedded ECU, etc.)

    - if there are multiple configurations available for such hardware, ensure it is specified

  - Operating System (OS) like RT Linux, QNX, etc. along with any special configurations made

- for simplicity and ease of benchmarking, **all nodes must run on a single process**

- a fixed number of nodes

  - each node with:

    - a fixed number of publishers and subscribers

    - a fixed *processing time* or a fixed *publishing rate*

- a fixed *message type* of fixed size to be used for every *node*

## 硬件平台

[Raspberry Pi 4B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)

## 核心组件

1. **Sensor Node**

    1. input node to system

    1. one publisher, zero subscribers

    1. publishes message cyclically at some fixed frequency

1. **Transform Node**

    1. one subscriber, one publisher

    1. starts processing for N milliseconds after a message is received

    1. publishes message after processing is complete

1. **Fusion Node**

    1. 2 subscribers, one publisher

    1. starts processing for N milliseconds after a message is received **from all** subscriptions

    1. publishes message after processing is complete

1. **Cyclic Node**

    1. N subscribers, one publisher

    1. cyclically processes all received messages since the last cycle for N milliseconds

    1. publishes message after processing is complete

1. **Command Node**

    1. prints output stats everytime a message is received

1. **Intersection Node**

    1. behaves like N transform nodes

    1. N subscribers, N publisher bundled together in one-to-one connections

    1. starts processing on connection where sample was received

    1. publishes message after processing is complete

## 性能需求KPI

# autoware参考系统

## 核心组件

- ROS2

  - Executors

    - Single Threaded

    - Static Single Threaded

    - Multithreaded

    - Callback Group

    - Prioritized

- *autoware_default_singlethreaded*: All nodes are assigned to the same single-threaded ROS executor

- *autoware_default_multithreaded*: All nodes are assigned to the same multi-threaded ROS executor

- *autoware_default_staticsinglethreaded*: Like *autoware_default_singlethreaded*, but using the `StaticSingleThreadedExecutor`.

- *autoware_default_prioritized*: Separate executors for front LiDAR nodes, rear LiDAR nodes fusion chain nodes, behavior planner, and everything else. Uses real-time priorities.

- *autoware_default_cbg*: Like *autoware_default_prioritized*, but uses the callback-group executor to remove the non-hotpath subscription in EuclideanClusterDetector from the executor for fusion chain nodes.

## 原型系统

![](https://tcs.teambition.net/storage/312h6cf709a5243a02bddaceb4dd425e9aee?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTg0NywiaWF0IjoxNjcxMjQ3MDQ3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg2Y2Y3MDlhNTI0M2EwMmJkZGFjZWI0ZGQ0MjVlOWFlZSJ9.mrwC_NVx89f1ljzpK3P6efT7L75c2j3vMOPuXBxdbOE&download=image.png "")

1. **Sensor Nodes**

    1. all sensor node have a publishing rate (cycle time) of **100 milliseconds**

    1. all sensor nodes publish the same message type

    1. total of **5 sensor nodes**:

        1. Front LiDAR Driver

        1. Rear LiDAR Driver

        1. Point Cloud Map

        1. Visualizer

        1. Lanelet2Map

1. **Transform Nodes**

    1. all transform nodes have one subscriber and one publisher

    1. all transform nodes start processing after a message is received

    1. publishes message after processing is complete

    1. total of **10 transform nodes:**

        1. Front Points Transformer

        1. Rear Points Transformer

        1. Voxel Grid Downsampler

        1. Point Cloud Map Loader

        1. Ray Ground Filter

        1. Object Collision Estimator

        1. MPC Controller

        1. Parking Planner

        1. Lane Planner

1. **Fusion Nodes**

    1. all fusion nodes have **two subscribers** and one publisher for this reference system

    1. all fusion nodes start processing after a message is received **from all** subscriptions

    1. all fusion nodes have a max input time difference between the first input received and last input received before publishing of **9999** seconds

    1. publishes message after processing is complete

    1. total of **5 fusion nodes:**

        1. Point Cloud Fusion

        1. NDT Localizer

        1. Vehicle Interface

        1. Lanelet2 Global Planner

        1. Lanelet 2 Map Loader

1. **Cyclic Nodes**

    1. for this reference system there is only **1 cyclic node**

    1. this cyclic node has **6 subscribers**and one publisher

    1. this cyclic node starts processing after a message is received **from any** single subscription

    1. publishes message after processing is complete

1. **Command Nodes**

    1. all command nodes have **1 subscriber** and zero publishers

    1. all command nodes prints out the final latency statistics after a message is received on the specified topic

    1. total of **2 command nodes:**

        1. VehicleDBWSystem

        1. IntersectionOutput

1. **Intersection Nodes**

    1. for this reference system there is only EuclideanClusterDetector

    1. this intersection node has **2 subscribers** and **2 publishers**

    1. publishes message after processing is complete on the correspoding publisher

## 关键指标

- **CPU utilization**

  - In general a lower CPU utilization is better since it enables you to choose a smaller CPU or have more functionality on a larger CPU for other things.

- **Memory utilization**

  - In general a lower memory utilization is better since it enables you to choose a smaller memory or have more space for other things

- **Number of dropped sensor samples in transform nodes**

  - The nodes in the reference system always use the most recent sensor data (i.e., use a history depth of 1)

    - This is a common strategy in real-world settings, as old sensor data is much less valuable than new sensor data

    - For example an image from 30 seconds ago is much less helpful while driving down the road than an image from 0.1 second ago

  - Fusion nodes drop messages during normal operation if the inputs publish with different frequencies

  - In transform nodes, however, dropped messages indicate that the transform node cannot keep up with its input

- **Number of front LiDAR samples that did not trigger an update in the Object Collision Estimator**

  - The Front and Rear LiDARs have the same publishing frequency

  - This means Object Collision Estimator should run for every LiDAR sample

  - Count number of executions of Object Collision Estimator and Front LiDAR and report any difference

- **Worst-case Latency between Front LiDAR and the Object Collision Estimator**

  - For worst-case latency we want to identify obstacles in time (i.e. early enough that we can still emergency-brake).

- **Average Latency between Front LiDAR and Object Collision Estimator**

  - For average latency we want to identify obstacles as soon as possible so we can account for the obstacle in our planning.

- **The Behavior Planner should be as cyclical as possible**

  - The desired behavior of the Behavior Planner is to be as cyclical as possible, meaning it should be executed as close to its set frequency of *100ms* as possible

  - Measure the jitter and drift over time of the timer callback

## 消息类型

A **single message type** is used for the entire reference system when generating results in order to simplify the setup as well as make it more repeatable and extensible.

This means **only one message type** from the list below is used during any given experimental run for every node in the reference system.

1. **Message4kB**

    1. reference message with a fixed size of 4 kilobytes (kB)

Other messages with different fixed sizes could be added here in the future.

When reporting results it is important to specify the message type used during the experiment, as the message size impacts the metrics.

# 参考

[https://github.com/ros-realtime/reference-system/tree/main/autoware_reference_system](https://github.com/ros-realtime/reference-system/tree/main/autoware_reference_system)
