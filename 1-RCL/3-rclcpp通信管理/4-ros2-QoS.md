# ROS2 qos

Note that not all publishers on the same topic with the same type will be compatible: they must have compatible Quality of Service policies.<br />不懂qos可能连收消息都收不到。

## 概述

由于整个网络环境中有不同等级的数据流量，有不同的质量需求，所以需要有服务质量控制的机制。

- 网络质量佳的时候，不同的数据流量，有不同优先级的需求
- 有的时候网络条件恶劣，比如走wifi，需要优先保证某数据流量的质量

ROS1使用tcp作为数据传输方式，在wifi条件下质量很差，而ros2下的dds主要用的是udp，可以进行服务质量控制。

## 概念

qos在dds的规范中有定义，所以dds是支持qos配置的。在ros中的 Topic, DataReader, DataWriter, Publisher and Subscriber.都支持qos配置。由于qos是基于Request vs Offerer Model, 请求与提供模型，所以Publications and Subscriptions的qos设置必须兼容，才能正常收发数据。

## qos策略

策略定义了一系列概念：

- 持久性durability
  - transient local: 本地持久。只在DataWriter起作用，如果没有dataReader接受，则持久保存该数据
    - 对于不会过期的数据，适用此配置
  - Volatile: 可变的。不保存数据，数据过期即作废。
- 可靠性reliability
  - best effort: 尽力传输，可能丢包
  - reliable: 可靠传输，速率受限
- history历史
  - 在keep last时候生效
    - 队列深度queue depth
  - keep all
- 数据历史保存sample history storage
  - keep last: 只保存尾部的N个数据，通过queen depth设置
  - keep all: 保存所有的数据，受制于dds的资源限制参数
- Deadline（数据过期）
  - 对于消息与消息之间的时间间隔定义了条款。对于消息订阅来说，定义了收取的消息之间最大的时间间隔。对于发布者来说定义了发送消息之间最大的时间间隔。如果rmw在指定时间内还没有收到消息，则认为消息丢失。对于订阅者来讲，时间间隔应该大于或等于发布者定义的间隔。deadline=0, 去使能deadline. 默认值=0
- Liveliness（保活）
  - 对实体报告自己的存活期定义了条款。发布者的Liveliness向接收者报告自己的存活等级，订阅者向发布者报告自己的存活等级
  - LIVELINESS_SYSTEM_DEFAULT-默认自动等级
  - LIVELINESS_AUTOMATIC
  - LIVELINESS_MANUAL_BY_NODE-node级别的心跳包，在该节点的任意主题发布一个数据，则认定该节点保活
  - LIVELINESS_MANUAL_BY_TOPIC-topic级别的心跳包。
  - 订阅者的等级必须低于发布者的等级，不然可能不能接受到心跳包。
- LifeSpan（有效期）
  - 定义了数据有效的期限。默认为0
  - 对于订阅者定义了消息在某个时间限度内为有效数据
  - 对于发布者定义了消息在某个时间后为无效数据
  - 超期的数据会被静默丢弃，不会发送或者收取。

有一个默认配置，由dds提供。<br />后面的三个配置应该是为了状态诊断而设置的。LifecycleNode一起使用。<br />和诊断一起保证节点的高可靠性。

## 与ROS1比较

history和depth是ros1中的queuesize.<br />reliability与ros1中的tcpros类似<br />besteffort与ros1中的udpros类似<br />durability的transient local与ros1中的latching类似<br />其他的在ros1中没有类似的策略

## 资源状态事件

Deadline和Liveliness策略都会产生资源事件，表示数据过期或者节点失活，节点可以定义当deadline超时时的事件。这些回调函数由用户设计。<br />回调函数只有在资源状态发生变化时才调用，不会每次事件发生时刻都调用。<br />Developers may subscribe to the following QoS events that are associated with a publisher:

- Offered deadline missedThe publisher has not published a message within the expected duration that was set out by the deadline QoS policy.
- Liveliness lostThe publisher has failed to indicate its liveliness within the lease duration.
- Offered incompatible QoSThe publisher has encountered a subscription on the same topic that is requesting a QoS profile that the offered QoS profile cannot satisfy, resulting in no connection between the publisher and that subscription.

Developers may subscribe to the following QoS events that are associated with a subscription:

- Requested deadline missedThe subscription has not received a message within the expected duration that was set out by the deadline QoS policy.
- Liveliness changedThe subscription has noticed that one or more publishers on the subscribed topic has failed to indicate their liveliness within the lease duration.
- Requested incompatible QoSThe subscription has encountered a publisher on the same topic that is offering a QoS profile that does not satisfy the requested QoS profile, resulting in no connection between the subscription and that publisher.

## 保活函数

除了发送消息和接收消息可以确保活性，还可以执行Assert Liveliness Functions

## 常用设置

- 默认配置：
- 为了保持与ROS1的无缝衔接，当前的网络行为与ROS1是相似的，为reliable, volatile durability, keep last history.
- 服务：
- reliable, volotile durability. voletile 对服务特别重要，这可以保证服务端重启后仍然可以接收到过期的服务请求
- SensorData
- 对与传感器数据，大部分时候的需求是准时收取，不需要所有数据都收到。所以QoS配置为best effort, small depth history
- parameters
- 参数配置是基于service的，所以配置也和service差不多。使用较大的queue depth所以数据没那么容易丢失。

Click here for the specific policies in use for the above profiles. The settings in these profiles are subject to further tweaks, based on the feedback from the community.

## 配置传递

qos设置通过rclcpp完成，也可以直接通过xml文件来配置dds，建议通过rcl。<br />当通过xml配置dds时，必须保证rmw和dds的qos配置相同，不然可能产生数据乱序的情况。<br />如果在dds中配置reliable，则新消息会被放入队列，等待所有老消息到来后再去除，而rmw的besteffort则会收到新消息后把老消息直接丢弃。<br />rclcpp中定义了一个qos结构体用来定义qos策略<br />rmw需要支持特定的qos策略，才能够使策略生效。

## 配置兼容性

发布者和订阅者都可以独立制定qos策略，只有两者兼容的情况下才能正常通信。<br />QoS profile compatibility is determined based on a “Request vs Offered” model. Subscriptions request a QoS profile that is the “minimum quality” that it is willing to accept, and publishers offer a QoS profile that is the “maximum quality” that it is able to provide.<br />订阅者的qos是针对所需要的数据的最低质量，发布者的qos是发布数据的最高质量。<br />_Compatibility of reliability QoS policies:_

| Publisher | Subscription | Compatible |
| --- | --- | --- |
| Best effort | Best effort | Yes |
| Best effort | Reliable | No |
| Reliable | Best effort | Yes |
| Reliable | Reliable | Yes |

_Compatibility of durability QoS policies:_

| Publisher | Subscription | Compatible |
| --- | --- | --- |
| Volatile | Volatile | Yes |
| Volatile | Transient local | No |
| Transient local | Volatile | Yes |
| Transient local | Transient local | Yes |

_Compatibility of deadline QoS policies:_<br />Assume _x_ and _y_ are arbitrary valid duration values.

| Publisher | Subscription | Compatible |
| --- | --- | --- |
| Default | Default | Yes |
| Default | _x_ | No |
| _x_ | Default | Yes |
| _x_ | _x_ | Yes |
| _x_ | _y_ (where _y_ > _x_) | Yes |
| _x_ | _y_ (where _y_ < _x_) | No |

_Compatibility of liveliness QoS policies:_

| Publisher | Subscription | Compatible |
| --- | --- | --- |
| Automatic | Automatic | Yes |
| Automatic | Manual by topic | No |
| Manual by topic | Automatic | Yes |
| Manual by topic | Manual by topic | Yes |

_Compatibility of lease duration QoS policies:_<br />Assume _x_ and _y_ are arbitrary valid duration values.

| Publisher | Subscription | Compatible |
| --- | --- | --- |
| Default | Default | Yes |
| Default | _x_ | No |
| _x_ | Default | Yes |
| _x_ | _x_ | Yes |
| _x_ | _y_ (where _y_ > _x_) | Yes |
| _x_ | _y_ (where _y_ < _x_) | No |

兼容性必须保证所有维度下都兼容才行。<br />即订阅者要求的数据质量必须得到满足，dds通信连接才会建立。

## 暴露更多QoS

**Plan to port more DDS QoS to ROS2**<br />Is there an official plan to port more DDS QoS to ROS2? If not, shall we start listing the useful DDS QoS, the candidates to be ported to ROS2? I will start 😃 TransportPriority, LatencyBudget, and Liveliness: useful for real time applications. OwnershipStrength: useful for software redundancy.<br /><https://discourse.ros.org/t/plan-to-port-more-dds-qos-to-ros2/19358><br />One of the advantages of Fast DDS is you can configure the QoS in an XML file, so for example, setting the transport priority is trivial, even if it is not exposed.<br />But is also true that it would be very easy to expose these QoS.<br />There is no plan to do it, neither plan to don’t do it. It is a matter of creating a PR of the QoS you want to expose and follow the process to get that PR accepted, as in any other ROS feature.

# rclcpp的qos定义

galactic\src\ros2\rclcpp\rclcpp\include\rclcpp\qos.hpp

## 使用方法

```cpp
rclcpp::QoS custom_qos_profile = rclcpp::QoS(rclcpp::QoSInitialization::from_rmw(rmw_qos_profile_default))
      .history(rmw_qos_history_policy_t::RMW_QOS_POLICY_HISTORY_KEEP_LAST)
      .keep_last(1)
      .reliability(rmw_qos_reliability_policy_t::RMW_QOS_POLICY_RELIABILITY_BEST_EFFORT)
      .durability(rmw_qos_durability_policy_t::RMW_QOS_POLICY_DURABILITY_TRANSIENT_LOCAL)
      .avoid_ros_namespace_conventions(false);
```

## 默认systemdefault

直接找rmw要默认值

```cpp
goal_sub_ = create_subscription<geometry_msgs::msg::PoseStamped>(
    "goal_pose",
    rclcpp::SystemDefaultsQoS(),
    std::bind(&BtNavigator::onGoalPoseReceived, this, std::placeholders::_1));

class RCLCPP_PUBLIC SystemDefaultsQoS : public QoS
{
public:
  explicit
  SystemDefaultsQoS(
    const QoSInitialization & qos_initialization = (
      QoSInitialization::from_rmw(rmw_qos_profile_system_default)
  ));
};

```

## 预定义QoS

### 传感器数据

```cpp
particlecloud_pub_ = create_publisher<geometry_msgs::msg::PoseArray>(
    "particlecloud",
    rclcpp::SensorDataQoS());

/**
 * Sensor Data QoS class
 *    - History: Keep last,
 *    - Depth: 5,
 *    - Reliability: Best effort,
 *    - Durability: Volatile,
 *    - Deadline: Default,
 *    - Lifespan: Default,
 *    - Liveliness: System default,
 *    - Liveliness lease duration: default,
 *    - avoid ros namespace conventions: false
 */
class RCLCPP_PUBLIC SensorDataQoS : public QoS
{
public:
  explicit
  SensorDataQoS(
    const QoSInitialization & qos_initialization = (
      QoSInitialization::from_rmw(rmw_qos_profile_sensor_data)
  ));
};
```

### map数据

```cpp
rclcpp::QoS map_qos(10);  // initialize to default
  if (map_subscribe_transient_local_) {
    map_qos.transient_local();
    map_qos.reliable();
    map_qos.keep_last(1);
  }
```

### 图像数据

图像数据往往和标定信息一起发布<br />根据nav2，autoware.auto代码，没有发现四种新的策略的应用情况。

# rmw的qos定义

galactic\src\ros2\rmw\rmw\include\rmw\qos_profiles.h

```cpp
static const rmw_qos_profile_t rmw_qos_profile_sensor_data =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  5,
  RMW_QOS_POLICY_RELIABILITY_BEST_EFFORT,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_parameters =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  1000,
  RMW_QOS_POLICY_RELIABILITY_RELIABLE,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_default =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  10,
  RMW_QOS_POLICY_RELIABILITY_RELIABLE,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_services_default =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  10,
  RMW_QOS_POLICY_RELIABILITY_RELIABLE,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_parameter_events =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  1000,
  RMW_QOS_POLICY_RELIABILITY_RELIABLE,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};

static const rmw_qos_profile_t rmw_qos_profile_system_default =
{
  RMW_QOS_POLICY_HISTORY_SYSTEM_DEFAULT,      RMW_QOS_POLICY_HISTORY_KEEP_LAST
  RMW_QOS_POLICY_DEPTH_SYSTEM_DEFAULT,        1
  RMW_QOS_POLICY_RELIABILITY_SYSTEM_DEFAULT,  RMW_QOS_POLICY_RELIABILITY_RELIABLE
  RMW_QOS_POLICY_DURABILITY_SYSTEM_DEFAULT,   RMW_QOS_POLICY_DURABILITY_VOLATILE
  RMW_QOS_DEADLINE_DEFAULT,                   ??
  RMW_QOS_LIFESPAN_DEFAULT,                   #define RMW_DURATION_UNSPECIFIED {0LL, 0LL}
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,   RMW_QOS_POLICY_LIVELINESS_AUTOMATIC
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,  #define RMW_DURATION_UNSPECIFIED {0LL, 0LL}
  false
};
// galactic\src\ros2\rmw_cyclonedds\rmw_cyclonedds_cpp\src\rmw_node.cpp
```

# 确认dds支持的qos策略

从源代码的角度查看ecal的支持<br />可以看到ecal支持所有类型的qos

```cpp
inline PublisherQOS CreatePublisherQOS(const rmw_qos_profile_t *rmw_qos)
{
    PublisherQOS qos;
    if (IsPolicySpecified(rmw_qos->history))
    {
        qos.ecal_qos.history_kind = ToECalPolicy(rmw_qos->history);
    }
    if (IsPolicySpecified(rmw_qos->reliability))
    {
        qos.ecal_qos.reliability = ToECalPolicy(rmw_qos->reliability);
    }
    qos.ecal_qos.history_kind_depth = ToECalDepth(rmw_qos->depth);


    qos.rmw_qos.avoid_ros_namespace_conventions = rmw_qos->avoid_ros_namespace_conventions;
    qos.rmw_qos.depth = rmw_qos->depth;
    qos.rmw_qos.history = ToRosPolicy(qos.ecal_qos.history_kind);
    qos.rmw_qos.reliability = ToRosPolicy(qos.ecal_qos.reliability);
    qos.rmw_qos.deadline = {0, 0};
    qos.rmw_qos.durability = RMW_QOS_POLICY_DURABILITY_VOLATILE;
    qos.rmw_qos.lifespan = {0, 0};
    qos.rmw_qos.liveliness = RMW_QOS_POLICY_LIVELINESS_UNKNOWN;
    qos.rmw_qos.liveliness_lease_duration = {0, 0};


    if (!rmw_qos->avoid_ros_namespace_conventions)
    {
        qos.topic_name_prefix = pub_name_prefix;
    }


    return qos;
}
```

## 覆盖硬编码的qos配置

从galactic开始，支持从parameter文件中加载qos配置

# 附录：rmw配置

问题: 在Wifi环境等恶劣的网路环境下, 数据会丢包, 可能导致接收端内核缓冲区满<br />当一个UDP包丢失一个帧时, 接收端缓冲区会阻塞,, 默认的缓冲区大小为256K, <br />表现: 网络阻塞, 数据丢失<br />解决方案: 将QoS配置从改成reliable 改成 best-effort<br />解释: 尽力而为减少了数据发送, 如果接收方阻塞<br />其他解决: <br />降低`ipfrag_time` parameter. 减少IP片在内存中的存活时间<br />`sudo sysctl net.ipv4.ipfrag_time=3`<br />增加重组IP报文的内存空间<br />`sudo sysctl net.ipv4.ipfrag_high_thresh=134217728 # (128 MB)`

# 参考

**DDS tuning information — ROS 2 Documentation: Foxy documentation**<br /><https://docs.ros.org/en/foxy/Guides/DDS-tuning.html><br />**About Quality of Service settings — ROS 2 Documentation: Rolling documentation**<br /><https://docs.ros.org/en/rolling/Concepts/About-Quality-of-Service-Settings.html><br />**ROS QoS - Deadline, Liveliness, and Lifespan**<br />Distilled design documents related to the ROS 2 effort<br /><https://design.ros2.org/articles/qos_deadline_liveliness_lifespan.html><br /><https://design.ros2.org/articles/qos.html><br />**External configurability of QoS policies.**<br />Distilled design documents related to the ROS 2 effort<br /><http://design.ros2.org/articles/qos_configurability.html><br />**About QoS of Images**<br />Dear ROS Community, These days I have been working with various packages related to images (ros_astra_camera, image_common, image_pipeline) and webots. I have found that the QoS of the images is a problem. As far as I know, an image should be published by default as rclcpp::SensorDataQoS (let’s talk in C ++, although I include any RCL), and the associated info as rclcpp:QoS(N).transient_local(). In general, any sensory data should be like this. I would like to know if this is correct, and if w...<br /><https://discourse.ros.org/t/about-qos-of-images/18744><br />**ros2outdoor/zed-ros2-wrapper**<br />ROS 2 wrapper beta for the ZED SDK. Contribute to ros2outdoor/zed-ros2-wrapper development by creating an account on GitHub.<br /><https://github.com/ros2outdoor/zed-ros2-wrapper>
