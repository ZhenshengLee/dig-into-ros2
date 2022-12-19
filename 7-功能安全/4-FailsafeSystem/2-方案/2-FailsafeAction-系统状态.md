# 风险状态管理设计

风险识别：已知将至

- 任务的复杂性和算法的局限性，使得当前的自动驾驶任务执行均带有ODD的局限。
- 本模块旨在在分析当前状态信息，选择一个合适的ODD，然后对外发布ODD状态
- 与业务模块协作执行最低风险策略
- 任务的复杂性和算法的局限性，使得通过人的参与缓解风险变得非常必要。
- 本模块还需要管理人员接管的状态

该资源已被删除<br />关键字：risk analysis, odd reporter, HMI proxy.

## 需求

需求见《诊断模块设计总体方案》

## 原理

自动驾驶人机交互有一项基本要求为：当用户无法及时响应自动驾驶系统发出的接管请求时，自动驾驶系统应执行最小风险策略MRM，以保证车辆运行安全<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427230204-95a09691-1772-485e-986f-d38953595913.png#averageHue=%23bacdb6&clientId=ua20c0e8f-4e7a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u3a75c750&margin=%5Bobject%20Object%5D&originHeight=702&originWidth=1243&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u2443ddc8-b315-4328-adbf-1b5d6f442ea&title=)<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427230204-c651df9f-63ed-469c-85e9-65f1939c9c03.png#averageHue=%23d8ba95&clientId=ua20c0e8f-4e7a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u05ea71da&margin=%5Bobject%20Object%5D&originHeight=360&originWidth=694&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u8cf5d7ba-b660-4d2d-b246-e82d1be02b0&title=)

## 方案

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427230251-2e44c55b-7671-489a-ad3b-488f4f5e3f53.png#averageHue=%23f8f8f7&clientId=ua20c0e8f-4e7a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u28acb39d&margin=%5Bobject%20Object%5D&originHeight=581&originWidth=954&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u42c8a7fa-b46d-411d-99db-531bb7599fc&title=)<br />ODD Manager接受的信息都是可预期事件（预定义数据SystemStatus），目前不考虑意外事件的发生<br />如果意外事件可以转化为可预见的信息，则可算作可预期事件。<br />诊断数据采集方案按照《总体方案》实施。<br />协作式风险缓解主要与规划模块协作停车任务的执行。<br />所有降级任务由各个模块订阅降级指令，自行降级。<br />输入信息：

- 计算机健康信息（资源占用情况）
- 功能模块健康信息（算法自诊断信息）
- 远程接管命令

输出信息：

- 风险等级（给各功能模块用于降级）
- 风险等级（给操作员查看，操作员决定是否远程接管）

协作信息：

- 与规划模块的停车交互

对于诊断数据采集：

- Sensor驱动无法判断数据是否有效
- Sensor传感器数据质量几何，从后果的角度看，是看是否有利于算法性能提升
- 由数据使用者对其进行输入校验

传感器数据可按照以下维度：

- 数据不符合客观事实
- IMU非常大的加速度
- 数据不符合算法的要求
- 数据帧率不够
- 数据无法使算法获得预期的性能
- 障碍物数量过多使得优化算法超时

风险等级：

- 看有多大的概率导致系统失效？

## 设计

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427230221-f831f589-22ff-43d1-9241-bf4869157576.png#averageHue=%23f5f5f5&clientId=ua20c0e8f-4e7a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u7d8be448&margin=%5Bobject%20Object%5D&originHeight=460&originWidth=921&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ua3f61200-613a-4f03-8ec9-7181438132c&title=)<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427230325-ae2241ad-4947-4ee2-a704-9f27edd9d7eb.png#averageHue=%23fbfafa&clientId=ua20c0e8f-4e7a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub5540409&margin=%5Bobject%20Object%5D&originHeight=683&originWidth=646&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u210d7c2c-4d23-4cd4-8c62-336575c1904&title=)<br />功能降级之后能恢复么？人员接管超时后进入恢复。<br />紧急停车后超时之后再结束，等待期间只能先恢复执行自检任务后接管？<br />恢复状态能够接管，安全停车不能接管，只能失败或者成功后接管？<br />每个状态的解释

- 标称运行
- 启动之后默认的运行状态，对诊断信息进行过滤分析
- 功能降级
- 存在Warn状态时，进入功能降级状态
- 该状态下对外发布降级状态
- 各个业务模块对降级状态消息进行响应（希望各个业务模块降级至符合远程接管要求）
- 远程接管
- ODDManager不主动请求HMI进行接管
- 操作人员通过HMI观察到告警信息后主动接管
- 该状态下HMI Proxy需要检查是否符合远程接管条件
- 远程接管情况下屏蔽所有诊断告警
- 安全停车
- 存在Error状态时自动进入安全停车状态
- 该状态下需要Planning模块检查安全停车条件，如果不满足则返回安全停车异常（安全停车状态的子状态，图中未画）
- 安全停车执行由规划模块完成，规划模块完成后返回成功
- 安全停车执行成功后进入恢复状态
- 恢复运行
- 恢复运行下执行健康诊断程序
- 各个业务模块应该恢复之前中断的任务
- 恢复成功之后进入标称模式
- 紧急停车
- 当遇到Fatal状态时进入该状态
- 当其他缓解措施执行失败时进入该状态
- 该状态不可恢复，业务模块结束运行，只保留部分模块运行
- 在该状态下必须等待操作人员现场解决机器故障

不同错误等级的解释

- 未定义
- OK
- 没有问题
- WARN
- 没有必要终止命令，但是需要记录日志以及通知用户
- ERROR
- 错误，必须终止当前命令执行，需要记录日志以及通知用户
- FATAL
- 致命，需要尽快停止当前命令执行，需要记录日志以及通知用户
- 没有时间限制，尽快直接通知底层进行紧急制动

## 状态机实现

状态机选型<br />也有不用第三方库直接手撸状态机的，比如CARMA的guidance模块和autoware.auto的汽车驱动接口，不过都是一层状态，状态迁移也比较简单。<br />使用第三方库可以有效模块化代码，增强可读性和可维护性。

|  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
| 名称 | 介绍 | UI界面 | ROS1 | ROS2 | 备注 |
| SMACH | 最早的ROS状态机 |  |  |  |  |
| autoware, CARMA | 简单版本的 | 无 | 有 | 有 | 用于规划模块 |
| ros_scxml | swri的状态机，基于QT | 无 | 有 | 有 | 美国西南大学 |
| flexbe_behavior_engine | 自动代码生成，基于python | 有 | 有 | 无 | DARPA2015团队 |
| SMACC | 修改自SMACH，基于boost | 有 | 有 | 未来有 | 202006最新 |

选择SMACC

### 消息定义

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671427230585-c52e6881-9c08-4310-b776-582421588c33.png#averageHue=%23efefef&clientId=ua20c0e8f-4e7a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u9a03ca83&margin=%5Bobject%20Object%5D&originHeight=715&originWidth=1827&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u8e202cfc-bf67-4a70-b85d-1cb46ed2f70&title=)

### 集成测试

目前carla可用性不高，自己开发mockNode

- mockMonitor
- rostopic
- mockHmi
- rostopic
- mockerPlanner
- 自己开发一个actionServer

## 后续文档

### 架构更新

增加对taskmanager的交互<br />增加对motorSetup的交互

### 状态机更新

逐步细化状态迁移机制

### WARN等级的细分

- WARN-LOW
- WARN-MIDDLE
- WARN-HIGH

### 各个模块的降级响应

- 定位：

### 远程接管条件

- 速度，区域等条件

### 恢复作业的条件

自诊断的规程
