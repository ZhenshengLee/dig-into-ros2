# ROS2框架概述

## 静态视图

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664518187787-e11ca48a-64c4-42d5-84bd-1bdc769cc93c.png#averageHue=%23e8d5bf&clientId=ueeddd5ec-62a8-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u6821b34f&margin=%5Bobject%20Object%5D&originHeight=522&originWidth=709&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ubbf6faa6-0eb7-47f9-aeb9-f6078030e11&title=)<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664518345309-1f1b9634-5558-42cd-aa9a-80adc81535ad.png#averageHue=%23eaeaea&clientId=ueeddd5ec-62a8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=266&id=OaNan&margin=%5Bobject%20Object%5D&originHeight=355&originWidth=823&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ud03a5f06-e4c4-407f-8fbd-714dff91f4e&title=&width=617)<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664518273391-823b7611-a699-42af-ac77-ed1c25588ad3.png#averageHue=%23e8e8e8&clientId=ueeddd5ec-62a8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=218&id=F6t0M&margin=%5Bobject%20Object%5D&originHeight=291&originWidth=1004&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u46fadded-ccea-492e-97c2-58ac9cb2798&title=&width=753)

## 动态视图

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664518330361-c317c202-1b01-4952-9ea0-6654eebc84b3.png#averageHue=%23fbfaf8&clientId=ueeddd5ec-62a8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=291&id=ue13bc44e&margin=%5Bobject%20Object%5D&originHeight=388&originWidth=1134&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u02c66a54-903e-459f-bf7d-a8e859ed6a0&title=&width=851)<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664520026326-2d01c92f-345a-4a62-b8d4-3d6aba4fe4a4.png#averageHue=%231c1c1c&clientId=ueeddd5ec-62a8-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u1b80f9d9&margin=%5Bobject%20Object%5D&originHeight=506&originWidth=794&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ud42e3894-efe9-4e87-a5e6-d406b67a83c&title=)

# 传输过程分析

## data-construction

### 介绍

数据的获取有两种方式：

- 从IO（外设）获取数据如v4l2子系统
- 从IO（文件系统）获取数据如文件
- 从订阅获得，executor通过rmw_take获取，调用回调函数，存储在进程缓冲区

不管是何种方式，此时data的格式往往不是ros-msg格式，举例，为cv::Mat格式

```cpp
// Capture a frame from OpenCV.
cv::VideoCapture cap_;
cv::Mat frame_;
cap_ >> frame_;
// 从文件获取图像
std::shared_ptr<cv_bridge::CvImage> m_input_cvimage{std::make_shared<cv_bridge::CvImage>()};
m_input_cvimage->image = cv::imread("./res/img/1920_1080.jpg");
```

### 拷贝分析

不管怎样，每一次都有一次（需具体分析）内存拷贝的过程，从IO设备到内存。

### 讨论

#### v4l2的摄像头数据采集发生了多少次拷贝？

#### 从感光单元采集到数据的时刻，到获取image的时延是多少？

## convert_to_ros

### 介绍

为了进行IPC通信，需要将数据转换为特定的格式，进行序列化后进行网络通信以获得最佳性能。<br />DDS为预定义的数据格式提供了工具函数用作数据定义，序列化，类型特征识别等。这些是通过对msg/idl接口描述文件进行编译后得到的。<br />为了将data转换为预定义格式，需要转换函数，在这个转换过程中，可能产生拷贝

### 拷贝分析

是否发生拷贝要根据转换函数来分析，一般来讲会发生一次拷贝<br />galactic\src\ros-perception\vision_opencv\cv_bridge\src\cv_bridge.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664522379629-2a4c481c-7d3c-42c2-a9ae-052cf0b4bd19.png#averageHue=%23252a32&clientId=u06e8da3d-34a7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=458&id=u48961c38&margin=%5Bobject%20Object%5D&name=image.png&originHeight=458&originWidth=655&originalType=binary&ratio=1&rotation=0&showTitle=false&size=62231&status=done&style=none&taskId=u7ba0900a-7ac8-4e02-8494-3a2f54f2192&title=&width=655)

### 讨论

#### cv-bridge的toImageMsg()转换过程的拷贝是否可避免？

提issue

#### 通用的零拷贝类型转换策略是什么？

能否为cv_bridge提供代码？

#### intra-process下能否使用cv_bridge的功能？

[https://github.com/ros-perception/vision_opencv/issues/413](https://github.com/ros-perception/vision_opencv/issues/413)

#### 转换成compressed_image的过程是否发生拷贝？

需要分析image_transport库的代码

## serialization

### 介绍

为了进行IPC通信，需要将数据转换为特定的格式，进行序列化后进行网络通信以获得最佳性能。<br />DDS为预定义的数据格式提供了工具函数用作数据定义，序列化，类型特征识别等。这些是通过对msg/idl接口描述文件进行编译后得到的。<br />这些工具库通过so提供，被rmw动态调用<br />如下图所示，提供c, py, cpp，接口，并提供针对不同rmw的适配，在include中有相应的头文件<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664520728350-d2200be4-8a63-43ab-874d-4a2bc7f95212.png#averageHue=%2320242b&clientId=ueeddd5ec-62a8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=259&id=Vsvpq&margin=%5Bobject%20Object%5D&name=image.png&originHeight=259&originWidth=401&originalType=binary&ratio=1&rotation=0&showTitle=false&size=21443&status=done&style=none&taskId=ud0630451-a429-4957-8045-13b591bf90b&title=&width=401)<br />这些功能是由下列包提供的<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664521213694-877f4384-662d-4695-b92f-bb42d7a65656.png#averageHue=%2322282e&clientId=ueeddd5ec-62a8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=150&id=WMI9Z&margin=%5Bobject%20Object%5D&name=image.png&originHeight=150&originWidth=389&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9073&status=done&style=none&taskId=u644b79d1-a369-4731-a8f8-e2bd5d572d1&title=&width=389)

### 拷贝分析

序列化和反序列化由dds负责，对于FastDDS而言，由FastCDR负责<br />colcon/install/shm_msgs/include/shm_msgs/shm_msgs/msg/detail/image1m__rosidl_typesupport_fastrtps_cpp.hpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664522781953-46dc0312-8ee6-4315-9ada-9c26e1b44baf.png#averageHue=%23272c36&clientId=u06e8da3d-34a7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=279&id=u1112614d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=279&originWidth=461&originalType=binary&ratio=1&rotation=0&showTitle=false&size=26311&status=done&style=none&taskId=ua933988c-729a-4f36-8f9c-47a04492e50&title=&width=461)<br />该接口由rmw_fastrtps调用<br />galactic\src\ros2\rmw_fastrtps\rmw_fastrtps_cpp\src\type_support_common.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664522863255-92b563d2-09c0-4c61-8ce0-39862ba94d3c.png#averageHue=%23242931&clientId=u06e8da3d-34a7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=366&id=uc8c421a6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=366&originWidth=684&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43990&status=done&style=none&taskId=ub3527c00-3445-43f8-92ad-2bd12228aa8&title=&width=684)<br />一般认为，这儿也会有一次拷贝，但是不同的序列化实现由不同的性能

### 讨论

#### protobuf的序列化性能是否优于cdr？

[https://github.com/eclipse-ecal/ecal/discussions/660#discussioncomment-2764014](https://github.com/eclipse-ecal/ecal/discussions/660#discussioncomment-2764014)<br />简单的结论：不是<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664523518898-b236c16d-cb7d-4dae-a538-834b154f1543.png#averageHue=%23f8f9db&clientId=u06e8da3d-34a7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=408&id=u5a160830&margin=%5Bobject%20Object%5D&name=image.png&originHeight=408&originWidth=851&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44424&status=done&style=none&taskId=uddd4eb6e-c180-49bf-aad5-4b34fc1fb76&title=&width=851)

#### 序列化能否避免？

目前的ros2传输中不可避免（待论证）

#### 序列化的拷贝能否避免？

原地序列化

#### loaned_api是否做到了原地序列化？

如果是原地序列化，则性能可大大增加，因为是loaned_api，需要论证！

#### yclonedds_iceoryx是否做到了避免序列化？

#### rmw_iceoryx是否做到了避免序列化？

对于POD类型，可以不序列化，对于非POD，由于size不固定，无法确定memfile的大小，所以必须进行序列化以获取确定的size<br />rmw_iceoryx使用cdr的序列化方式

## rmw_pub&take

### 介绍

rmw_pub由publisher负责<br />rmw_take由executor负责<br />callback由executor负责<br />在rmw_pub和rmw_take之间，publisher通知executor取数据的过程，由dds负责

### 拷贝分析

rmw_take的过程，由dds承担，发生dds通信，产生dds层的拷贝，dds层的拷贝参考《FastDDS架构和拷贝次数分析》

## deserialization

executor通过dds获取完数据之后，根据传递的数据定义自定进行反序列化，获得数据<br />其性能与serilization一致，略。

## convert_to_custom

### 介绍

一般在callback中convert_to_custom，转换为自定义格式是为了接入算法库如opencv

### 拷贝分析

cv_bridge提供了由sensor_image转换为cv_image的函数<br />galactic\src\ros-perception\vision_opencv\cv_bridge\src\cv_bridge.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664525390299-aab1c9b0-e4ff-496c-856f-a20783de1238.png#averageHue=%23242931&clientId=udbfa51aa-6db5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=284&id=ue791de73&margin=%5Bobject%20Object%5D&name=image.png&originHeight=284&originWidth=713&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38121&status=done&style=none&taskId=u91ebcf96-a2a9-4843-8add-659ceb3c695&title=&width=713)<br />这一次拷贝一般是可以避免的，而以牺牲访问性为代价<br />如果后续要做处理，则必须进行拷贝

### 讨论

#### 为什么一定要返回ConstSharedPtr禁止写？

#### 可不可以返回可以写的SharedPtr?

能否向cv_bridge贡献代码

# rclcpp中的优化

## zero-copy(since galactic)

为了使用dds的功能，在rclcpp中也增加了loaned_api以支持DDS的zero-copy<br />FastDDS和iceoryx的zero-copy都是基于loaned-api机制

## type_adaption-intra-process(since humble)

type_adaption功能设计原理在 [https://ros.org/reps/rep-2007.html](https://ros.org/reps/rep-2007.html)<br />type_adaption在inter-process上没有性能优化，在Intra-process上可以避免convert_to_ros和convert_to_custom两次数据拷贝<br />通过设计intra-process manager，如果存在type-adaption且intra-process过程，则避免进行type-conversion<br />[https://github.com/ros2/rclcpp/pull/1849](https://github.com/ros2/rclcpp/pull/1849)<br />For context, the point of this PR is to enhance [TypeAdaptation](https://ros.org/reps/rep-2007.html) so that it avoids unnecessary conversions. In particular, if a Publisher and Subscription are both in the same context, using intra-process communications, and use the same custom TypeAdapted type, **then we can just place the published data directly from the publisher into the Subscription queue for delivery without doing any conversions**. If any of several other situations happen (Publisher is ROS message, Subscription is custom, Publisher is custom, Subscription is ROS message, etc), then we do the conversion as necessary.<br />**This has two primary benefits, both related to performance:**<br />**两个好处，都是关于性能增强的**

1. We can avoid doing the (possibly) expensive and unnecessary to a ROS type on the publisher, and then from the ROS type on the subscription.
   1. 避免类型转换，也就是避免了多余的拷贝
2. We can use a custom type to hold onto custom "handles", allowing the Publisher to pass those handles directly to the Subscription. Those handles might be handles to hardware accelerators, which means that the system can use the hardware accelerator to do the conversion without synchronizing with the CPU to publish to the next node.
   1. 可以保存custom "handlers"，节点间可共享任何类型的数据（之前必须是msg定义的类型），这些handlers可以是指针，进程内共享的指针，可以是CUDA设备内存地址

## type-adaption在cuda程序中的应用

[https://github.com/NVIDIA-ISAAC-ROS/ros2_examples/tree/humble/rclcpp/type_adaptation/accelerated_pipeline](https://github.com/NVIDIA-ISAAC-ROS/ros2_examples/tree/humble/rclcpp/type_adaptation/accelerated_pipeline)<br />![](https://cdn.nlark.com/yuque/0/2022/gif/23125517/1664448390902-1aed7ff3-4315-4102-9ac7-a49f30051e96.gif#averageHue=%23f5f3f5&crop=0&crop=0&crop=1&crop=1&from=url&id=FMy88&margin=%5Bobject%20Object%5D&originHeight=450&originWidth=800&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />该技术基于Cuda，与是否是集成显卡关系不大，所以jetson和非jetson设备均可使用！<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664528927869-45c08f59-39fc-4e6f-8408-a08f490987e5.png#averageHue=%23fbfbfb&clientId=u40cf46f6-8428-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub037dded&margin=%5Bobject%20Object%5D&name=image.png&originHeight=489&originWidth=961&originalType=url&ratio=1&rotation=0&showTitle=false&size=37091&status=done&style=none&taskId=u82c354c2-f467-4953-b8c8-621abb6eb7d&title=)

## 讨论

### cuda内存句柄在进程间共享是否可行

不可行。<br />CUDA内存拷贝的工作在IPC下是不可避免的。<br />之前介绍了一种技术，cuda-ipc，app1可以获取cuda_mem_handle，传递给app2，app2根据这个handle访问cuda的同一块内存<br />[https://forums.developer.nvidia.com/t/gpu-inter-process-communications-ipc-question/35936/5](https://forums.developer.nvidia.com/t/gpu-inter-process-communications-ipc-question/35936/5)<br />这种技术初看很美好，但是有巨大的缺陷，没有将app1和app2完全解耦，使得该技术集成到ros2中很困难<br />其app1，及handle的创建者必须长期存在，当app1关闭，则context失效，app2也无法访问了<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664450425846-d361072e-a8f5-465b-be6c-5c5f1749e218.png#averageHue=%23e6f8c3&clientId=u1556d302-1221-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=159&id=uf78b1a53&margin=%5Bobject%20Object%5D&name=image.png&originHeight=159&originWidth=1250&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38774&status=done&style=none&taskId=u6a719f13-ef48-4ec7-adea-367b2740f75&title=&width=1250)<br />如果publisher使用了该handle技术，则当publisher意外关闭时，cuda_context失效，subscriber恰好在回调阶段使用了handle，则subscriber也会崩溃

# ROS2拷贝分析汇总

| Transport | data<br />construction | convert_to_ros | rmw<br />serial | rmw_pub&take<br />(dds layer) | rmw<br />deserial | convert_to_custom | 备注 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| cyclonedds-default | 1 | 1 | 1 | ip(1...) | 1 | 1 | 默认udp |
| fastdds-default | 1 | 1 | 1 | dds_shm(4) | 1 | 1 | 默认shm |
| zero-copy | 1 | 1 | 1 | 0 | 1 | 1 | 要求msg是POD |
| intra-process | 1 | 1 | - | - | - | 1 | 要求进程内 |
| intra-process with<br />type-adaption | 1 | 0 | - | - | - | 0 | 要求进程内 |
| 注意：<br />data-processing的拷贝次数由开发者定义，此处没有考虑<br />convert_to_ros和convert_to_custom的拷贝次数由开发者定义，例如由cv-bridge的使用决定，表格中为了简化，使用1次来代替<br />type-adaption可以简化msg-conversion的开发，但只能在intra-process场景下避免msg-conversion，其他情况不能避免<br />type-adaption-opt可以避免序列化，Humble还没实现 |  |  |  |  |  |  |  |

# 结论

- 通过zero-copy机制可以大大减少拷贝次数，增强其性能
  - zero-copy机制下的序列化机制需要进一步论证
- 通过使用type-adaption-intra-process，可以大大增强cuda程序的性能
  - 其依赖intra-process机制，要求所有订阅节点都在一个进程内
  - 在存在IPC订阅者的情况下，intra-process的性能会减弱
  - [https://github.com/ros2/rclcpp/issues/1860](https://github.com/ros2/rclcpp/issues/1860)

# 示例代码

## image_talker

```cpp
class Talker : public rclcpp::Node {
private:
  using Topic = sensor_msgs::msg::Image;

public:
  explicit Talker(const rclcpp::NodeOptions &options)
      : Node("image_talker", options) {

    // m_input_cvimage->image = cv::imread("./res/img/205_182.png");
    // m_input_cvimage->image = cv::imread("./res/img/1024_768.jpeg");
    m_input_cvimage->image = cv::imread("./res/img/1920_1080.jpg");
    m_input_cvimage->header.frame_id = "camera_link";
    m_input_cvimage->encoding = "bgr8";
    // cv::imshow("input image", m_input_cvimage->image);
    // cv::waitKey(0);

    auto publishMessage = [this]() -> void {
      m_input_cvimage->header.stamp = now();

      auto msg = std::make_shared<Topic>();
      // Note that msg.data is a std::array generated by the IDL compiler
      m_input_cvimage->toImageMsg(*msg);

      RCLCPP_INFO(this->get_logger(), "Publishing with ts: %u.%u", m_input_cvimage->header.stamp.sec, m_input_cvimage->header.stamp.nanosec);

      m_publisher->publish(std::move(*msg));
      // We gave up ownership and loanedMsg is not supposed to be accessed
      // anymore

      m_count++;
    };

    // rclcpp::QoS qos(rclcpp::KeepLast(10));
    // rclcpp::QoS qos(rclcpp::SensorDataQoS());
    rclcpp::QoS custom_qos_profile = rclcpp::QoS(rclcpp::QoSInitialization::from_rmw(rmw_qos_profile_default))
      .history(rmw_qos_history_policy_t::RMW_QOS_POLICY_HISTORY_KEEP_LAST)
      .keep_last(5)
      .reliability(rmw_qos_reliability_policy_t::RMW_QOS_POLICY_RELIABILITY_BEST_EFFORT)
      .durability(rmw_qos_durability_policy_t::RMW_QOS_POLICY_DURABILITY_VOLATILE)
      .avoid_ros_namespace_conventions(false);

    m_publisher = this->create_publisher<Topic>("image", custom_qos_profile);

    // Use a timer to schedule periodic message publishing.
    m_timer = this->create_wall_timer(0.1s, publishMessage);
  }

private:
  uint64_t m_count = 1;
  rclcpp::Publisher<Topic>::SharedPtr m_publisher;
  rclcpp::TimerBase::SharedPtr m_timer;
  std::shared_ptr<cv_bridge::CvImage> m_input_cvimage{std::make_shared<cv_bridge::CvImage>()};

};
```

## image_listener

```cpp
public:
  explicit Listener(const rclcpp::NodeOptions &options)
      : Node("image_listener", options) {

    // subscription callback to process arriving data
    auto callback = [this](const Topic::SharedPtr msg) -> void {

      RCLCPP_INFO(this->get_logger(), "Received...");
      last_cvimage = cv_bridge::toCvShare(msg);
      // last_cvimage = cv_bridge::toCvCopy(msg);

      auto time_offset_ns = (now() - last_cvimage->header.stamp).nanoseconds();
      auto timestamp_offset_ns = (rclcpp::Time(msg->header.stamp) - m_last_image_ts).nanoseconds();
      auto time_offset_ms = time_offset_ns / 1000000.0F;
      auto timestamp_offset_ms = timestamp_offset_ns / 1000000.0F;
      RCLCPP_INFO(get_logger(), "get-image-transport-time: %.3f", time_offset_ms);
      if(m_last_image_ts.nanoseconds() > 0.0)
      {
        RCLCPP_INFO(get_logger(), "get-image-timestamp_offset-time: %.3f", timestamp_offset_ms);
      }
      m_last_image_ts = msg->header.stamp;
      // cv::imshow("im show", last_cvimage->image);
      // cv::waitKey(0);
    };

    // rclcpp::QoS qos(rclcpp::KeepLast(10));
    rclcpp::QoS custom_qos_profile = rclcpp::QoS(rclcpp::QoSInitialization::from_rmw(rmw_qos_profile_default))
      .history(rmw_qos_history_policy_t::RMW_QOS_POLICY_HISTORY_KEEP_LAST)
      .keep_last(5)
      .reliability(rmw_qos_reliability_policy_t::RMW_QOS_POLICY_RELIABILITY_BEST_EFFORT)
      .durability(rmw_qos_durability_policy_t::RMW_QOS_POLICY_DURABILITY_VOLATILE)
      .avoid_ros_namespace_conventions(false);

    m_subscription = create_subscription<Topic>("image", custom_qos_profile, callback);
  }

private:
  rclcpp::Subscription<Topic>::SharedPtr m_subscription;

  cv_bridge::CvImageConstPtr last_cvimage;
  rclcpp::Time m_last_image_ts{0, 0, RCL_ROS_TIME};
};
```
