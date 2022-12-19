# 术语

| 名词 | 解析 | 举例 |
| --- | --- | --- |
| 应用程序管理 | 程序的打开，关闭，重启等 |  |
| 生命周期管理 | 对软件组件，甚至设备进行更加细致的管理<br />组件需要暴露状态切换和上报的接口 |  |
| ODDM管理 | 是对系统能力的评估和管理，用来规避风险，保证车辆安全 | <br /> |
| local_app | **安全员**通过平板使用的app，主要用来下发任务，然后离场 |  |
| remote_hmi | **调度员**使用的远程监管客户端，包含全生命周期管理功能 |  |
| 诊断系统 | 采集各个软件组件错误告警信息的组件 |  |
| 设备生命周期状态 | 设备在生命周期管理下的各个状态 |  |
| 算法组件生命周期状态 | 算法组件在生命周期管理下的各个状态 | 激活 |
| 系统能力状态ODD状态 | ODDM对系统能力的评估后得到的状态，是系统(顶层)级别的状态 | 接管状态(**关键**算法组件**失效**且不可**自恢复**) |
| 算法组件能力状态 | 反映算法能力的状态，与生命周期状态不应混合 | PNC的STOP状态是一种能力状态<br />(算不出来才停的) |
| 算法组件的模式 | 算法组件允许在线重配置，修改一些参数，达到不同的算法性能 | 定位组件在高能耗**模式**(最大迭代次数设置为100)下依然上报了定位失效的**能力状态** |

生命周期管理举例：

- 安全员通过app登录设备，设备进入登录激活模式
- 安全员按下启动按钮，设备导航系统软件启动，各个组件进入unconfigured模式，等待参数配置
  - 系统设备生命周期状态为自检通过配置中

系统能力状态举例：

- 定位组件在低能耗模式下上报了定位失效的能力状态
- ODDM根据该信息，得到顶层能力状态为关键算法失效，紧急停车
- 采取自恢复措施，将感知组件去激活，将定位组件调整到高能耗模式，在高能耗模式下，同时启动两种定位组件，CPU占用率高，但有助于迅速恢复
- 高能耗模式下持续上报定位失效，紧急停车且请求接管

# 概述

在之前的导航系统实现中，用户已经可以通过robot_hmi对各个算法进程进行启动停止，对任务进行简单的控制，查看算法的状态等<br />[ODDM迭代方案-面向功能](https://www.yuque.com/docs/share/4fcf0cb9-92c9-4677-8200-e86bc0c3e1e2?view=doc_embed)<br />在上文中对前述的功能进行了总结，对新增功能的需求进行了简单分析。<br />调度员和安全员希望通过人机交互的方式，对机器人导航系统及自动化作业任务进行全生命周期管理，由于场景和参与交互的时机不同，调度员和安全员对人机交互的需求各有侧重。<br />导航系统属于算法密集型系统，管理组件与算法组件强烈耦合，生命周期管理不可避免要获取算法的状态，所以算法组件的生命周期管理需求明确且强烈。<br />算法组件生命周期管理，区分启动，配置，激活，暂停(去激活)等阶段，明确算法状态，可以得到多个好处

- 根据人机交互的需要配置算法组件活动，包括启动，配置，激活，暂停，停止。
- 细化每个阶段的工作，提前发现问题，增强系统可靠性和使用友好性
- 动态配置算法的能力，使能去使能算法模块，实现动态计算，按需使用计算资源
- 区分不同算法状态，明确算法能力，使ODD真正发挥作用，方便对算法进行不同能力的配置（定位高功耗模式，感知低功耗模式等）

# 架构

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757122940-79d028be-480d-4e7e-b1cf-e582b68fc8b5.png#averageHue=%23f7f5f4&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u198a512f&margin=%5Bobject%20Object%5D&originHeight=701&originWidth=1282&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u435d01f3-33d6-4d83-b354-06e09269393&title=)

- 当前的server_bridge具有三个组件的职责：remote_hmi + lifecycle_manager + mode_manager
- odd_manage的本质是反映算法组件（skill layer）的能力，分析风险，执行风险操作保证当前能力

## 算法组件和管理组件

算法组件(skill layer)包括：

- localization
  - 新架构由一个节点分解为4个节点
  - 多个节点的还应该有localization manager算法管理组件统筹状态发布
- perception, 感知
  - 感知架构为单体架构，已经有一个app manager的组件
- planning，规划避障
- control, 控制
  - 包括dbw作为底盘控制代理
- map-provider
  - 为感知规划定位提供先验知识
- 任务规划器task planner
  - 任务生成task generator，对清扫区域的顺序进行维护，以清扫效率为原则
  - 路径生成route generator，对全覆盖清扫进行路径生成，以清扫效率为原则

管理组件(deliberation layer)包括（此处的管理为基于人机交互的生命周期管理）

- 登录(用户)管理(图中略)
  - 目前与server-bridge 合并
- 生命周期管理器lifecycle-manager
  - 与server-bridge合并
- RPC代理remote_hmi
  - 与server-bridge合并
- 任务执行管理task-exe-manager
  - 与taskplanner合并为task_manager
- 地图管理map manager
  - 目前与map provider合并

### 算法和管理耦合

可以看到task_manager和map_provider是同时具有算法和管理职能的

## ServerBridge和ODDManager的职责划分

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757122922-3e3f0c7b-c239-4ab7-b99e-59a8caeaaf36.png#averageHue=%23f4f4e5&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u442adabe&margin=%5Bobject%20Object%5D&originHeight=439&originWidth=1011&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uaf07ccc2-0627-4d71-a162-a5894223615&title=)<br />**这个图的问题在于，将生命周期管理和算法能力管理耦合在一起。**<br />**只有任务执行的时候，ODD才能够生效。**

App(RPCClient) -> ServerBridge(RPCServer) -> 各个算法组件和管理组件<br />ServerBridge本质上是生命周期管理器，负责转发调度员的操作意图，向调度员反映当前软件生命周期，其上报状态靠任务执行管理和各个算法上报的数据作为支撑。

ODDManager本质上是算法状态(模式配置)管理器，负责管理算法的能力(清扫区域，非清扫区域，车位，空旷环境，人流密集环境)，根据环境和算法本身，切换算法的配置（降级，低功耗，高性能模式等），在算法失效情况下与算法协作执行最小风险操作<br />算法组件-> Monitor->ODDManager -> DBW

当前RPC代理，生命周期管理（应用程序管理），状态上报等功能都合并在ServerBridge，如果由odd_manager管理软件生命周期，则会出现serverbridge与oddmanager同时(主动)管理算法启动状态的情况。

- 分析：odd中的ready并不是生命周期的ready，而是算法能力的ready，odd_manager不应参与管理生命周期
- 解决办法：odd_manager只被动反映是否进入standby(ready) 状态，由生命周期管理器触发从OFF->Ready

# 关键决策

## 总体流程

lifecycle-manager接受app的rpc request，转译为ros机制，返回rpc response<br />lifecycle-manager可以不设置状态机，但是必须检测前置项是否已经配置，比如下发任务之前必须检查是否已经配置地图，是否已经配置任务<br />lifecycle-manager通过ros机制与各个管理组件通信（当前管理组件和算法组件往往耦合为一个节点，比如hdmap）<br />lifecycle-manager采集各个管理和算法组件的状态，转译上报

## 设备生命周期管理

APP是对设备进行生命周期管理的媒介工具<br />同时server端应该有一个状态机服务APP响应<br />对设备的管理分为：

- 设备访问确权
  - 登录
  - 连接设备
- 设备启动，设备可用性确认
  - 进入主页
  - 遥控模式
- 导航初始化
  - 自动定位
  - 基于对话的定位
  - 纯手动定位
- 任务管理
  - 新建任务
  - 下发任务
  - 已下发任务查看
  - 任务执行
- 任务进度查看
  - 略
- 安全（健康）管理（该安全中心在remote-hmi，因为local_app不涉及运行时监管，下发任务人即离场）
  - 设立一个安全中心页面，作为主页的一部分
  - 中断及恢复功能也在其中

## 算法组件生命周期管理

所有算法组件都必须有生命周期，包括map_provider, task_planner

### 进程的状态模型

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757122956-cf9ef068-7105-45fd-a7c7-8e73930d3016.png#averageHue=%23f0f0f0&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u5ea012fb&margin=%5Bobject%20Object%5D&originHeight=231&originWidth=358&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u7c1bae54-4814-458e-8e5b-ec7e1766d45&title=)<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757123650-cbe2d0a1-6d42-4e72-820e-f30edfcd5297.png#averageHue=%23efefef&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=CGdeT&margin=%5Bobject%20Object%5D&originHeight=357&originWidth=442&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u1bd7b229-e738-41fa-b710-c44c23f3de1&title=)

### 进程状态管理接口

ps -aux 通过虚拟文件系统获取进程信息的/proc directory<br />kill -9 是通过信号机制管理的

### 算法组件的状态模型

ROS2设计人员在2015年给出了lifecycle_node的设计

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757123929-3d4149fe-f3fd-4254-981e-7c299ddbf59e.png#averageHue=%23fafafa&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uff8316a9&margin=%5Bobject%20Object%5D&originHeight=545&originWidth=834&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uf3b6a84f-7bf5-440e-88a0-805523da900&title=)

博世的开发者在此设计理念的基础上在ROS1上实现了lifecycle_node<br /><https://github.com/boschresearch/ros1_lifecycle>

### 算法状态管理接口

ROS的机制，就是topic和service

## 算法组件模式管理
<https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md> 中给active状态添加了更多的modes模式，在各个模式下，算法组件具有不同的parameter，用来表示动态行为的不同配置。

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757124495-6b37038d-51c4-4fc3-9327-3a245eb06232.png#averageHue=%23161515&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u80dd6d57&margin=%5Bobject%20Object%5D&originHeight=333&originWidth=400&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u48021dec-058f-41fd-90fc-b8bc59f84bc&title=)

## 算法组件能力状态

端到端

# Server设计

## 自检逻辑

### 需求

自检的本质是，检查**自动作业能够开始的必要条件。（本质上还是一个失效分析问题）**<br />主要包含计算系统自检，嵌入式系统自检，导航系统自检<br />进入主页后，所有进程默认启动，但是算法组件处于unconfigured状态<br />此时monitor会执行系统自检：检查网络，各个进程，软件版本，地图版本等信息<br />算法自检：此时各个算法组件还要执行自检过程，提前报告问题，减少操作人员工作量，避免在运行中发现问题

### 自检机制

**机制1：**将自检归结于一个诊断问题（可复用health_check）<br />hardware_status版本，借用现有的health_check，在自检过程中只检查system_status中的hardware_status，如果hardware_status无错误即可判断自检通过<br />**机制2：**将自检归结于一个生命周期问题（主动执行）<br />如果一定要加软件自检，则软件从启动到未配置状态之间需要执行必要的自检过程，该过程做哪些工作，发现的问题依然可以通过node_status向上报告<br />在生命周期状态机中加入self_check工作，组件启动过程中执行，组件主动上报自检状态，接口依然是node_status<br />**机制3：**将自检归结于一个生命周期问题（被动执行）<br />每个组件增加self_test 服务，由生命周期管理者负责轮询发起，所有组件返回成功则说明自检成功，否则返回失败原因<br />认为自检方案1是比较实际的

## ODDM状态(系统能力状态)设计

ODDM状态是顶层状态<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757124976-c82f8c5f-2a0b-4bd5-89f5-81063c5cd263.png#averageHue=%23f5f4ef&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u41a4e289&margin=%5Bobject%20Object%5D&originHeight=287&originWidth=907&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u32808d13-599c-41b1-9767-23ad0ccc2ea&title=)<br />处于OFF状态下，不会做任何操作<br />待机状态下，odd_manager不会发送紧急停车指令到motor_setup

## 任务中断和恢复

恢复分为手动恢复和自动恢复，在自动恢复失败后请求接管，然后进行手动恢复。<br />手动恢复：

- 在车辆紧急停车后，请求接管
- 现场人员遥控移动车辆到安全位置，报告结束作业，ODD进入standby
- 向任务管理发送恢复指令

自动恢复(图中未体现)：在算法组件提交错误诊断信息后，车辆进入自主恢复状态，在自主恢复状态下，车辆实施自主逃脱，逃脱成功（诊断信息不包含错误）则回归normal，逃脱失败（超时），则进入紧急停车模式，紧急停车模式下请求接管。<br />自动恢复的风险：在系统不稳定的情况下，系统可能一直处在自主恢复状态下（在normal和recovery状态间震荡）<br />自主逃脱机制：比如定位进入高功耗模式，增强定位可靠性，感知进入低功耗模式为系统让出资源，规划实施后退行为等<br />建议先实现手动恢复，再实现自动恢复。

## 设备生命周期状态定义

设备生命周期状态只关注组件的生命周期，是否启动，是否配置好，是否激活，是否在工作<br />所以本状态图没有包含FAULT状态，FAULT状态关注的是算法能力。<br />IDLE-WORKING-FAULT? 这三个状态和ODD的三个状态有啥区别？<br />认为生命周期状态包含启动，配置，激活，去激活，结束<br />如果没有配置和去激活状态，那么就是启动，激活，结束状态。

## 算法组件生命周期状态定义

### 统一的算法状态定义

算法组件生命周期

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757124998-d38adfaf-5586-4d07-852a-467f18272487.png#averageHue=%23fafafa&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ue3790895&margin=%5Bobject%20Object%5D&originHeight=1079&originWidth=1342&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ufcd3a135-d5f2-4641-89d3-126f7034734&title=)

### 算法模式定义

通过对外暴露dynamic reconfigure来暴露动态配置接口，使得算法组件进入某种模式<br />如下列的模式设计：<br />定位模式设计：默认，高性能，低能耗<br />规划模式设计：默认，避让（靠边缓行），保守（降级），进攻（加速）

### 非统一的算法生命周期状态定义

简化的生命周期管理和模式管理<br />由planning的简化版得到（luoyilong），suspend, start, stop.<br />需要上报状态消息，对外暴露外部控制接口等

## 算法能力状态定义

错误类型

# Local-app设计

## 运行逻辑

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1656061126339-64da68b1-1317-43e6-af1a-df7bc06e7e20.png#averageHue=%23f6f5f1&clientId=u10c7e399-c420-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=714&id=GTAo2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=714&originWidth=412&originalType=binary&ratio=1&rotation=0&showTitle=false&size=70624&status=done&style=none&taskId=ud4f54406-a6c3-4255-992e-27ac1cb187b&title=&width=412)

## 自检结果呈现

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757125083-77f61437-6cdb-40af-8855-e9a7020ad244.png#averageHue=%23fbfbfb&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u74388313&margin=%5Bobject%20Object%5D&originHeight=741&originWidth=940&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ua2d09c92-b2a2-4295-a3c8-6c8fda3e467&title=)

## 状态上报和显示

### 设备生命周期状态

设备生命周期状态，启动，登录，地图管理，初始定位，任务配置，任务执行。<br />已经内嵌在app中，最终用户通过该app可以知道设备处在何种配置之中

### 组件生命周期状态

每个组件处在何种状态

| 组件名 | 状态 | 注释 |
| --- | --- | --- |
| task planner | unconfigured | 请配置任务 |
| map_provider | unconfigured | 请配置地图 |
| localization | error | 请检查定位节点 |

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757125152-b82c54f6-8555-4f74-ad75-e51e1aa669cf.png#averageHue=%23f4f1ef&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u9a03b523&margin=%5Bobject%20Object%5D&originHeight=117&originWidth=625&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ubed4704e-7cbd-4d2e-9bde-724720cb270&title=)

# remote-hmi设计

## 状态上报和显示

### 设备生命周期状态

设备生命周期状态，启动，登录，地图管理，初始定位，任务配置，任务执行。

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757125517-e387d20b-ce67-4243-9c97-0654f93b14f3.png#averageHue=%23f8f8f8&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u8d88357f&margin=%5Bobject%20Object%5D&originHeight=185&originWidth=272&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u1abe3ef8-25ed-4cce-8917-3531c51396f&title=)

### 组件生命周期状态

生命周期状态图，表明每一个组件是处在unconfigure， activate , inactivate<br />与local的一致

### 系统能力状态(ODD)

ODD状态图，表明当前算法系统处于何种能力之下，当能力不足时采取了何种风险缓解措施<br />off, ready, active(normal), emergency_stop, take_over, automatic_recovery<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1654757125689-26c2ea94-3345-44f8-8809-2a71c992127b.png#averageHue=%23f5f5f0&clientId=u4fccb784-faf3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uf1e47bf0&margin=%5Bobject%20Object%5D&originHeight=271&originWidth=858&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u39ef7a5a-780d-4d14-a8ce-a2afd46ff25&title=)

# 参考代码

生命周期管理器：[https://github.com/ros-planning/navigation2/tree/main/nav2_lifecycle_manager](https://github.com/ros-planning/navigation2/tree/main/nav2_lifecycle_manager)<br />模式管理器：[https://github.com/micro-ROS/system_modes](https://github.com/micro-ROS/system_modes)<br />生命周期驱动程序：[https://github.com/ros-drivers/ros2_ouster_drivers](https://github.com/ros-drivers/ros2_ouster_drivers)<br />生命周期定位程序：[https://github.com/ros-planning/navigation2/tree/main/nav2_amcl](https://github.com/ros-planning/navigation2/tree/main/nav2_amcl)<br />生命周期任务管理程序：[https://github.com/ros-planning/navigation2/tree/main/nav2_bt_navigator](https://github.com/ros-planning/navigation2/tree/main/nav2_bt_navigator)<br />生命周期规划程序：[https://github.com/ros-planning/navigation2/tree/main/nav2_planner](https://github.com/ros-planning/navigation2/tree/main/nav2_planner)<br />生命周期控制程序：[https://github.com/ros-planning/navigation2/tree/main/nav2_controller](https://github.com/ros-planning/navigation2/tree/main/nav2_controller)

# 讨论与回复

太复杂了，为什么要这么设计？<br />因为很多情况要考虑。

# 实施情况

总的来讲：任务状态，组件生命周期状态，设备生命周期状态全部耦合在一起！

- 没有lifecycle-manager，app-bridge通过nav_cli启动所有节点
  - 算法组件没有configure, inactive, active 等状态
  - 没办法提前做到发现错误，没办法做自检
- 没有mode-manager，不通过odd_manager进行降级操作
  - 通过各个算法节点各自判断定位状态进行suspend模式
  - 没有顶层统筹算法模式的能力，odd_manager没参与
  - 各自判断，流程紊乱，故障难以定位
- 提供robot_status，对task_manager的状态进行反映
  - 自检失败，则不允许配置和启动任务。
