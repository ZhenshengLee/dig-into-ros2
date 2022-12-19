# 需求

## 概念

智能网联汽车自动驾驶包括有条件自动驾驶、高度自动驾驶和完全自动驾驶。有条件自动驾驶是指在系统的设计运行条件下完成所有动态驾驶任务，根据系统动态驾驶任务接管请求，驾驶人应提供适当的干预；高度自动驾驶是指在系统的设计运行条件下完成所有动态驾驶任务，在特定环境下系统会向驾驶人提出动态驾驶任务接管请求，驾驶人/乘客可以不响应系统请求；完全自动驾驶是指系统可以完成驾驶人能够完成的所有道路环境下的动态驾驶任务，不需要驾驶人/乘客介入。

设计运行条件（Operational Design Condition，ODC）是驾驶自动化系统设计时确定的适用于其功能运行的各类条件的总称，包括设计运行范围、车辆状态和驾乘人员状态等条件。其中，设计运行范围（Operational Design Domain，ODD）是驾驶自动化系统设计时确定的适用于其功能运行的外部环境条件，一般包括：1）道路边界与路面状态；2）交通基础设施；3）临时性道路变更；4）其他交通参与者状态；5）自然环境；6）网联通信、数字地图支持等条件。

《安全指南》第1、2条要求，自动驾驶系统应有明确的设计运行范围（ODD），并采取相应措施，确保车辆在ODD范围内安全运行。系统供应商应根据自动驾驶汽车的性能及使用方式确定设计运行范围，例如具体运行的道路条件（高速公路、普通道路、车道数目等）、地理条件（城市、山区等）、环境条件（天气状况、夜间等）、其他条件（限速、特定道路运行、安全人员乘车与否等），并对自动驾驶汽车引发的人身事故进行合理预见并预防其发生。在确定ODD范围之后，自动驾驶系统应在ODD范围内启动，司机应能控制自动驾驶系统的启动。厂商应确保控制系统及传感器必须有冗余，保障系统安全。如果车辆运行超出ODD范围，L3级自动驾驶系统应向司机发出警告要求介入，而L4级自动驾驶系统应能自动将车辆停到安全的地方。

《安全指南》要求应有确保自动驾驶系统良好运作的人机交互机制和技术措施保障。具体而言，厂商和经销商应采取措施，让驾驶员了解包括系统使用方法、ODD范围、系统异常时车辆的反应、适当进行软件升级等一系列信息。例如，该指南第4条规定，自动驾驶系统的人机界面（HMI）应能监控驾驶员，确保驾驶员可以随时接管车辆操作，包括必要时发出警报（L3级别）或能让驾驶员或乘客事先知道车辆会自动停止前进（L4级别）。L4级别的自动驾驶汽车应设置摄像头和声音通信设备，使运行管理中心可以监控车内状况，且应在乘客可轻易触碰到的地方设置紧急停止按钮。该指南第7条还规定，在紧急状况停车或有事故时，系统应能自动通报运行管理中心；车辆紧急停止时和运行管理中心的联络，以及紧急状况下的应对情况，应能通过人机界面方便地传递给乘客。

## 名词解释

| 名词 | 解析 | 项目裁剪 |
| --- | --- | --- |
| 功能安全 | 自动驾驶系统应有明确的设计运行范围（ODD），并采取相应措施（监控系统内部和外部的条件），确保车辆在ODD范围内安全运行 | 不裁剪 |
| ODD（设计运行域） | 由于技术不成熟，设定一个限制条件，自动驾驶系统在该条件下运行，自动驾驶系统能够检测系统运行环境是否符合该条件，在不符合该条件情况下应安全退出自动驾驶功能<br />（主要涉及系统外部的条件） | 不考虑外部天气，道路状况，夜间等<br />仅考虑道路中的动态障碍物如行人，自行车等<br />考虑外部通信 |
| 诊断系统 | （系统内部的条件）<br />算法自身产生的问题，导致自动驾驶系统失效。系统采取措施，使得故障能够迅速诊断和记录。 | 考虑自身计算系统的可用性<br />检测算法系统使用是否符合条件 |
| MRM（最小风险缓解措施） | （自身补偿措施）：当出现问题，自动驾驶系统采取的措施， | 最保守的措施包括降速，最激进的措施就是~~紧急停车~~安全停车（安全停车和紧急停车需要区分）<br />Minimal Risk Manoeuvre |
| MRC（最小风险条件） | （进入最小风险安全状态的条件）<br />执行风险缓解措施后，系统进入的状态 | 通过判定一系列条件，才能进入该状态，否则继续执行MRM<br />Minimal Risk Condition |
| 远程人机交互HMI | 在进入和退出自动驾驶功能过程中依赖与监控人员的交互机制，根据交互机制的不同，自动驾驶均可以主动，被动进入和退出 | 不裁剪 |

## 总体要求

ISO-22737-2021

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1646119632934-5c28a6a4-5e6d-4f9f-87b1-64705a7cdfe6.png#averageHue=%23f9f8f7&clientId=u99200f3e-dc85-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u2914767d&margin=%5Bobject%20Object%5D&originHeight=523&originWidth=494&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u9a647f8e-15a5-43b6-9cc3-ea23d38c8d1&title=)

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1646119632908-25d94d18-e05f-4419-80be-6c9f91174ecc.png#averageHue=%23fcfcfc&clientId=u99200f3e-dc85-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u6deba27a&margin=%5Bobject%20Object%5D&originHeight=638&originWidth=881&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u0a2cb47f-8a06-43a9-a023-f3a2a4d7ee1&title=)

关键词<br />A1 开机并通过自检<br />B1 系统故障或断电 调度员命令或电源关闭<br />B2 满足 ODD 条件，调度员发出参与 ADS 命令，配备 ADS 的车辆具有数据记录能力并参与其中。<br />C1 调度员解除指令<br />C2 乘客或调度员启动紧急停车<br />C3 检测到 LSAD 系统无法处理的危险情况或 DDT 性能相关的系统故障或失去安全关键的 V2X 通信或即将违反 ODD 或未收到调度员的安全继续确认授权<br />C4 车辆处于静止状态，即 0 米 / 秒<br />C5 调度员确认进入待机状态<br />C6 车辆处于静止状态，即 0 米 / 秒，并由调度员确认进入待机状态

## 总体需求

宏观需求：基于功能安全的人机交互<br />当前的策略：算法搞不定的，就报ODD

- 每个模块生成若干indicator，形成一个模块状态，每个模块在任何时刻都有一个状态
- 各个模块的状态汇总成为一个总体状态，称作odd状态，代表当前系统的能力

终极目标

- 尽可能早的发现故障，通过人机交互指导操作人员排除故障
  - 对于故障，要找到根本原因
  - 处理故障之间的依赖（算法故障依赖数据故障，数据故障依赖驱动故障，驱动故障依赖硬件故障）
- 让系统诊断出并且记录所有已经发生的已知故障
  - 找到故障
- 根据不同等级的故障，做出不同的mrm（最小风险策略Minimal Risk Manoeuvre）
  - 故障的细粒度
  - 失效分析，场景分析，各个算法对ODD状态的响应

## keyword

ROS, autonomous vehicle safety, fail safe operation, fail safe action, monitoring system, diagnostic system, state check,  functional safety, operational design domain

# 当前方案

# 诊断数据的采集

## 要求

LSAD 系统应使用传感器，以实现部分动态驾驶任务。这包括探测物体、车辆、行人、建筑物、路径等。应针对传感器性能和故障以及其他安全关键系统元素进行适当的危险分析和风险评估。 LSAD 系统的开发应根据 ISO 26262 系列和 ISO 21448 进行<br />内部外部诊断数据的采集汇总

## 当前实现

失效分析+数据采集<br />（状态数据有很多个指标构成indicator）

- 系统计算状态
  - 操作系统资源
  - 网络资源
- ROS状态
  - 消息通信时延
  - 消息丢失
- 算法的失效
  - 计算失效
  - 通信失效

## 新增实现

- 传感器和执行器等硬件失效（驱动级别）
- 当前环境状态（物体、车辆、行人、建筑物、路径）
  - 障碍物的种类
  - 障碍物的距离和速度
- 检测通信丢失
- 不仅仅汇报错误的状态，还应该汇报正确的状态

# 业务状态汇报

## 要求

- 分析当前的车辆状态，系统（模块）一个时刻只能处于一个状态（状态机）
- 在启动之后，系统能够有充足的理由判断（经过自检，验证odd条件）可以进入自动驾驶odd, 从standby 到 active
- 运行过程中危险情况确定

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1646119738332-1a94819f-fb21-4c47-bb19-37c065f23ff2.png#averageHue=%23f4f4e5&clientId=u70462f48-5954-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=439&id=u9e693489&margin=%5Bobject%20Object%5D&name=image.png&originHeight=439&originWidth=1011&originalType=binary&ratio=1&rotation=0&showTitle=false&size=130906&status=done&style=none&taskId=u4f7fee99-3525-43c2-b530-e3e97342bbf&title=&width=1011)

## 当前实现

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1646119750625-737dab6f-fa04-4f16-b6cd-6b4bb9a8e367.png#averageHue=%23e0e0e0&clientId=u70462f48-5954-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=163&id=u04a9b733&margin=%5Bobject%20Object%5D&name=image.png&originHeight=163&originWidth=1125&originalType=binary&ratio=1&rotation=0&showTitle=false&size=93602&status=done&style=none&taskId=u36bfe50d-a752-4732-98a7-2083e1d331f&title=&width=1125)

- 算法模块启动顺序为手动，无法实现顺序开启和顺序关闭
- 系统无法确认启动过程中的中间状态，在中间状态下的车辆行为未定义/不清晰
- 规划算法实现了一个状态机，但是该状态为内部使用，在rviz中可视化，未上报给监控人员

## 新增实现

顺序开启和顺序关闭<br />各个模块需要有关闭，打开，初始化，激活运行，去激活，暂停等状态<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1646119760695-b543439f-ef2f-49a2-a54f-1b20c7956a4b.png#averageHue=%23fafafa&clientId=u70462f48-5954-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1079&id=u360f7adf&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1079&originWidth=1342&originalType=binary&ratio=1&rotation=0&showTitle=false&size=146384&status=done&style=none&taskId=u9fc19b5c-5cbd-4916-8da3-7aa2ac6bf9c&title=&width=1342)

每个模块间的依赖关系需明确

- 导航流程状态
  - planning状态：撞边？正常？可驾驶区域被封锁？
  - tasking状态：等待任务，分解任务，下发任务
    - tasking目前无状态
- control无状态

# odd状态和MRM

## 要求

LSAD系统应按照图 3 的状态转换图运行。超出图 3 描述的具体实施应是制造商的责任。

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1646119633169-3e4909f3-c5f6-4515-993f-27041779864a.png#averageHue=%23fcfcfc&clientId=u99200f3e-dc85-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u27d1fc25&margin=%5Bobject%20Object%5D&originHeight=684&originWidth=911&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uf22892bc-b0ef-4223-a4e9-afa5a67b724&title=)

## 当前实现

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1650768895390-a3fe7209-8797-42ea-9365-0dd9dc39fa17.png#averageHue=%23f5f5f5&clientId=u7c859085-6f9d-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u5dbc289c&margin=%5Bobject%20Object%5D&originHeight=460&originWidth=921&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u02a048ae-9fb1-47b6-8ad7-dba210c367f&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1650767092206-d87a57f3-ffa5-475f-b13b-6b4356425509.png#averageHue=%23f5f4f4&clientId=ud655aae2-3a01-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=351&id=uc6a52480&margin=%5Bobject%20Object%5D&name=image.png&originHeight=351&originWidth=351&originalType=binary&ratio=1&rotation=0&showTitle=false&size=29679&status=done&style=none&taskId=u1edcfae7-ea1b-41a4-b883-a32be52e2a6&title=&width=351)<br />系统（模块）一个时刻只能处于一个状态（状态机）

- 总体状态
  - Normal, EmergencyStop, 等
- 目前仅有线控模块响应紧急状态
  - 当处于紧急停车状态时刻，则不响应导航模块下发的底盘控制指令
  - 远程监控介入后，也屏蔽导航下发的底盘控制指令

MRM有实现，但是局限在规划模块内部

- 降级状态（如遇行人减速，遇障碍物停车）
  - 降级状态可以不走oddmanager，当有需要协同降级的时候，需要顶层odd_manager的参与
  - 也许有观点认为可以不走odd_manager，但是基于功能安全的运营交互要求这些状态必须上报，意味着这些状态必须是可知，而且是集中上报的。

## 新增实现

- 其他MRM
- 更多算法功能降级
  - 定位算法选择另一种配置
  - 规划算法选择另一种配置
  - 感知算法停止某种传感器数据处理以降低系统负载

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1646119633175-45f58667-5195-40a2-9738-05bc593d4025.png#averageHue=%23f2efef&clientId=u99200f3e-dc85-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ua6867212&margin=%5Bobject%20Object%5D&originHeight=385&originWidth=794&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u5e8c4cae-d7eb-4ac3-9830-9193aba47cd&title=)

# 远程人机交互

## 要求

根据 LSAD 系统的实现，安全关键事件数据应在 LSAD 系统驾驶的车辆和调度员或控制中心之间进行通信。安全关键数据的选择应根据 7.3 的规定，由相关的利益相关者（如当地政府、服务提供商、制造商等）商定

## 当前实现

状态可视化和记录

- indicator可视化
  - runtime monitor
  - remote hmi
- odd可视化
  - 状态图可视化
- 日志记录
  - log数据库

## 新增实现

业务状态汇报的可视化和日志

# 环境交互

## 要求

在 LSAD 激活状态下， LSAD 系统应监测由 LSAD 系统驾驶的车辆的周围环境，并应确定是否存在危险情况。危险情况可能涉及到骑自行车的人、行人（儿童和成人）或车辆和 / 或静止的和动态的障碍。一个危险情况可以由于其他静态 / 动态物体而被遮挡住。一旦 LSAD 系统确定了危险情况，该系统应采取行动，避免与障碍物相撞，并向外部道路使用者发出警告。

## 当前实现

没有根据odd状态设置任何警示标识

## 新增实现

- 向道路使用者提供警告
  - 警示灯
  - 声音
- 向道路上的其他自动设备提供信号
  - 发送特定报文

# 新增功能参考实现

| 领域 | 归属模块 | 当前不足 | 改进方案 | 参考实现 |
| --- | --- | --- | --- | --- |
| 诊断数据采集 | monitor/算法 | 传感器和执行器等硬件失效 | 驱动程序诊断，节点数据分析 | [https://github.com/autowarefoundation/autoware.universe/blob/main/system/velodyne_monitor/Readme.md](https://github.com/autowarefoundation/autoware.universe/blob/main/system/velodyne_monitor/Readme.md) |
| 业务状态汇报 | 算法 | 算法模块状态不明<br />算法模块没有降级运行机制 | 算法模块建立状态机<br />明确关键算法模块的降级机制 | <https://github.com/boschresearch/ros1_lifecycle> |
| MRM的触发机制 | 算法 | 虚报率较高<br />失效分析和接管场景覆盖不全 | 针对《自动驾驶运营规范》等标准性文件，采集更多状态数据 | 基于失效分析 |
| odd状态和mrm | odd | odd状态细化<br />安全响应加上自动恢复机制<br />安全响应加上降级机制 | mrm增加减速行驶，靠边停车<br />增加自动恢复机制 | [https://github.com/autowarefoundation/autoware.universe/blob/main/system/emergency_handler/README.md](https://github.com/autowarefoundation/autoware.universe/blob/main/system/emergency_handler/README.md) |
| diag消息定义 | odd/monitor | 多层嵌套没必要system status，node status, hardware status | 采用ROS2 diagnostics单层嵌套方案<br />对应的可视化开发 | <https://github.com/ros2/common_interfaces/tree/master/diagnostic_msgs/msg> |

# 新方案讨论

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1650766846637-0ce4b02e-f64e-44c2-b8a4-1f7c039c39b9.png#averageHue=%23eeeeee&clientId=ue1a58df3-1143-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=382&id=ubfae2b8b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=611&originWidth=1280&originalType=binary&ratio=1&rotation=0&showTitle=false&size=449001&status=done&style=none&taskId=ub951dd7f-913f-4597-ba71-50a8a74da7e&title=&width=800)

## 消息定义

## 接管后任务恢复

主要工作在任务管理，odd暂不需要改变

# 未来工作

ROS2工具移植：以ODDM为例<br />参考《深圳自动驾驶运营规范》适配更多场景

# 参考

【腾讯文档】ISO 22737 - LSAD-standard<br /><https://docs.qq.com/pdf/DYmtpYkxUQ2pScHFF><br />【腾讯文档】ISO 22737 预定低速自动驾驶系统-系统要求、性能要求和性能测试规范(中文版)<br /><https://docs.qq.com/pdf/DYnZ4cW1Vc2dzSG91>

<https://github.com/tier4/AutowareArchitectureProposal.iv/tree/use-autoware-auto-msgs/system>
