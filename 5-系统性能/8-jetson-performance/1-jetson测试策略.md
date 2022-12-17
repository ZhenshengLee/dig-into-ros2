# 测试策略

## 场景

不考虑

- 不考虑无线场景

- 不考虑跨域DDS

- 不考虑MEM占用

重点关注

- 考虑大文件丢包

- 考虑CPU占用

- 关注不同工况下高频大尺寸传输

- QoS对dds性能影响

## 工况

## 数据大小

点云一帧0.3M, 10hz, 吞吐量 3M/s,   点云4倍

图像一帧2M, 24hz, 吞吐量48M/s， 图像6倍

## Qos

传感器工况

指令工况

# 测试工具

## 基准测试

osrf buildfarm test

apex performance test: 对两个节点的消息通信进行配置

此处选择apex的工具，增加

## 系统测试

trancingtools_analysis: 测试回调执行时间等指标

这个是通用的

另外还有两个reference_system和pendimun_test是有针对性的基准测试
