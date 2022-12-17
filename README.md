# Dig-into-ros2 - 深入理解ROS2

Dig into ROS2 is mainly to help learning the ROS2 core system, deeply.

- The basic steps will not be included.
- The driver, algorithm, and apps will not be included.

## contents

```sh
date && tree -d -L 3
Sat Dec 17 11:08:16 AM CST 2022
.
├── 0-SOA概念
│   └── pdf
├── 10-MicroROS
├── 1-RCL
│   ├── 0-概念
│   ├── 1-rcl接口
│   ├── 2-rcl实现
│   ├── 3-rclcpp通信管理
│   ├── 4-rclcpp执行管理
│   ├── 5-rclcpp节点管理
│   ├── 6-rclcpp实现
│   ├── 7-rclcpp扩展
│   └── 8-高级话题
├── 2-RMW
│   ├── 0-dds规范
│   ├── 1-dds使用
│   │   ├── 1-cyclonedds
│   │   ├── 2-iceoryx
│   │   ├── 3-fastdds
│   │   ├── 4-ecal
│   │   └── 5-connextdds
│   ├── 2-dds设计
│   │   ├── 1-cyclonedds
│   │   ├── 2-iceoryx
│   │   ├── 3-fastdds
│   │   ├── 4-ecal
│   │   └── 5-connextdds
│   ├── 3-rmw使用
│   │   ├── 1-cyclonedds
│   │   ├── 2-iceoryx
│   │   ├── 3-fastdds
│   │   ├── 4-ecal
│   │   └── 5-connextdds
│   ├── 4-rmw接口
│   ├── 5-rmw实现
│   │   ├── 0-总体设计
│   │   ├── 1-cyclonedds
│   │   ├── 2-iceoryx
│   │   ├── 3-fastdds
│   │   └── 4-ecal
│   ├── 6-rosidl接口
│   ├── 7-rosidl实现
│   └── 8-DataExchangeFormat
├── 3-内部工具
│   ├── 1-构建系统
│   │   ├── 1-colcon工具
│   │   ├── 2-ament系统
│   │   └── 3-打包发布
│   ├── 2-软件组件
│   │   ├── 1-ros2parameter
│   │   ├── 2-ros2cli
│   │   ├── 3-ros2launch
│   │   ├── 4-ros2logging
│   │   └── 8-misc
│   └── 3-机器人工具
│       ├── 1-rosbag2
│       ├── 2-tf2
│       └── 8-misc
├── 4-外部工具
│   ├── 1-扩展组件
│   └── 2-外部互联
├── 5-系统性能
│   ├── 0-LTTng
│   ├── 1-app-tracing
│   ├── 2-DDS-tracing
│   ├── 3-Kernel-tracing
│   ├── 4-testing-platform
│   │   ├── 1-buildfarm
│   │   ├── 2-apex-performance-test
│   │   ├── 3-reference-system
│   │   ├── 4-pendulm-test
│   │   └── 5-osrf-tsc-report
│   ├── 5-dds-performance
│   │   ├── 1-系统配置
│   │   ├── 2-零拷贝特性
│   │   ├── 3-测试报告
│   │   └── 4-misc
│   ├── 6-rmw-performance
│   ├── 7-rclcpp-performance
│   ├── 8-jetson-performance
│   └── 9-tools
├── 6-实时系统
│   ├── 0-Concepts
│   ├── 1-SOC
│   ├── 2-RTOS
│   │   ├── 1-系统选型
│   │   ├── 2-OSes
│   │   └── 3-Techniques
│   ├── 3-DDS
│   ├── 4-RCL
│   ├── 5-BSW
│   │   ├── 1-memory
│   │   └── 2-logging
│   ├── 6-Tool
│   └── 7-App
├── 7-功能安全
│   ├── 0-技术咨询
│   ├── 1-流程认证
│   ├── 2-产品认证
│   ├── 3-人员培训
│   ├── 8-信息安全
│   │   ├── 1-信息安全基础
│   │   ├── 2-信息安全模型
│   │   ├── 3-ROS2信息安全方案
│   │   ├── 4-SROS2
│   │   └── pdf
│   └── 9-法律法规标准
├── 8-算法安全
│   ├── 1-流程和工具
│   ├── 2-软件技术
│   ├── 3-可复用库
│   └── 4-实践指南
└── 9-标准遵循
    ├── 1-ROS实现标准
    │   ├── 1-ROS-REP
    │   ├── 2-ROS-REP-Plus
    │   ├── 3-AutowareGuideline
    │   └── 4-ROS-Patterns
    ├── 2-AUTOSAR-AP
    │   ├── 0-总体需求
    │   ├── 1-总体架构
    │   ├── 2-内部数据接口规约
    │   ├── 3-外部数据接口规约
    │   ├── 4-通信协议接口规约
    │   ├── 5-通信管理
    │   ├── 6-线程模型
    │   ├── 7-生命周期管理
    │   ├── 8-构建部署
    │   └── 9-开发规范
    └── 3-OMGDDS
        └── pdf

131 directories

```

## getting started

## layout

the lay out of this repo is mainly inspired by <https://github.com/daohu527/dig-into-apollo>, and also take the following projects as examples

- <https://github.com/ros2/ros2_documentation>
- <https://github.com/ros2/design>
- <https://github.com/ros-realtime/ros-realtime.github.io>
