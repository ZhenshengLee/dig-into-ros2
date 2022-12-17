# 概述

本文摘要osrf-tsc对dds的测试报告，自Galactic开始，每个版本发布一次<br />报告涉及的测试环境除了研发网络buildfarm，还有无线网络wifi和质量欠佳的网络mininet，此处只关注质量好的有线网络buildfarm<br />在性能中，除了延迟和丢包，还有cpu占用率，内存资源消耗，此处主要关注延迟和丢包<br />报告除了关注性能，还关注代码质量，开源社区活跃度，用户满意度等，此处仅关注代码质量<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663826456588-9bd213e1-e575-4fb4-9e76-e54ea56c689c.png#averageHue=%23729273&clientId=u7f09a805-fa51-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=305&id=u09cd1f90&margin=%5Bobject%20Object%5D&name=image.png&originHeight=609&originWidth=1105&originalType=url&ratio=1&rotation=0&showTitle=false&size=1041065&status=done&style=none&taskId=ua42285f1-e3cb-4391-a59e-9ad445ce66b&title=&width=553)

# 报告思路

## 选择的dds

选择cyclonedds和fastdds的原因

- 在REP中属于TIER 1级别的实现
- 开源许可证
- 符合DDS规范

还有两款商业dds，gurumdds和connextdds没有被osrf选择，而connextdds占据大部分商业份额<br />由于eCAL没有基于RTPS协议遵循DDS<br />事实上：cyclonedds包含其他套件，包括iceoryx和zenoh，iceoryx在零拷贝IPC上有强大性能，zenoh在公网(5G)网络通信上具有强大性能

## 性能验证思路

- 应用程序性能, RMW的总体性能
- 社区欢迎度, 开源用户是否充分利用其性能

## 测试方法概述

- Build Farm Performance Data, 试验场测试，基于简单的网络环境和优化后的环境， 对RMW测试， 测试项包括内存，CPU，丢包
  - <https://github.com/ros2/buildfarm_perf_tests>
- Mininet Performance Data，实际最小网络测试，基于真实环境的最简单环境，这种环境用来测试RMW何时会失败
- Wifi 性能测试。真实场景下的测试，主要测试带宽受限场景。
  - 测试方法：带宽占满，看RMW的发送接受包数量。
  - 不变量为数据大小，变量为发送频率，测试结果为接受到的消息个数
- REP-2004代码质量数据。对代码质量进行测试
- GitHub使用情况。开源社区的欢迎程度
- 用户体验调查。ROS2社区的用户体验

# osrf基准性能测试

这里主要关注build_farm测试结果，，研发环境，单个节点，发送单个消息，单个接受。<br />关于版本，我们测试过程中尽量使用最新版，但是用户一般会使用较老的版本。<br />用户还需要注意影响性能的因素：<br />第一个是具体的实现。<br />第二个是对某项性能的极端要求。<br />最后，本测试的目的是选择默认RMW，所以主要针对的是新的ROS用户的需求，高端用户当然要根据自己的需求选择对应的RMW<br />具体性能还和QOS策略有关。

## buildfarm测试方案

cpu消耗：启动进程spining-无收发消息，查看cpu消耗<br />内存：同步发消息，只发一次

## Galactic选型竞赛(2020测的是Foxy)

- CPU占用，cyclone占优势
- 内存占用，cyclone占优势
- 消息丢包，fastdds占优势，尤其是大数据大于1M
- 消息延时，cyclonedds占优势

In terms of CPU and memory utilization Cyclone DDS RMW performed slightly better. In terms of message latency and messages received both vendors appear to perform well up until approximately the 1MB message size. For messages greater than ~1MB Cyclone DDS RMW has better results with lower latency and the number of messages sent.<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663827269579-e7b97d94-0bcd-45cb-af3f-9c7db1f0cd69.png#averageHue=%23fcfcfc&clientId=u7f09a805-fa51-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1080&id=u2108dba7&margin=%5Bobject%20Object%5D&originHeight=2160&originWidth=1800&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u5d392fb0-f44f-49e5-b15d-2da0a401781&title=&width=900)

## Humble选型竞赛(2021测的是Galactic)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663829447836-11ea0b1f-6e98-4eca-b598-fa7875e56df1.png#averageHue=%23818181&clientId=u7f09a805-fa51-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1080&id=u34dd32a6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=2160&originWidth=1800&originalType=url&ratio=1&rotation=0&showTitle=false&size=298401&status=done&style=none&taskId=uf8b59666-bb93-4979-bbad-f3099c173c2&title=&width=900)

# 供应商对用户关键问题的测试回复

测试平台是ROS2 Galactic

## 大尺寸消息的吞吐量和时延

### 4k图像数据30Hz传输的性能怎么样？

#### CycloneDDS默认配置时延

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663837709069-6929ce4d-f637-4027-ac3f-f38d908fee92.png#averageHue=%23f6f5f5&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=674&id=u02c87619&margin=%5Bobject%20Object%5D&name=image.png&originHeight=674&originWidth=1050&originalType=binary&ratio=1&rotation=0&showTitle=false&size=97371&status=done&style=none&taskId=ucbcfc649-4eef-41cf-8456-b0bbed7fcd2&title=&width=1050)

#### CycloneDDS默认配置吞吐量

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663832580004-0d54314c-c7fa-4a2a-9d74-eb0bfbdffc6b.png#averageHue=%23e2cbcc&clientId=u7f09a805-fa51-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=620&id=u055aaa6a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=620&originWidth=975&originalType=binary&ratio=1&rotation=0&showTitle=false&size=85164&status=done&style=none&taskId=u9710ba63-f11b-4759-b03e-d7929f3d968&title=&width=975)

#### CycloneDDS+Iceoryx零拷贝

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663832701652-cf2bcc77-0901-426b-a1f3-e16b6a6d580a.png#averageHue=%23e0bec0&clientId=u7f09a805-fa51-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=620&id=u9676ae65&margin=%5Bobject%20Object%5D&name=image.png&originHeight=620&originWidth=945&originalType=binary&ratio=1&rotation=0&showTitle=false&size=98206&status=done&style=none&taskId=u7041f104-9a6b-4bd0-b972-e1ecca75ec9&title=&width=945)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663838285622-a75aabe6-556a-4c17-8d16-c5f2845b4880.png#averageHue=%23e3c5c6&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=617&id=uf5ae0ddb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=617&originWidth=962&originalType=binary&ratio=1&rotation=0&showTitle=false&size=99740&status=done&style=none&taskId=u4936470f-0127-4d32-b697-d99a4edb783&title=&width=962)

#### FastDDS默认配置时延和吞吐量

We are taking the results from the tests with**2MB data** size and 30 Hz from our [Benchmarking](https://github.com/ros2middleware/TSC-RMW-Reports/blob/main/humble/eProsima-response.md#benchmarking). The following table compares the average latencies of Fast DDS and Cyclone DDS implementations with default configurations on inter-process and intra-process deployments:

|  | **Inter-process** | **Intra-process** |
| --- | --- | --- |
| Fast DDS | 2.79867 ms | 1.05944 ms |
| Cyclone DDS | 6.114223 ms | 1.70604 ms |

Bear in mind that by default Fast DDS uses asynchronous publishing, while Cyclone DDS uses synchronous publishing. **Having asynchronous publishing by default has the advantage of preventing a blocking behavior in the write operation, which is usually considered critical in many ROS 2 applications**. Refer to [this post](https://discourse.ros.org/t/rmw-fast-dds-publication-mode-sync-vs-async-and-how-to-change-it/17153) for further information. **If any application is sensitive to the latency, it can still reduce it using Fast DDS with synchronous publishing**, that obtains yet lower latencies:

|  | **Inter-process** | **Intra-process** |
| --- | --- | --- |
| Fast DDS (sync) | 2.56477 ms | 0.9638ms |
| Cyclone DDS | 6.114223 ms | 1.70604 ms |

With the default configuration and the given size and frequencies, none of the implementations saturate the throughput. In order to get throughput values, we increased the rate to 1000 messages per second. Here we can see that Fast DDS has much better response than Cyclone DDS in terms of throughput:

- For a single subscriber on inter-process deployments, Fast DDS gets more than **double** the throughput.
- Even though both implementations are able to keep the maximum publication rate in intra-process deployments with a single subscriber, Cyclone DDS's throughput degrades rapidly with increasing number of subscribers (see following questions).
|  | **Inter-process** | **Intra-process** |
| --- | --- | --- |
| Fast DDS | 1999 MBps | 2000 MBps |
| Cyclone DDS | 822 MBps | 2000 MBps |

#### FastDDS零拷贝时延

见文末单独章节

### 数据大小对时延影响

#### FastDDS

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663837995228-30c4b4f4-48cb-4b5b-aeb9-20df76598ed4.png#averageHue=%23fcfbfa&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=623&id=ucaae88b8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=623&originWidth=814&originalType=binary&ratio=1&rotation=0&showTitle=false&size=61268&status=done&style=none&taskId=u804c3218-c18c-4522-bf58-676dbefba48&title=&width=814)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663838069265-4351b933-530b-4442-909f-a6af949f5ca8.png#averageHue=%23fcfbfb&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=622&id=ub85b97e4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=622&originWidth=809&originalType=binary&ratio=1&rotation=0&showTitle=false&size=54114&status=done&style=none&taskId=u0a7f6ec1-5d78-4884-8d27-f97cdf846a7&title=&width=809)

## 通信规模提高对性能的影响

### topic个数提高对时延性能退化影响

#### CycloneDDS测试结果

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835547505-f9466889-43aa-4687-af04-9dcd82eb0896.png#averageHue=%23fbfaf9&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=784&id=u370858ec&margin=%5Bobject%20Object%5D&name=image.png&originHeight=784&originWidth=1059&originalType=binary&ratio=1&rotation=0&showTitle=false&size=104233&status=done&style=none&taskId=u0bee7660-b247-4225-8622-f026b2160ab&title=&width=1059)

#### FastDDS测试结果

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663832381122-902e7e4f-d88c-4fe8-b104-eb26f4e54c68.png#averageHue=%23fbfafa&clientId=u7f09a805-fa51-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=684&id=I8X2b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=684&originWidth=920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=49018&status=done&style=none&taskId=ufd77acf5-35c0-45a4-afff-ee2dd770dad&title=&width=920)

### 节点个数增加对时延性能退化的影响

#### cyclonedds测试

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835586012-8c3dfa31-0a87-4a38-bc1a-3b8eba28f9b5.png#averageHue=%23fbfaf9&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=768&id=ua9c2b894&margin=%5Bobject%20Object%5D&name=image.png&originHeight=768&originWidth=1036&originalType=binary&ratio=1&rotation=0&showTitle=false&size=110375&status=done&style=none&taskId=u66b922f3-bfbd-47cd-9832-ed4b7c5d12f&title=&width=1036)

#### fastdds

测试结果使用的是number of subscriber

### 订阅者(话题Topic)个数增加对时延影响

#### cyclonedds

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663838540041-6a05bc27-8347-4540-945b-25890e4416ea.png#averageHue=%23fbfbfa&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=788&id=uf8aea75f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=788&originWidth=1034&originalType=binary&ratio=1&rotation=0&showTitle=false&size=105088&status=done&style=none&taskId=u027c4ae2-5b93-4f1a-8c56-d1f4017e392&title=&width=1034)

#### fastdds

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663836865009-38b4a587-c4b2-4679-b450-d92571dfa668.png#averageHue=%23fbfafa&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=619&id=u9323e97b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=619&originWidth=837&originalType=binary&ratio=1&rotation=0&showTitle=false&size=39916&status=done&style=none&taskId=u3071379a-2986-4b09-85b8-db3df24114d&title=&width=837)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663836874219-b4ce6fd1-02ab-4a5c-a413-551fc6b6aeb0.png#averageHue=%23fbfafa&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=632&id=u7b483236&margin=%5Bobject%20Object%5D&name=image.png&originHeight=632&originWidth=826&originalType=binary&ratio=1&rotation=0&showTitle=false&size=41664&status=done&style=none&taskId=u0c317e7c-ff8d-47c8-acb4-fdeaec3fd45&title=&width=826)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663837856493-86331eea-6b1f-4445-8941-9397af168103.png#averageHue=%23fcfafa&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=593&id=u9bd3789f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=593&originWidth=795&originalType=binary&ratio=1&rotation=0&showTitle=false&size=67196&status=done&style=none&taskId=u9622364a-d30e-4b07-8d27-b88488889c4&title=&width=795)

### 订阅者个数增加对吞吐量影响

#### CycloneDDS

缺失

#### FastDDS

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663836920049-ab4440d2-2db5-4ba5-958b-b5d4dbadf2e4.png#averageHue=%23fdfcfb&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=602&id=uf97ad8bc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=602&originWidth=812&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38540&status=done&style=none&taskId=u0f5d29e1-9138-4f19-9e1c-058c4ba0169&title=&width=812)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663836930358-be2c8054-aff9-4e28-892d-fbed25a3f19c.png#averageHue=%23fdfcfb&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=616&id=u318154c9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=616&originWidth=820&originalType=binary&ratio=1&rotation=0&showTitle=false&size=53455&status=done&style=none&taskId=u9ad4f19e-9148-458c-bebe-a8fbea48ab6&title=&width=820)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663838029560-7700ea40-7e9f-45ef-b757-4eea3bfb7763.png#averageHue=%23fcfbfb&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=598&id=u4ee82e5f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=598&originWidth=829&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63690&status=done&style=none&taskId=u10c4f278-023e-458a-b9f0-4eed5c884ff&title=&width=829)

### 话题频率增加对时延影响

#### cyclonedds

缺失

#### FastDDS

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663837912317-0b10803a-138a-44ed-b8d8-f9166b438f9b.png#averageHue=%23fcfbfa&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=599&id=u1d0d4917&margin=%5Bobject%20Object%5D&name=image.png&originHeight=599&originWidth=810&originalType=binary&ratio=1&rotation=0&showTitle=false&size=62109&status=done&style=none&taskId=ucee9ee5d-03d8-468f-94f4-565b6d07b69&title=&width=810)

# 总体性能论证

## 两个问题

各种通信机制下的吞吐量测试？<br />Please provide benchmarks for inter-host, inter-process (with and without LoanedMessages), and intra-process (with and without LoanedMessages) throughput for both large and small message sizes, on both Linux and Windows.<br />通信机制包括：inter-host, inter-process (with and without LoanedMessages), and intra-process (with and without LoanedMessages)<br />数据大小包括：both large and small message sizes<br />Also note that Fast DDS uses Boost.interprocess to implement shared memory and data sharing deliveries. <br />单个进程下的1:1通信时延，吞吐量等性能如何受topic频率和大小的影响？<br />For a pub/sub pair in separate processes, what is the average round-trip time, throughput, and CPU/memory utilization? How does this scale with topic frequency and topic size?

## Cyclonedds总体回复

Following is the summary overview of the results for all tests, all platforms. How to instructions, test scripts, raw data, tabulated data, tabulation scripts, plotting scripts and detailed test result PDFs for every individual test are [here](https://github.com/osrf/TSC-RMW-Reports/tree/main/humble/eclipse-cyclonedds-report). Also see [Appendix B: Performance Summary Per Platform](https://github.com/ros2middleware/TSC-RMW-Reports/blob/main/humble/eclipse-cyclonedds-report.md#appendix-b-performance-summary-per-platform).<br />For 569 tests with varying message types, sizes, rates, subscribers, QoS, and platforms the result was the following (also see the graphs below the bullet list):

- Cyclone DDS has lower latency in 443 tests
- Cyclone DDS has lower latency jitter in 447 tests
- Cyclone DDS has higher throughput in 106 of 141 tests with other failing 37 of the 141 throughput tests
- Cyclone DDS has lower memory utilization in 451 tests
- Cyclone DDS has lower CPU utilization in 484 tests (387 limited + 97 unlimited)
- Cyclone DDS scaled better by number of topics and nodes
- Cyclone DDS failed 4 tests while the other middleware failed 63 tests
- Cyclone DDS failed 4 tests were Windows 10 default network buffer size, large messages, 500Hz

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663839052339-35d216a4-1134-436a-bd36-a1699d2dce85.png#averageHue=%23f8f8f7&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=416&id=u9111bb99&margin=%5Bobject%20Object%5D&name=image.png&originHeight=416&originWidth=1091&originalType=binary&ratio=1&rotation=0&showTitle=false&size=109944&status=done&style=none&taskId=uf86d32f0-5267-460d-a999-5124dc9bc93&title=&width=1091)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663839061723-0113e8cf-a79a-42ac-ba7a-f9a99252c9d2.png#averageHue=%23f8f8f8&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=428&id=ue2647bbb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=428&originWidth=1069&originalType=binary&ratio=1&rotation=0&showTitle=false&size=93465&status=done&style=none&taskId=uc8267321-dbc1-447c-80b3-c3a8b364b30&title=&width=1069)

### Cyclone DDS with iceoryx Improvement

Cyclone DDS with built-in iceoryx zero-copy was tested using the rclcpp RMW “LoanedMessage” API. As you can see below Cyclone DDS with iceoryx improves upon Cyclone DDS in every measure: latency, jitter, throughput, memory, total CPU usage, relative CPU per message. The instructions to use zero-copy with Cyclone DDS + iceoryx are [here](https://github.com/ros2/rmw_cyclonedds/blob/galactic/shared_memory_support.md). Software engineers at ADLINK & Apex.AI followed the other middleware's published instructions to use LoanedMessage API, but the other middleware failed these tests without getting any messages through.<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663839071940-38ea9f9c-97d0-4e38-a75c-3019da3856cb.png#averageHue=%23f9f9f9&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=419&id=u83ca5459&margin=%5Bobject%20Object%5D&name=image.png&originHeight=419&originWidth=1056&originalType=binary&ratio=1&rotation=0&showTitle=false&size=86608&status=done&style=none&taskId=u78c05d32-a9eb-49d4-b35b-e29e3abd802&title=&width=1056)

## FastDDS总体回复

### 总体意见

The general conclusions can be summarized as follows:

- Fast DDS has much better scallability with the data size than Cyclone DDS, in latency, CPU usage and throughput. For 2MB data sizes, Fast DDS has double the throughput, half the latency and half the CPU usage. This with the default configuration. Using data-sharing results are much better yet.
- Fast DDS consistently gets **at least double** the throughput than Cyclone DDS, with all configurations.
- For smaller data sizes, Latency with Fast DDS synchronous mode configurations is better than latency with Cyclone DDS. As expected, latency with asynchronous mode is worse.
- Memory consumption in Fast DDS is higher than in Cyclone DDS. This was somehow expected, since Fast DDS supports many more configurations and features that require data structures residing in memory.

### 时延

- Fast DDS (on synchronous mode) has consistently lower latencies than Cyclone DDS.
- Specially with large data sizes, where Cyclone DDS has **double** latencies than Fast DDS (without data-sharing).
- We extended some tests on the Windows platform beyond 2MB that confirm that latency in Cyclone DDS increases at a much larger rate than in Fast DDS.
- Results are even better if using Fast DDS data-sharing delivery, especially on large data sizes, since data copies are avoided. Total latency is **a quarter** of the latency of Cyclone DDS.
- Latencies seem to increase at a similar rate with the number of subscribers.
- Surprisingly, in all implementations the latency falls down with increasing publication rates. We believe this is a consequence of the process becoming idle on lower rates and the platform's scheduler changing to another job, so that when the next sample is sent, it has to wait until a running slot is available.

### 吞吐量

In order to get throughput values, we used the tests for 2MB data size and a rate of 1000 messages per second.

- Fast DDS consistently gets **at least double** the throughput than Cyclone DDS, with all configurations.
- For example, all tested Fast DDS configurations can keep with the required publication rate of 2000 Mbps with one subscriber, while Cyclone DDS does not get to half that rate.
- Fast DDS shared-memory delivery is the implementation that is least affected by the number of subscribers. It is able to keep the required 2000 Mbps rate up to 10 subscribers.
- We extended some tests on the Windows platform beyond 2MB that confirm that throughput in Cyclone DDS decreases at a much larger rate than in Fast DDS.

# FastDDS零拷贝IPC测试(Sony)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835249832-c1df75b1-b413-4432-8448-d9927aef7348.png#averageHue=%23f9f9f8&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=525&id=u9b6188e6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=525&originWidth=921&originalType=binary&ratio=1&rotation=0&showTitle=false&size=88392&status=done&style=none&taskId=u7155710e-da79-4bc4-a78f-44da5fd16e8&title=&width=921)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835267043-6757dc8c-89fe-4bc0-ab78-63d3479db2e9.png#averageHue=%23fbfbfa&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=743&id=u82ec7b2e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=743&originWidth=935&originalType=binary&ratio=1&rotation=0&showTitle=false&size=89776&status=done&style=none&taskId=u7a7c82a8-c3ea-433a-9349-11f1d26e83f&title=&width=935)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835282532-fc7981e9-26cc-4936-a20a-ed84cdedd376.png#averageHue=%23f9f9f8&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=442&id=udb58f74b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=442&originWidth=918&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74674&status=done&style=none&taskId=ub6c3d94a-8739-4edd-85d0-556bfd6c0c0&title=&width=918)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835301338-a972ad75-2cc2-471c-8925-dc2241d99062.png#averageHue=%23fbfafa&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=832&id=uac467357&margin=%5Bobject%20Object%5D&name=image.png&originHeight=832&originWidth=925&originalType=binary&ratio=1&rotation=0&showTitle=false&size=106368&status=done&style=none&taskId=u4bffc2ac-0596-4e38-9909-f02d7eb1291&title=&width=925)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835314831-4941cb48-9d44-4030-b99b-055890546671.png#averageHue=%23f9f8f8&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=353&id=u8940118e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=353&originWidth=921&originalType=binary&ratio=1&rotation=0&showTitle=false&size=59327&status=done&style=none&taskId=u87b866ac-5acc-4a86-87f6-c42e58995b8&title=&width=921)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835344651-3f910d1d-b6ae-4a40-9161-8aff621494d6.png#averageHue=%23fbfafa&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=911&id=u077b99d2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=911&originWidth=926&originalType=binary&ratio=1&rotation=0&showTitle=false&size=118880&status=done&style=none&taskId=ud713716f-df06-44e8-a5b4-52cf4780096&title=&width=926)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835356452-215fda9d-a9ba-4d8c-b2e8-f282bb0218f9.png#averageHue=%23f9f8f7&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=245&id=u624f9afc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=245&originWidth=925&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44982&status=done&style=none&taskId=ud1db5b44-df5f-44df-8a77-43cd96d00b6&title=&width=925)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835373936-feb16aae-48b3-4a5b-ba71-6eb871ad6bc1.png#averageHue=%23f9f9f8&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=683&id=u9a058d62&margin=%5Bobject%20Object%5D&name=image.png&originHeight=683&originWidth=934&originalType=binary&ratio=1&rotation=0&showTitle=false&size=116032&status=done&style=none&taskId=ua6c72feb-f2ae-4a1f-9f17-875ca1befc8&title=&width=934)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663835383590-9a2d2e51-cf02-4ac7-938b-d7b77fe609b6.png#averageHue=%23fdfdfd&clientId=ua6c18c64-1ba0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=335&id=u22890fdc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=335&originWidth=910&originalType=binary&ratio=1&rotation=0&showTitle=false&size=21122&status=done&style=none&taskId=u04c4f697-1227-4650-92ad-c7ee8ea9b26&title=&width=910)

# 供应商总结

## CycloneDDS总结

Many have adopted the Eclipse Cyclone DDS ROS middleware after performing their technical due diligence of the available ROS middleware implementations. Adopters include:

- Default ROS middleware for [ROS 2 Galactic](https://discourse.ros.org/t/ros-2-galactic-default-middleware-announced/18064)
- Most [Nav2](https://github.com/ros-planning/navigation2) users run Eclipse Cyclone DDS per the Nav2 WG Leader
- Default ROS middleware for [Autoware Foundation](https://www.autoware.org/autoware-auto) developers
- Default ROS middleware for [Apex.OS](https://www.apex.ai/apex-os)
- Default ROS middleware for [Indy Autonomous Challenge](https://www.indyautonomouschallenge.com/) Base Vehicle Software WG
- Default ROS middleware for [SVL Simulator](https://www.svlsimulator.com/)
- Default ROS middleware for [SOAFEE](http://soafee.io/) "Sophie" Scalable Open Architecture For Embedded Edge open source with Arm, ADLINK, Apex.AI, AutoCore, Capgemini Engineering, Continental, CARIAD, Green Hills Software, Linaro, Marvell, MIH Consortium, Red Hat, SUSE, Tier IV, Volkswagen, Woven Planet (Toyota Research), Zing Robotics as explained [here](https://www.arm.com/company/news/2021/09/new-arm-technologies-to-transform-the-software-defined-future-for-the-automotive-industry). ZDNet explains SOAFEE [here](https://www.zdnet.com/article/arm-lays-the-groundwork-for-the-software-defined-vehicle/).
- [Eclipse Cyclone DDS adopters](https://iot.eclipse.org/adopters/#iot.cyclonedds), [Eclipse iceoryx adopters](https://iot.eclipse.org/adopters/#technology.iceoryx), [Eclipse Zenoh adopters](https://iot.eclipse.org/adopters/#iot.zenoh)

## FastDDS总结

Unsurprisingly, the performance of the tested configurations depend on the kind of deployment: inter or intra-process, best effort or reliable, the number of subscribers, the data size, and so forth. However, in most cases the differences among the implementations and configurations can be considered non-significant, and the effect of the number of subscribers, publication rate and data size is similar in most cases too. There are, however, some exceptions that show really big differences, and these are the ones we are highlighting here. For a more detailed discussion about the smaller differences, please refer to the complete report.

- Cyclone DDS has very poor performance with large data sizes. In the case of 2MB data:
  - Latency is at least 50% larger than Fast DDS without data-sharing.
  - CPU usage is at least 50% larger than Fast DDS without data-sharing.
  - We extended some tests on the Windows platform beyond 2MB that confirm that latency in Cyclone DDS increases at a much larger rate than in Fast DDS.
- **By large, the best option for large data sizes is Fast DDS with data-sharing**:
  - On a linux platform the throughput exceeds 2000 MBps, even with 10 subscribers.
  - On a Raspberry platform, it has still 10 times larger throughput than any other configuration.
  - The effect of the data size on the latency is almost negligible compared to the other configurations.
- The throughput with Cyclone DDS is the poorest by a large margin.
  - Fast DDS without data-sharing has around 4 times better throughput regardless of the number of subscribers.
  - We extended some tests on the Windows platform beyond 2MB that confirm that throughput in Cyclone DDS decreases at a much larger rate than in Fast DDS.
  - As mentioned before, **the best throughput is achieved with Fast DDS with data-sharing delivery**.
- Fast DDS consumes around 25% more memory in all its configurations.This was somehow expected, since Fast DDS supports many more configurations and features that require data structures residing in memory. This is also why all Fast DDS configurations have similar memory requirements

# 分析

### 为什么回答4k图像数据传输，都用的是2M的基准测试数据

如果无损传输，则(24/8) *3840* 2160 bytes or about 24.9MB<br />建议使用image_compressed方式传输图像，使用压缩比较高的格式<br />jpeg: 10到40的压缩比<br />[https://wiki.ros.org/compressed_image_transport](https://wiki.ros.org/compressed_image_transport)<br />该问题需要继续调查<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664163164361-cc6d9f51-504d-4b77-a566-70a854428f66.png#averageHue=%23fefbfa&clientId=ud18288a3-af53-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=712&id=uaf57f52b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=712&originWidth=920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=75476&status=done&style=none&taskId=u4544df1e-8551-4723-b756-29e599f10c3&title=&width=920)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664163216741-4fec6f1c-b0c6-4378-bcff-ec4671b8c6f7.png#averageHue=%23f5efef&clientId=ud18288a3-af53-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=430&id=ue99fe32b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=430&originWidth=832&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60942&status=done&style=none&taskId=uf7320e81-a58f-4fb5-b199-76a89f0d5bd&title=&width=832)

### 性能对比维度太多，怎么做性能对比呢？

- 基础易用性测试，可以到网上提取公开测试结果
- 针对自己项目的特定场景，提取关键Case，进行测试

### 当性能相当时，对dds选型的考虑是什么？

易用性，信息安全，外部工具的可用性，API友好性等等

# 初步结论

摘要翻译osrf的性能测试报告，梳理cyclonedds和fastdds的性能测试报告，认为<br />1）单个消息的尺寸应小于8MB<br />2）理论吞吐量为2000MB/s，实际使用过程中的吞吐量远远小于2000MB/s <br />3）应该根据实际情况对DDS进行详细的配置以获取最佳性能，不应使用默认配置

# 参考
<https://osrf.github.io/TSC-RMW-Reports/>
