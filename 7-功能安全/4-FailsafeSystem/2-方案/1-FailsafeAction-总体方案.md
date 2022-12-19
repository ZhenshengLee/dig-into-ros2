SafeAutonomy

# 诊断模块设计方案

## 术语

本文中的Diagnostic, healthchecker, guardian均指诊断模块。<br />软件术语和安全术语的对应

|  |  |  |
| --- | --- | --- |
| 软件术语 | 安全术语 | 备注 |
| 异常exception | 风险risk | 系统遇到的某种程度的失效 |
| 异常处理 | 风险缓解 |  |
| 异常处理措施 | 风险策略（RM） | 风险缓解的策略 |
| 异常状态 | 风险状况（RC） | 系统失效的严重程度 |

## 概念

诊断系统存在的意义是判断整个系统的健康状态，评估系统风险并且采取适当的措施缓解风险。<br />目前关于自动驾驶系统的安全法律法规和标准政策已经比较完备，主要包括ISO26262（功能安全），ISO21448（预期功能安全），ISO/TR 4804（设计，测试和确认规范），ANSI/UL4600（L4/L5），风险定义，风险评估，以及风险缓解策略应该参照上述标准，结合低速自动驾驶系统进行裁剪适配。

## 需求

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427170270-6ba1d804-e2e2-4726-ba72-41e8208e5789.png#averageHue=%23f6e9d6&clientId=u1e1088f9-c381-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ud47ba1ff&margin=%5Bobject%20Object%5D&originHeight=393&originWidth=982&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u15e20461-5ef6-4be1-918c-6af35d31d2e&title=)<br />系统监控子系统（System monitor）处在业务系统后端，服务于整个自动驾驶系统，包括以下部分：

- monitor: 数据采集（可视化）和初步风险评估
- process-monitor: 操作系统级别的数据采集，驻留在单独的进程上
- module-specific-monitor: 每个算法模块的数据采集，驻留在每一个算法进程上
- guardian: 针对初步风险评估进行风险汇总，产生和执行风险策略。
- ODD：管理高层场景，管理风险状况

各个业务算法模块的职责：

- 根据风险评估的需求，提供module-specific monitor所需的信息，如算法的迭代次数以及对应的风险等级
- 算法人员需要将算法模块化，进行失效分析，为每个模块添加详细的调试和校验代码，上报风险状态
- 针对ODD的风险状况要求，以及module-specific-monitor的指令，执行指定的风险缓解措施，如暂停运行，暂停发送消息等。
- 单体算法节点应该具备状态切换的能力，如初始化，配置，激活，去激活等状态
- 分布式算法节点应具备集中式控制能力，具备“管理器”（如planning的decision_maker）

数据流分析：

- monitor和guardian的互动
- monitor不仅具有数据采集的功能，还应执行风险缓解，或者作为风险缓解的commander。
- 而guardian的职责是风险汇总和风险评估，产生细粒度的风险缓解措施
- guardian和ODD的协作
- guardian的风险缓解措施和ODD的宏观状态管理结合起来进一步进行宏观风险状态评估
- todo

## 原理综述

自动驾驶系统的风险定义，风险评估和风险缓解<br />主要参考的中文文档为百度《自动驾驶安全第一白皮书》

- 故障树分析
- 是失效分析的一个工具
- 风险状态和风险策略
- 失效安全能力和失效降级能力
- 以安全因素对系统能力的分级描述

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427170278-6dea4dd2-75c4-4f35-9afa-1d0f76b450f3.png#averageHue=%23f5f5f5&clientId=u1e1088f9-c381-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub955af7e&margin=%5Bobject%20Object%5D&originHeight=364&originWidth=1339&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u15f2de3c-c762-4910-90d2-7c1a3c24339&title=)<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427170260-1c86fe67-7318-4265-b798-95ba8654e8b1.png#averageHue=%23adccd3&clientId=u1e1088f9-c381-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u19a6944b&margin=%5Bobject%20Object%5D&originHeight=406&originWidth=1322&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u4da80468-a721-4f2a-a4d9-e7b36c554f2&title=)<br />最终的目的是实现如下的状态切换表，保证系统安全运行能力<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427170282-0433541c-2a61-4c14-98de-484d4a565689.png#averageHue=%23e4efef&clientId=u1e1088f9-c381-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub55c2290&margin=%5Bobject%20Object%5D&originHeight=708&originWidth=1376&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u674d4513-b898-474b-a411-eda85cdc847&title=)

## Autoware的方案

current implementation review

### Autoware的数据采集和风险评估方案

autoware在1.11增加Health checker

### 总体架构

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427170272-63efef96-d0e7-4643-849a-83c0fa4b8cc8.png#averageHue=%2391c17b&clientId=u1e1088f9-c381-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uf5e58223&margin=%5Bobject%20Object%5D&originHeight=866&originWidth=1290&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u669c631a-ddd7-47a9-8dd2-00c887fff16&title=)

- health_checker
- 提供一系列的方法对算法的状态进行监控，作为插件嵌入到算法模块
- health_aggregator
- 聚合所由子模块的诊断数据，发布整体诊断数据
- health_analyzer
- 对整体诊断数据进行分析

### 算法节点和诊断协作

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427170922-21d01207-e565-48e6-bca4-3becc1cb29f0.png#averageHue=%23eae8e8&clientId=u1e1088f9-c381-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u3b546709&margin=%5Bobject%20Object%5D&originHeight=138&originWidth=457&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ubef2e14a-8afe-4cf3-9520-6c4a04007d1&title=)<br />两种协作方式

- 对于每个算法模块，链接diag库，发送定义好的状态信息
- 每个算法模块旁接一个monitor进程，专门处理该算法模块的状态，在RVIZ上做可视化

### 风险定义

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427171040-cd2dcf98-0c78-4e8b-aa0c-59fecca6e2bc.png#averageHue=%23f2f2f2&clientId=u1e1088f9-c381-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u138d7185&margin=%5Bobject%20Object%5D&originHeight=866&originWidth=820&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u7c13a2f5-8f9b-411b-bd46-9dfd6db2c6d&title=)<br />对需要提取的诊断数据进行了定义

- DiagnosticStatus诊断状态
- 使用key value键值对来对所有目标的运行状态进行风险评估
- 这个是最小状态集合
- SystemStatus
- NodeStatus节点状态
- HardwareStatus硬件状态

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427170994-26bd047c-fba9-4eef-8409-ea78df4817be.png#averageHue=%23f5f5f5&clientId=u1e1088f9-c381-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u743b3ad0&margin=%5Bobject%20Object%5D&originHeight=438&originWidth=832&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u56187af6-e1c1-4607-857d-c20daa03b05&title=)<br />autoware也对风险评估定义了规则，开发人员根据业务特点设置不同的阈值用于区分风险等级。

- 未定义
- OK
- 没有问题
- WARN
- 没有必要终止命令，但是需要记录日志以及通知用户
- ERROR
- 错误，必须终止当前命令执行，需要记录日志以及通知用户
- 如果在规定时间内没有终止命令执行，需要强制通知底层进行紧急制动
- FATAL
- 致命，需要尽快停止当前命令执行，需要记录日志以及通知用户
- 没有时间限制，尽快直接通知底层进行紧急制动

### Autoware的风险状态和风险缓解

autoware在1.13增加failsafe action framework，负责在整个系统失效时执行动作使得恢复到安全状态。<br />目前包括紧急风险状态和半紧急风险状态的缓解措施<br />软件架构设计如下<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427171048-eaa4a97c-264c-4c34-b665-28220c5368e8.png#averageHue=%23f3f3f3&clientId=u1e1088f9-c381-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u8173b506&margin=%5Bobject%20Object%5D&originHeight=981&originWidth=940&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ufb6eadcb-a1fb-41a8-aa27-9841ab58e62&title=)<br />主要包括以下模块

- emergency_handler
- 为了实施风险缓解，紧急情况下规划和驱动层面的功能由紧急控制器接管，此模块通过诊断模块穿透规划和驱动层直接进行接管。
- 目前实现了两种缓解措施： "emergency stop plan" and "semi-emergency stop plan"
- ros_observer
- 为了防止基于ROS的emergency_handler失效，还设置了看门狗机制，在这些ROS节点失效的情况下直接发送指令到驱动
- 不采用基于ROS的通信机制，而是shared_memory与底层通信
- twist_gate
- 紧急情况下接管planning的shared_memory通道
- ssc_interface
- 紧急情况下接管driver的shared_memory通道

autoware的方案: failsafe action framework, or emergency handler

- 对状态的精简health_analyzer
- 风险缓解节点emergency_handler
- 对状态的过滤分析system_status_filter

autoware的emergency handler实现比较简单，无法满足需求<br />(涉及安全的高层规划)(状态管理)

- 不是一个Manager, 也不是一个Handler, 是一个reporter or teller
- 在apollo中类似于guardian和story telling的复合体
- 在autoware中类似于decision_maker和emergency_handler的复合体
- 在nav, nav2, robot nav中类似于bt planner, locomotor等高层规划节点

### autoware底层原理
<http://wiki.ros.org/diagnostics><br /><http://wiki.ros.org/bond_core><br /><https://index.ros.org/p/diagnostic_msgs/github-ros2-common_interfaces/#foxy><br /><https://github.com/ros-safety/software_watchdogs>

## 本方案相对于autoware的不同

- emergency handler需要处理更多紧急行为，要考虑更多车辆运行状态
- emergency handler改进为状态机
- emergency handler不仅负责状态信息的过滤，还负责紧急行为的执行
- 由于紧急行为较为复杂，且紧急行为也具有任务的属性（靠边停车既可以作为紧急行为，也可以作为工作任务）
- 紧急行为的执行由各个业务模块执行，如规划，远程接管由远程接管模块负责
- 该方案缺点：emergency handler没有兜底的功能，即紧急行为的执行仍然较为复杂，还需要一个emgerency handler的emergency handler
- emergency handler已经不具备“兜底”功能，其功能更像是一个涉及安全的状态管理器，safety involved seci重命名为
- 增加状态可视化（GUI或者TUI）

## 人员分工

- 设计方案和软件架构（黎振胜）<br />- 定义状态信息结构体，定义风险等级（黎振胜）<br />- 提取状态信息（各个业务节点按方案开发）<br />- 汇聚状态信息（集成到monitor-杨东）<br />- 对汇总的状态信息分析得到风险状态，将状态对外输出（黎振胜）<br />- 对缓解指令的执行，完成状态切换由业务节点执行（停车刘健，远程接管杨东）<br />风险定义需要参考业务模型<br />风险评估需要采集数据<br />风险评估需要映射<br />风险缓解需要机制

## 开发步骤

- 以定位算法模块为例打通流程

## Apollo的方案

Apollo的相关功能统称The surveillance system<br />- **Monitor** - The surveillance system of all the modules in the vehicle including hardware.<br />- 包括硬件在内的监控系统，接受所有模块发送的monitor msg，可视化到HMI模块<br />- 当模块或者硬件发生故障时，monitor发送告警信息到Guardian, guardian进行紧急状况下的决策<br />- **Guardian** - A new safety module that performs the function of an Action Center and intervenes should Monitor detect a failure.<br />- Guardian是一个ActionCenter，基于monitor发送过来的数据进行决策<br />- 有两个功能<br />- 所有模块正常工作——Guardian允许数据流正常通过，透传到CANBus<br />- Monitor发现某模块故障——Guardian会阻断控制信号发送到CANBus，并且紧急制动车辆。<br />- **Storytelling** - A new module that isolates and manages complex scenarios, creating stories that would trigger multiple modules' actions. All other modules can subscribe to this particular module.<br />- 是一个高层场景管理器<br />### Apollo的数据采集和风险评估<br />主要由monitor完成，monitor的实现细节<br />todo

### Apollo的风险状态和风险策略

主要由Guardian完成<br />目前也有两种紧急状态：缓慢停车和立即停车<br />Guardian的紧急停车机制：<br />- 如果超声波雷达没有检测到障碍物，则缓慢停车<br />- 如果超声波雷达没有响应，则立即停车<br />- 如果通过HMI通知驾驶员异常情况，但是驾驶员在10s内没有介入，则立即停车<br />注意：<br />在检测到任何故障的情况下，一定会停车<br />Monitor与Guardian解耦的原因：<br />- 避免单点故障，增强可靠性<br />- 方便ActionCenter扩充功能<br />- 解耦monitor和Guardian的职责，monitor与HMI联系，而Guardian专注Action

### Apollo底层原理

略

## 参考页面
<https://github.com/Autoware-AI/autoware.ai/issues/1860><br /><https://github.com/Autoware-AI/autoware.ai/issues/1986><br /><https://gitlab.com/autowarefoundation/autoware.ai/common/-/issues/4><br />ROS Observer (!47) · Merge requests · Autoware Foundation / MovedToGitHub / common<br />Implemented feature As the part of failsafe action framework, ros_observer in common has been implemented in this MR. ...<br /><https://gitlab.com/autowarefoundation/autoware.ai/common/-/merge_requests/47><br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427171162-3aae7568-cf6c-48bd-80aa-424030da9946.png#averageHue=%2387c03c&clientId=u1e1088f9-c381-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u941f7246&margin=%5Bobject%20Object%5D&name=image.png&originHeight=167&originWidth=168&originalType=url&ratio=1&rotation=0&showTitle=false&size=4341&status=done&style=none&taskId=udfc49e77-9027-4cd2-a4ed-7c52dfe21b9&title=)

## 问题讨论

- emergency planner是单独做还是交给planning的decision_maker统一调控，还是分层，先给decision_making做，再给emergency planner做<br />- odd planer的定位，是风险缓解（风险状态管理）还是功能切换（多功能模式场景管理）？我认为还是单纯做风险缓解比较好。功能切换交给task manager做就好。<br />- 与远程接管系统的协作，ROS系统的taskmanager 还是emergency handler与MDVR沟通？

## todo

- 日志记录和离线分析<br />- 数据可视化开发<br />- 开机自检
