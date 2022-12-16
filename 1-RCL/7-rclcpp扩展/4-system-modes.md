# 前言

- 动态计算：初始化，正常运行，降级，暂停，重启，失效

  - 可均衡资源使用

- 分段计算：更好支持实时性配置

  - 运行阶段配置实时能力

- 扩展协作能力：允许节点间状态协作

  - 我们需要合适的节点间管理来实现更层次的复杂行为，比如失效恢复

  - 为了方便管理，我们将节点分层，以树的形式管理

# 概述：新的节点间协作机制

分布式系统同样面临节点间关系管理问题。

为什么需要节点间关系管理？

业务就是通过节点间关系来实现的。

现代的机器人软件架构是分层的，SLAM算法，视觉算法，运动规划算法一般分类在skill layer或者functional layer。当执行复杂任务时，这些能力被更上层调用协调，称作executive layer或者planning layer，或许有别的名字叫做task and mission layer或者deliberation layer。

我们发现在deliberation layer有三种不同但是很类似的组件协作：

1. **Task Handling**: Orchestration of the actual task, the *straight-forward*, *error-free* flow.

    1. 任务处理

1. **Contingency Handling**: Handling of task-specific contingencies, e.g., expectable retries and failure attempts, obstacles, low battery.

    1. 可预测错误处理

1. **System Error Handling**: Handling of exceptions, e.g., sensor/actuator failures.

    1. 不可预测错误处理，异常处理

实现机器人能力的skill layer的机制一般有services, actions，参数重配置，设置值，激活/去激活组件等等。我们将其分类为两种，面向方法的和面向系统的。

*function-oriented calls* to a running skill component (set values, action queries, etc.) and *system-oriented calls* to individual or multiple components (switching between component modes, restart, shutdown, etc.).

面向方法的调用通过改变配置值，执行特定动作。面向系统的方法通过切换节点模式，重启，关闭等方法实现。

Analogously, we distinguish between *function-oriented notifications* from the skill layer in form a feedback on long-running service calls, messages on relevant events in the environment, etc. and *system-oriented notifications* about component failures, hardware errors, etc.

面向方法的方式，通知服务和动作的调用结果，面向系统的方法通知组件失效，硬件错误等状态。

![](https://tcs.teambition.net/storage/31297aa54d8264f6f79e4f032b1665442bf9?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTU1OCwiaWF0IjoxNjcxMTg2NzU4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk3YWE1NGQ4MjY0ZjZmNzllNGYwMzJiMTY2NTQ0MmJmOSJ9.TAiFxILvYX3ftq_Y7MCIzSuAfzV8L2EqBt9jXj_LUUc&download=image.png "")

我们又发现，做上述能力实现过程中会使得节点进入一个更高层次的复杂控制流程。我们可以将这些复杂度简化并抽象为若干 system-oriented calls and notifications.

Therefore, our **goal** within this work is to provide suitable abstractions and framework functions for (1.) system runtime configuration and (2.) system error and contingency diagnosis, to reduce the effort for the application developer of designing and implementing the task, contingency and error handling.

所以，我们目标是使用一种合适的抽象来做系统运行时配置以及系统错误诊断，来减少开发者开发任务处理的工作量。

![](https://tcs.teambition.net/storage/3129b2a1f53551d6125cd16c02f114cfaba0?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTU1OCwiaWF0IjoxNjcxMTg2NzU4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjliMmExZjUzNTUxZDYxMjVjZDE2YzAyZjExNGNmYWJhMCJ9.vueGgKVT1rYKDyppf0JD5bRLX6DVYPsdxSzDhjG7ik0&download=image.png "")

The main features of the approach are (detailed in the remainder):

1. *Extended Lifecycle*: Extensible concept to specify the runtime states of components, i.e ROS 2 lifecycle nodes.

    1. 可扩展的生命周期

1. *System Hierarchy and Modes*: Modeling approach for specifying a ROS system in terms of its system hierarchy and *system modes*, i.e. different (sub-)system configurations.

    1. 层次和模式化系统

1. *Mode Manager*: A module to manage and change the system runtime configuration.

    1. 模式管理员

1. *Mode Inference*: A module for deriving the entire system state and mode from observable system information, i.e. states, modes, and parameters of its components.

    1. 模式推理

1. *Error Handling*: Lightweight concept for specifying an error handling and recovery mechanism.

    1. 错误处理和恢复机制

# 背景知识：lifecycle机制

节点间协作机制依赖lifecycle基础，lifecycle的基本目标是允许ROS节点的状态控制。

[lifecycle-node](https://thoughts.teambition.com/share/61376613d3c8920045eac3b4#title=lifecycle-node)

# 需求

(1.) system runtime configuration and (2.) system error and contingency diagnosis.

## 系统运行时配置

### 模型

- 子系统的层次化建模

  - 允许子系统包含多个模块（ROS nodes, micro-ROS nodes），形成层次化模型，或者至少两阶段模型（模块，子模块）

  - 诊断/监控是否也可以层次化建模呢？

  - 一些机器人框架具有宏包或者层次化包。我们希望更灵活的方式来独立定义各个子系统。比如，即将所有的驱动包看成整体，使用同样的调度策略，将相机驱动和障碍物识别作为一个系统进行统一配置。

- 系统，子系统，组件模式的建模

  - 基于系统，子系统，组件的定义，系统应该形成运行时配置的概念，来定义每个元素的模式

- 标准但是可扩展的组件运行生命周期

  - 组件运行生命周期（component mode/state）应该基于OMG标准以及现行ROS2标准

- 对子系统内的错误，错误传播和严重等级进行建模

  - Describe causal dependencies between components of a (sub)system with regard to errors. (Sources of inspiration are Fault-Tree Analysis and various academic works.)

  - 描述子系统错误的传播方式和路径

- 将组件状态/模式和组件的参数配置统一为一个模型

  - 使用ROS的动态参数重配置机制

  - 可能会造成组件声明周期和组件参数的耦合？

- **Timing/causality-aware switching between modes**

  - 两种模式：1）按照指定的顺序重配置2）同时对所有组件进行重配置

### 实现

- 系统，子系统，组件模式的建模

  - Strive for lightweight approach by a simple configuration file or even use of C++ API.

  - Should be well manageable (diffable, mergeable) with Git.

- **API primitives for defining preconditions and invariants with regard to system configuration**

### 运行时/执行

- 分布式系统状态在不同设备间的传递

  - 系统应该支持子系统的状态变更在所有设备中的感知一致性

  - It shall also consider that a computing device may reconfigure its sub-systems or components (in a limited scope) independent of the sub-systems and components other computing devices.

## 系统错误以及诊断

系统应该允许，提供API来检测系统典型的错误和偶然故障

1. **Monitors for communication layers**

    1. Check for receive rates, latencies, ...

1. **Operating system monitors**

1. **Templates and APIs for hardware monitors**

    1. Basic blocks for GPIOs (voltage, current)

    1. Basic blocks for connected HW modules (via SPI, I²C)

1. **Templates and APIs for monitoring of functional properties**

    1. No need of status messages here, but use of function-oriented topics

    1. Building blocks/patterns for analysis of time-series (with a sliding window) for bounds, for delays, ...

# ROS2 系统模式

system modes假设系统组件具有生命周期的，它为节点上添加了系统和子系统的层次化概念，为节点的参数配置提供了一个模式的概念，不同模式下，参数具有不同的值。

## system modes package

该包以ROS2包的形式呈现，作为库，提供模式推理，模式管理，模式监视的功能。

## Hierarchal System Modeling

We assume the system to comprise of loosely-coupled - potentially distributed - components with a runtime lifecycle; hereinafter referred to as '*nodes*'. We call semantic grouping of these nodes a *(sub-)system*. We assume that these (sub-)systems can again be hierarchically grouped into further (sub-)systems, see [system-of-systems](https://en.wikipedia.org/wiki/System_of_systems). All nodes and (sub-)systems that belong to a certain (sub-)system are referred to as *parts* of this (sub-)system.

The introduced notion of (sub-)systems does not refer to a concrete software entity, but rather a *virtual* abstraction that allows efficient and consistent handling of node groups.

子系统的概念是虚拟的，包含若干个节点

*Note:* In a first stage of this concept, we assume that the entire system can be specified up-front. Later revisions of this concept might have to take care of changing systems, i.e. further nodes and/or (sub-)systems joining at runtime.

## Lifecycle

Within this package, we assume that nodes are [ROS 2 Lifecycle Nodes](http://design.ros2.org/articles/node_lifecycle.html). We extend the ROS 2 default lifecycle by the following aspects:

1. We introduce **modes** that are specializations of the *ACTIVE* state, see [System Modes](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#system-modes).

    1. 在ACTIVE状态下扩展了模式的概念

1. We introduce an optional transition from *ACTIVE* to *activating* to allow changing modes without deactivating the node.

    1. 在ACTIVE状态下模式的迁移，不会去激活节点

We additionally establish the same lifecycle for the (sub-)systems introduced [above](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#hierarchical-system-modeling). Hence, all *parts* of a system can be assumed to have the same lifecycle.

![](https://tcs.teambition.net/storage/312f9f1f88e18251bf9c153ce1b5e9d4d1b9?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTU1OCwiaWF0IjoxNjcxMTg2NzU4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmY5ZjFmODhlMTgyNTFiZjljMTUzY2UxYjVlOWQ0ZDFiOSJ9.SIGnTjlfLR1B9BHbaDjS0kartkXkB8JAsFfy1sg2C3Y&download=image.png "")

## System Modes

System modes extend the *ACTIVE* state of the ROS 2 lifecycle and allow to specify different configurations of nodes and (sub-)systems:

- **Modes of nodes** consist of parameter values.

  - 模式是包含参数的值的一个配置

- **Modes of (sub-)system)** consist of modes of their *parts*.

  - 子系统的模式包含子系统内部组件的模式

For example, a node representing an actuator might provide different modes that specify certain maximum speed or maximum torque values. An actuation sub-system, grouping several actuator nodes, might provide modes that activate/deactivate certain contained actuator nodes and/or change their modes based on its own modes.

Both, the [system hierarchy](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#hierarchical-system-modeling) as well as the system modes are specified in a system modes and hierarchy model file (SHM file, yaml format) that can be parsed by the [mode inference](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#mode-inference) mechanism. The SMH file adheres to the following format *(curly brackets indicate placeholders, square brackets indicate optional parts, ellipses indicate repeatability)*:

定义了系统和子系统（节点）的模式。

```yaml
{system}:
  ros__parameters:
    type: system
    parts:
      {node}
      […]
    modes:
      __DEFAULT__:
        {node}: {state}[.{MODE}]
        […]
      {MODE}:
        {node}: {state}[.{MODE}]
        […]
      […]
[…]

{node}:
  ros__parameters:
    type: node
    modes:
      __DEFAULT__:
        ros__parameters:
          {parameter}: {value}
          […]
      {MODE}:
        ros__parameters:
          {parameter}: {value}
          […]
      […]
[…]
```

The [system_modes_examples](https://github.com/micro-ROS/system_modes/blob/master/system_modes_examples) package shows a simple example consisting of modes for one system and two nodes. The model file of the example can be found [here](https://github.com/micro-ROS/system_modes/blob/master/system_modes_examples/example_modes.yaml).

## Mode Inference

Since the introduced (sub-)systems are not concrete software entities, their state and mode has to be *inferred* from the states and modes of their parts. This inference mechanism is part of the system modes library and is used by the [mode manager](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#mode_manager) and [mode monitor](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#mode_monitor) that are also included in this package. We can show that system states and modes can be deterministically inferred under the following conditions:

1. Nodes can be asked for their state, mode, and parameters This is true, since the lifecycle nodes provide the according lifecycle state service (GetState) and the [mode manager](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#mode_manager) provides the according mode service (GetMode).

1. *Target* states and modes are known Before attempting a state or mode change for a system or node, the [mode manager](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#mode_manager) publishes information about the request. The according topics might need to be *latched* in order to allow nodes to do the inference after joining a running system.

## Mode Observer

Additionally, the library comprises a *Mode Observer* that serves as a local cache of states and modes of all observed system parts. The mode observer will try to obtain the current state and mode initially via sevrice calls (GetState, GetMode) and subsequently monitors according transitions events and mode events.

本地缓存，定期调用服务，更新状态到本地

The mode observer is supposed to be instantiated within a ROS 2 node to have states and modes available locally for fast access, see this exemplary [ModeObserverNode](https://github.com/micro-ROS/system_modes/blob/master/system_modes/test/launchtest/modes_observer_test_node.cpp) used for testing.

## Mode Manager

The mode manager is a ROS node that accepts an SHM file (see [above](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#system-modes)) as command line parameter. It parses the SHM file and creates the according services, publishers, and subscribers to manage the system and its modes.

- For (sub-)systems, it mirrors the lifecycle services that are available for ROS 2 lifecycle nodes, i.e.

  - `/{system}/get_available_states` - lifecycle_msgs/GetAvailableStates

  - `/{system}/get_state` - lifecycle_msgs/GetState

  - `/{system}/change_state` - lifecycle_msgs/ChangeState

- For (sub-)systems *and* nodes, it provides similar services for modes, i.e.

  - `/{system_or_node}/get_available_modes` - [system_modes/GetAvailableModes](https://github.com/micro-ROS/system_modes/blob/master/system_modes/srv/GetAvailableModes.srv)

  - `/{system_or_node}/get_mode` - [system_modes/GetMode](https://github.com/micro-ROS/system_modes/blob/master/system_modes/srv/GetMode.srv)

  - `/{system_or_node}/change_mode` - [system_modes/ChangeMode](https://github.com/micro-ROS/system_modes/blob/master/system_modes/srv/ChangeMode.srv)

- Service calls to these services publish information on the requested state change or mode change before attempting them. These are published on the following topics:

  - `/{system_or_node}/transition_request_info` - lifecycle_msgs/TransitionEvent

  - `/{system_or_node}/mode_request_info` - [system_modes/ModeEvent](https://github.com/micro-ROS/system_modes/blob/master/system_modes/msg/ModeEvent.msg)

Running the manager: $ `ros2 launch system_modes mode_manager.launch.py modelfile:=[path/to/modelfile.yaml]`

## Mode Monitor

The mode monitor is a ROS node that accepts an SHM file (see [above](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#system-modes)) as command line parameter. It continuously monitors and displays the entire system state and mode based on the mode inference introduced [above](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#mode-inference). It monitors the following topics:

- `/{system_or_node}/transition_request_info` for all known (sub-)systems and nodes from the model file to monitor their target states

- `/{system_or_node}/mode_request_info` for all known (sub-)systems and nodes from the model file to monitor their target modes

- `/parameter_events` to infer the current modes for all known nodes based on their parameter values

![mode_monitor](https://github.com/micro-ROS/system_modes/raw/master/system_modes_examples/doc/screenshot-monitor.png "Screenshot of the mode monitor from system_modes_examples")

Running the monitor: $ `ros2 launch system_modes mode_monitor.launch.py modelfile:=[path/to/modelfile.yaml]`

## Error Handling and Rules

If the *actual* state/mode of the system or any of its parts diverges from the *target* state/mode, we define rules that try to bring the system back to a valid *target* state/mode, e.g., a degraded mode. Rules work in a bottom-up manner, i.e. starting from correcting nodes before sub-systems before systems. Rules are basically defined in the following way:

```text
if:
 system.target == {target state/mode} && system.actual != {target state/mode} && part.actual == {specific state/mode}
then:
 system.target := {specific state/mode}

```

if *actual* state/mode and *target* state/mode diverge, but there is no rule for this exact situation, the bottom-up rules will just try to return the system/part to its *target* state/mode. *Potentiall dangereous, to be discussed:* what's happening, if the system is already on its way. E.g., a system or part was just commanded to transition to *ACTIVE.foo*, but is currently *activating* (so doing everything right). In this case we have to avoid that the bottom-up rules will trigger.

## How to Apply

When designing the hierarchy of your system, try to group parts semantically, e.g., everything that belongs to *perception* or *navigation*. You want to group those parts of a system that are often jointly managed (initialized, shutdown, configured). Hierarchies don't necessarily need to be designed in one big tree, but can form several parallel trees.

尝试根据业务特点将各个节点分门别类，经常一起管理的节点也要放在一起，整个系统要像树一样，也可以是两个并排的树

When designing system modes for your system, try to focus on platform-specific aspects (so those that are generally present) rather than aspects specific to a certain application. Good examples are *degraded* and *performance* modes of the platform, bad examples are system modes to encode the current number of (grasping) re-tries.

设计系统模式时，关注平台相关的特性，而不是应用相关的特性，比较好的例子：*degraded* and *performance* modes of the platform，差的例子：current number of (grasping) re-tries.

定位模式设计：默认，高性能，低能耗

规划模式设计：默认，避让（靠边缓行），保守（降级），进攻（加速）

Also, do not model any "read-only" system modes, e.g., modes that discretize/encode a read-only internal state. An example for such a mode specification to avoid is *low energy* and *full*, discretizing the charging level of a battery component. The System Modes concept assumes that the activatability of a system mode of a given node or subsystem should depend only on the states and modes of the other nodes and subsystems (and on the higher-level task executed by some deliberation layer). Note that the same applies to the ROS 2 node lifecycle states (*Unconfigured*, *Inactive*, etc.). The only exception is the *ErrorProcessing* state, which can be entered autonomously by the node itself. Within the mode inference, if a node performs a transition to *ErrorProcessing*, this is automatically propagated upwards as inferred state along the hierarchy. It is up to the deliberation layer to handle the failure of this node or subsystem.

不要暴露任务只读模式，模式是服务于配置的。比如显示点亮的low energy, full，这些只读的状态信息应该通过诊断数据发送出去。

# 失效模式示例：microros中的应用

## 前提

使用该组件有两个前提，第一个是建模，第二个建立诊断机制

- 建模

  - 建立系统和子系统的层次化模型

  - 建立每个子系统的状态，状态切换的触发条件（参数配置，时间等）

  - 在多个子系统间共享状态

- 系统诊断机制

  - 诊断信息的沟通渠道

  - 操作系统诊断

  - 硬件的诊断

  - 功能的诊断

    - 不一定需要status msg，面向功能的话题也行的。

## 扩展状态

在active状态中建立子状态，在rclc_lifecycle中做了实现

## 系统层次和模式

建模成一个SMH模型文件，表示系统和子系统的状态

## 模式推理

根据每个节点的lifecycle states，模式和参数配置推理出系统的总体模式。模式管理器解析SMH文件，订阅状态改变事件，模式切换事件和参数改变事件。

根据lifecycle状态改变事件，管理器知晓所有节点的状态。根据参数改变事件，管理器知晓所有节点的参数事件，这就允许基于SMH的模式推理。

根据SMH模型，基于所有节点的状态，可以自底向上推理出一个系统的模式。

模式可以使用mode monitor来管理

## 模式管理器

在模式推理机制之上，还可以做到强制切换状态

## 错误处理和机制

如果实际状态和目标状态冲突了，则有定义好的规则将系统回滚到合理的状态。例如degraded模式，规则定义形式如下

```bash
if:
 system.target == {target state/mode} && system.actual != {target state/mode} && part.actual == {specific state/mode}
then:
 system.target := {specific state/mode}

```

*Note:* This feature is suited for simple, well-defined rules according to the depicted syntax. For more complex orchestration, integration of system modes with ontological reasoning (*metacontrol*) has been validated and successfully shown in the [MROS project](https://robmosys.eu/mros/), e.g., within a [navigation sub-system of a mobile robot](https://github.com/MROS-RobMoSys-ITP/Pilot-URJC).

# 状态设计示例：nav2中的应用

lifecycle实际上赋予了节点以基础的状态机，帮助获得确定性行为，同时也可以帮助结构化程序。

## 何种状态做何事

When a node is started, it is in the unconfigured state, only processing the node’s constructor which should **not** contain any ROS networking setup or parameter reading. By the launch system, or the supplied lifecycle manager, the nodes need to be transitioned to inactive by configuring. After, it is possible to activate the node by transitioning through the activing stage.

This state will allow the node to process information and be fully setup to run. The configuration stage, triggering the `on_configure()` method, will setup all parameters, ROS networking interfaces, and for safety systems, all dynamically allocated memory. The activation stage, triggering the `on_activate()` method, will active the ROS networking interfaces and set any states in the program to start processing information.

To shutdown, we transition into deactivating, cleaning up, shutting down and end in the finalized state. The networking interfaces are deactivated and stop processing, deallocate memory, exit cleanly, in those stages, respectively.

The lifecycle node framework is used extensively through out this project and all servers utilize it. It is best convention for all ROS systems to use lifecycle nodes if it is possible.

Within Nav2, we use a wrapper of LifecycleNodes, `nav2_util LifecycleNode`. This wrapper wraps much of the complexities of LifecycleNodes for typical applications. It also includes a `bond` connection for the lifecycle manager to ensure that after a server transitions up, it also remains active. If a server crashes, it lets the lifecycle manager know and transition down the system to prevent a critical failure.

额外定义了一个bond机制，如果server崩溃，那么整个系统都会停止。

节点间协作是通过manager来做的，这是一种集中式机制，所以要防止manager的单点失效

顺序启动, 是由launch.py来实现的，当检测到节点到达某状态，则启动下一个节点。

# 参考

[Navigation Concepts — Navigation 2 1.0.0 documentation](https://navigation.ros.org/concepts/index.html#lifecycle-nodes-and-bond)

[Eloquent to Foxy — Navigation 2 1.0.0 documentation](https://navigation.ros.org/migration/Eloquent.html#eloquent-migration)

[Lifecycle Manager — Navigation 2 1.0.0 documentation](https://navigation.ros.org/configuration/packages/configuring-lifecycle.html#parameters)

[Lifecycle and System Modes](https://micro.ros.org/docs/concepts/client_library/lifecycle_and_system_modes/)

[system_modes/requirements.md at master · micro-ROS/system_modes](https://github.com/micro-ROS/system_modes/blob/master/system_modes/doc/requirements.md)
