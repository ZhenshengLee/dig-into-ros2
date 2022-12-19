# 总体方案

autoware auto方案<br /><https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/issues/821><br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671428051866-aa4f1223-bce0-481c-bbe2-4783e80d17f0.png#averageHue=%23f7f6f5&clientId=u75237b64-aac6-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u3a760446&margin=%5Bobject%20Object%5D&originHeight=614&originWidth=1550&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u7dc381b8-a297-4349-8951-2cabc9acd21&title=)<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671428051801-bcf5b820-031e-4cc7-b1e0-0deedd534663.png#averageHue=%23f3f3f3&clientId=u75237b64-aac6-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub7df9832&margin=%5Bobject%20Object%5D&originHeight=948&originWidth=2052&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uf1f209d0-fed1-470f-af9c-20c4093dee5&title=)<br />tier4方案<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671428051883-97d2a250-59bb-4e8e-b97d-b76d3a49a041.png#averageHue=%23f7f6f4&clientId=u75237b64-aac6-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uffeffe8a&margin=%5Bobject%20Object%5D&originHeight=1040&originWidth=1920&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ud63878d6-affa-4148-a375-3a8025f91ea&title=)<br /><https://gist.github.com/xmfcx/7eaae9750d6317a3a2aa23745ac99444><br />Detect when nodes' incoming messages are skipped (#821) · Issues · Autoware Foundation / Autoware.Auto / AutowareAuto<br />Description Add a mechanism built into nodes to check that all incoming topics are arriving without skipping any messages. We have...<br />https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/issues/821

## 与ROS1区别

- 诊断数据采集
- 只采集了订阅数据是否延迟，没有诊断任何算法失效数据
- 新增状态分析
- ROS1只采集了算法失效数据，而ROS2新增state monitor，诊断出当前车辆的状态
- 车辆当前状态使用一个状态机建模

# 实现

## Msg

Add the monitoring system related messages (!38) · Merge requests · Autoware Foundation / Autoware.Auto / autoware_auto_msgs<br />The goal is to provide messages used by the monitoring system mentioned in<br /><https://gitlab.com/autowarefoundation/autoware.auto/autoware_auto_msgs/-/merge_requests/38/diffs>

## 总体提交

Draft: [#821 Add monitoring system components (!1087) · Merge requests · Autoware Foundation / Autoware.Auto / AutowareAuto<br />Description The goal is to provide the monitoring system mentioned in Autoware<br /><https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/merge_requests/1087>

## MonitorNodeAPI

health_checker<br />[#821] Initial implementation of the monitored node API (!1205) · Merge requests · Autoware Foundation / Autoware.Auto / AutowareAuto<br />Description Monitored Node is a set of API to build ROS2 nodes which will be automatically monitored via the /diagnostic topic....<br /><https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/merge_requests/1205>

## state monitor

Autoware monitoring system: Implement Autoware State Monitor (#1236) · Issues · Autoware Foundation / Autoware.Auto / AutowareAuto<br />Description Purpose<br /><https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/issues/1236><br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671428053470-893ec9a9-38bd-4cc7-bdef-574de9bb1544.png#averageHue=%23fdfbfb&clientId=u75237b64-aac6-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ucddbd648&margin=%5Bobject%20Object%5D&originHeight=874&originWidth=866&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ucce6d8ab-1f4f-4550-8947-05f9f90c7ad&title=)

## ErrorMonitor

Autoware monitoring system: Implement Autoware Error Monitor (#1233) · Issues · Autoware Foundation / Autoware.Auto / AutowareAuto<br />Description graph TD classDef tier_iv fill:#f9f;<br /><https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/issues/1233><br />Draft: [#1233] Error Monitor - Design (!1289) · Merge requests · Autoware Foundation / Autoware.Auto / AutowareAuto<br />Just a draft for the design of the error monitor, created in order to collect feedback. The error monitor will be part of the larger Autoware monitoring system.<br /><https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/merge_requests/1289>

## EmergencyHandler
<https://github.com/tier4/AutowareArchitectureProposal.iv/blob/use-autoware-auto-msgs/system/emergency_handler/README.md><br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671428054001-53e96a2d-4067-4599-9c9c-3a5d55fe98bb.png#averageHue=%234a4a4a&clientId=u75237b64-aac6-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uef304005&margin=%5Bobject%20Object%5D&originHeight=241&originWidth=841&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u92a13e3a-e4ad-463c-b370-b9a3600e9e5&title=)

# 参考

Issues · Autoware Foundation / Autoware.Auto / AutowareAuto<br />Documentation: <https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/><br /><https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/issues?scope=all&search=monitoring&state=all>
