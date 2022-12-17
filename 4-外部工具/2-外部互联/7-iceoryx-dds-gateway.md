# 应用层方案：代理proxy

每当iceoryx新增一个topic，则针对该topic在ros2/dds域内新建一个实体转发该topic

[__https://github.com/ZhenshengLee/rmw_iceoryx/tree/foxy/iceoryx_ros2_bridge__](https://github.com/ZhenshengLee/rmw_iceoryx/tree/foxy/iceoryx_ros2_bridge)

新增一个rosnode，通过rmw_iceoryx接口订阅iceoryx topic，通过rostopic转发出去

# 实践

# 现状

目前rmw_iceoryx和rmw_fastrtps无法交互

目前rmw_iceoryx和rmw_cyclonedds无法交互

iceoryx在rmw_cyclonedds中已经作为一个子模块，可以和iceoryx进行交互

# cyclonedds_iceoryx channel

这种方法在galactic中已经提供，但是性能没有经过充分测试

# rmw_iceoryx with cyclonedds gateway

- Terminal 1: `./build/iox-roudi`

- Terminal 2: `./build/iceoryx_dds/iox-gw-iceoryx2dds` to send all samples from the publisher to DDS

- Terminal 3: `./build/iceoryx_examples/icedelivery/iox-cpp-publisher`

Open another three terminals on machine `B` and execute the commands:

- Terminal 1: `./build/iox-roudi`

- Terminal 2: `./build/iceoryx_dds/iox-gw-dds2iceoryx` to receive all samples from the publisher via DDS

- Terminal 3: `./build/iceoryx_examples/icedelivery/iox-cpp-subscriber`

# 测试

## 构建

需要master分支，或者master backport to v1.01

## 可视化

是否可以订阅

是否可以使用rviz进行数据可视化

是否可以ros2 topic list查看

## 问题

无法去除拷贝

实现零拷贝还是需要rmw_iceoryx

# 参考

[Discussion on future of `iceoryx_ros2_bridge` · Issue #56 · ros2/rmw_iceoryx](https://github.com/ros2/rmw_iceoryx/issues/56)

[iceoryx/README.md at master · eclipse-iceoryx/iceoryx](https://github.com/eclipse-iceoryx/iceoryx/blob/master/iceoryx_dds/README.md)

[https://github.com/eclipse-iceoryx/iceoryx/blob/master/iceoryx_dds/README.md](https://github.com/eclipse-iceoryx/iceoryx/blob/master/iceoryx_dds/README.md)
