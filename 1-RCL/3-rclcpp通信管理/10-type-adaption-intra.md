# 概述

对于ROS2应用来讲，msg-conversion一般不可省略，就算是Intra-process也不一定<br />type-adaption可以简化msg-conversion，避免过多的拷贝<br />type-adaption-opt可以避免序列化，进而真正避免拷贝，Humble还没实现

# intra-process-type-adaption

## 原理

通过设计intra-process manager，如果存在type-adaption且intra-process过程，则避免进行type-conversion<br />[https://github.com/ros2/rclcpp/pull/1849](https://github.com/ros2/rclcpp/pull/1849)<br />For context, the point of this PR is to enhance [TypeAdaptation](https://ros.org/reps/rep-2007.html) so that it avoids unnecessary conversions. In particular, if a Publisher and Subscription are both in the same context, using intra-process communications, and use the same custom TypeAdapted type, **then we can just place the published data directly from the publisher into the Subscription queue for delivery without doing any conversions**. If any of several other situations happen (Publisher is ROS message, Subscription is custom, Publisher is custom, Subscription is ROS message, etc), then we do the conversion as necessary.<br />**This has two primary benefits, both related to performance:**<br />**两个好处，都是关于性能增强的**

1. We can avoid doing the (possibly) expensive and unnecessary to a ROS type on the publisher, and then from the ROS type on the subscription.
   1. 避免类型转换，也就是避免了多余的拷贝
2. We can use a custom type to hold onto custom "handles", allowing the Publisher to pass those handles directly to the Subscription. Those handles might be handles to hardware accelerators, which means that the system can use the hardware accelerator to do the conversion without synchronizing with the CPU to publish to the next node.
   1. 可以保存custom "handlers"，节点间可共享任何类型的数据（之前必须是msg定义的类型），这些handlers可以是指针，进程内共享的指针，可以是CUDA设备内存地址

## 实现

当前功能已集成到rclcpp-humble<br />[https://github.com/ros2/rclcpp/pull/1849](https://github.com/ros2/rclcpp/pull/1849)

## 当前缺陷

当multisub包含IPC的时候，intra-process-typeadaption的通信链路性能也会受影响<br />[https://github.com/ros2/rclcpp/issues/1860](https://github.com/ros2/rclcpp/issues/1860)

## 未来实现

未来的实现计划去掉serdes<br />[https://github.com/ros2/rclcpp/issues/1665](https://github.com/ros2/rclcpp/issues/1665)

# image-container-cuda-intra-process

有了上述定义，对于拥有一个cuda内存句柄，在进程内部的节点间共享，避免cpu到gpu的内存拷贝，是非常有意义的！<br />![](https://cdn.nlark.com/yuque/0/2022/gif/23125517/1664448390902-1aed7ff3-4315-4102-9ac7-a49f30051e96.gif#clientId=ub1bbbb21-0fd2-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ud5de094b&margin=%5Bobject%20Object%5D&originHeight=450&originWidth=800&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ue4083b6d-7684-4d59-aa50-5b83fe9f72a&title=)

# 讨论

其最大的优点在于降低了难度，在ros的通信框架下做了进程内cuda内存指针的共享

- 在ROS的通信框架，意味着可以方便进行ros2 topic echo，只不过通信时延会很大（如果是IPC，则还是要做CUDA内存拷贝的工作）
- 利用节点机制，享受节点的各种优势

进程内通信共享句柄的方法，如果不使用ros2机制的话，可以采用

- 使用通用cpp进行进程内多线程开发，也是可以的，但是难度显然比这个大
- 定义一个msg，内部包含一个cuda句柄，在节点间共享，这也是OK的，因为句柄在进程内部有效

## cuda内存句柄在进程间有效的方法？

CUDA内存拷贝的工作在IPC下是不可避免的？<br />之前介绍了一种技术，cuda-ipc，app1可以获取cuda_mem_handle，传递给app2，app2根据这个handle访问cuda的同一块内存<br />[https://forums.developer.nvidia.com/t/gpu-inter-process-communications-ipc-question/35936/5](https://forums.developer.nvidia.com/t/gpu-inter-process-communications-ipc-question/35936/5)<br />这种技术初看很美好，但是有巨大的缺陷，没有将app1和app2完全解耦，使得该技术集成到ros2中很困难<br />其app1，及handle的创建者必须长期存在，当app1关闭，则context失效，app2也无法访问了<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664450425846-d361072e-a8f5-465b-be6c-5c5f1749e218.png#clientId=u1556d302-1221-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=159&id=uf78b1a53&margin=%5Bobject%20Object%5D&name=image.png&originHeight=159&originWidth=1250&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38774&status=done&style=none&taskId=u6a719f13-ef48-4ec7-adea-367b2740f75&title=&width=1250)<br />如果publisher使用了该handle技术，则当publisher意外关闭时，cuda_context失效，subscriber恰好在回调阶段使用了handle，则subscriber也会崩溃

# 性能测试

| 类型 | 传输时延 | Talker |  |  | Listener |  | 进程内共享<br />数据类型 |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  |  | data<br />cpu2gpu | data<br />gpu2cpu | convert_to_ros | convert_to_custom | data<br />cpu2gpu |  |
| intra_image_node | 3.2ms |  | <br /> | ✓ | ✓ |  | sensor_msg_image |
| intra_gmat_node | 4.7ms | <br /> | ✓ | ✓ | ✓ | ✓ | sensor_msg_image |
| mat_image_node | 0.15ms | <br /> | <br /> | <br /> | <br /> | <br /> | cv::Mat |
| mat_image_node-multisub | 1.8ms |  |  | ✓ | <br /> |  | cv::Mat |
| cuda_image_node | 1.2ms | ✓ |  |  |  |  | cuda_raw_pointer |
| cuda_image_node-multisub | 3.0ms | ✓ | ✓ | ✓ |  |  | cuda_raw_pointer |
| gmat_image_node | 1.2ms | ✓ |  |  |  |  | cv::cuda::GpuMat |
| gmat_image_node-multisub | 3.5ms | ✓ | ✓ | ✓ |  |  | cv::cuda::GpuMat |
| 代码：<br />[https://github.com/ZhenshengLee/ros2_shm_msgs/tree/humble/cuda](https://github.com/ZhenshengLee/ros2_shm_msgs/tree/humble/cuda)<br />[https://github.com/ZhenshengLee/ros2_shm_msgs/tree/humble/intra](https://github.com/ZhenshengLee/ros2_shm_msgs/tree/humble/intra)<br />测试环境：<br />gpu云桌面，22.04docker镜像，opencv4.6.0-cuda<br />1920X1080 图像，1Hz，Keeplast(5), besteffort<br />注意：当multisub包含IPC的时候，intra-process-typeadaption的通信链路性能也会受影响<br />参见[https://github.com/ros2/rclcpp/issues/1860](https://github.com/ros2/rclcpp/issues/1860)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1666076738630-ccc3a8d6-dcda-4557-94d5-c88358760c5f.png#clientId=ub1ae9627-42c1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=328&id=ptthb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=437&originWidth=641&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71619&status=done&style=none&taskId=ucab763b5-ba4a-4cf0-912f-0193de610d9&title=&width=481) |  |  |  |  |  |  |  |

# 结论

对于cuda视觉应用，目前intra-process-type-adaption是最佳方案

# 参考

[https://github.com/ros2/rclcpp/pull/1849](https://github.com/ros2/rclcpp/pull/1849)<br />[https://github.com/ros2/ros2_documentation/issues/2377](https://github.com/ros2/ros2_documentation/issues/2377)<br />[https://github.com/ros2/rclcpp/issues/1664](https://github.com/ros2/rclcpp/issues/1664)<br />[https://discourse.ros.org/t/new-type-adaptation-open-source-examples-for-humble/26241](https://discourse.ros.org/t/new-type-adaptation-open-source-examples-for-humble/26241)<br />[https://developer.nvidia.com/blog/improve-perception-performance-for-ros-2-applications-with-nvidia-isaac-transport-for-ros/](https://developer.nvidia.com/blog/improve-perception-performance-for-ros-2-applications-with-nvidia-isaac-transport-for-ros/)
