# 概述

描述ROS Type与其他Type之间互相转换的类定义方法，使得数据类型转换更加方便，作为rclcpp的扩展机制

# 动机

The primary reason for this change is to improve the developer's experience working with ROS 2. Currently, developers often write code to convert a ROS interface into another custom data structure for use in their programs.<br />最主要的还是要提升ROS2用户的体验，因为总是需要将数据转换成标准的数据类型，经常会遇到数据类型转换<br />This can be trivial, in the case accessing the data field in std_msgs::msg::String; or more involved such as when converting OpenCV's cv::Map to ROS's sensor_msgs/msg/Image type [1]. Such conversions are additional work for the programmer and are potential sources of errors.<br />还有一个是性能考虑，该接口可以允许用户定义直接在用户类型上串行和解串的方法，特别是在Intra-process的情况下还需要进行数据类型转换，就很神奇<br />An additional reason for this change is performance. This interface will allow us to define methods for serializing directly to the user requested type, and/or using that type in intra-process communication **without ever converting it**. Whereas without this feature, even to send a cv::Mat (or a data structure containing a cv::Mat) from one publisher to a subscription **in the same process** would require converting it to a sensor_msgs::msg::Image first and then back again to cv::Mat.<br />避免在进程内通信的情况下还要做类型转换，直接传递Image就可以了。进程间通信

# 定义一个type adapter

将custom type映射到ROS Type，用户需要创建一个模板实现。

- change is_specialized to std::true_type,
- specify the custom type with using custom_type = ...,
- specify the ROS type with using ros_message_type = ...,
- provide static convert functions with the signatures:static void convert_to_ros_message(const custom_type &, ros_message_type &),static void convert_to_custom(const ros_message_type &, custom_type &)
- static void convert_to_ros_message(const custom_type &, ros_message_type &),
- static void convert_to_custom(const ros_message_type &, custom_type &)

一个将std::string转到std_msgs::msg::String的映射

```cpp
template<>
struct rclcpp::TypeAdapter<
   std::string,
   std_msgs::msg::String
>
{
  using is_specialized = std::true_type;
  using custom_type = std::string;
  using ros_message_type = std_msgs::msg::String;

  static
  void
  convert_to_ros_message(
    const custom_type & source,
    ros_message_type & destination)
  {
    destination.data = source;
  }

  static
  void
  convert_to_custom(
    const ros_message_type & source,
    custom_type & destination)
  {
    destination = source.data;
  }
};
```

# 使用topic

The adapter can then be used when creating a publisher or subscription, e.g.:

```cpp
using MyAdaptedType = TypeAdapter<std::string, std_msgs::msg::String>;
auto pub = node->create_publisher<MyAdaptedType>("topic", 10);
auto sub = node->create_subscription<MyAdaptedType>(
  "topic",
  10,
  [](const std::string & msg) {...});
```

You can also be more declarative by using the adapt_type::as metafunctions, which are a bit less ambiguous to read:

```cpp
using AdaptedType = rclcpp::adapt_type<std::string>::as<std_msgs::msg::String>;
auto pub = node->create_publisher<AdaptedType>(...);
```

If you wish, you may associate a custom type with a single ROS message type, allowing you to be a bit more brief when creating entities, e.g.:

```cpp
// First you must declare the association, this is similar to how you
// would avoid using the namespace in C++ by doing `using std::vector;`.
RCLCPP_USING_CUSTOM_TYPE_AS_ROS_MESSAGE_TYPE(std::string, std_msgs::msg::String);

// Then you can create things with just the custom type, and the ROS
// message type is implied based on the previous statement.
auto pub = node->create_publisher<std::string>(...);
```

Note that it is also possible to use a ROS type with a publisher or subscriber that has been specialized to use a custom message, e.g.:

```cpp
using AdaptedType = rclcpp::adapt_type<std::string>::as<std_msgs::msg::String>;
auto pub = node->create_publisher<AdaptedType>(...);

// Publish a std::string
std::string custom_msg = "My std::string"
pub->publish(custom_msg);

// Publish a std_msgs::msg::String;
std_msgs::msg::String ros_msg;
ros_msg.data = "My std_msgs::msg::String";
pub->publish(ros_msg);
```

# 设计考虑

## 选择术语

There are various terms that may be suitable for type adapting feature described. In selecting a term,

| High priority: |  |
| --- | --- |

- Clearly communicate the described feature
- Clearly communicate the order of custom type and ROS type arguments
| Low priority: |  |
| --- | --- |

- The custom type should be the first argument so that *the custom type is the first argument in both the explicit and implicit syntax* the custom type is read first, for convenience
- The syntax reads well

### Candidate terms

Several possible terms were considered. Here is a brief summary of the discussion around different terms.

#### Masquerade面具

There is some precedent for using masquerade in similar settings, IP Masquerading in the Linux kernel [2] for example. "Masquerade" is also a verb, which may make it easier to discuss among developers. However, it was thought that "Masquerade" would be a confusing word for non-English and non-French speakers. One disadvantage of "Masquerade" is that there is ambiguity in its usage. For example,

```cpp
Masquerade<std_msgs::msg::String>::as<std::string>
```

```cpp
Masquerade<std::string>::as<std_msgs::msg::String>
```

both seem to make sense. This ambiguity may result in frustration on the part of the ROS 2 developer:

- frequently having to refer back to documentation
- possibly opaque error messages

#### Facade门面

"Facade" seems to be a more common English word than "masquerade". It also is commonly used as a design pattern in object oriented programming. However, the "Facade pattern" is typically used to simplify a complex interface [3], which is not the major feature being proposed here.<br />It was thought to use "Facade" in the following form:

```cpp
Facade<std::string>::instead_of<std_msgs::msg::String>
```

#### Adapter适配器

"Adapter" is certainly a common English word, and the "Adapter pattern" is a common design pattern for adjusting an interface [4], which matches well with the feature being suggested here. Also, using "Adapter" is consistent with the documentation of a similar feature in ROS 1 (i.e., "Adapting C++ Types" [5]).<br />"Adapter" also has the advantage of being a noun and of being related to the verb "Adapt". This flexibility may make it easier for developers to discuss its use.<br />"Adapter" could be used in the following syntax:

```cpp
TypeAdapter<std::string>::as<std_msgs::msg::String>
```

### Additional terms considered

Here is a brief listing of additional terms that were considered and why they were not selected:

| Convert:转换器 | Passed in favor of "Adapter", which expresses a similar idea and has a common design pattern. |
| --- | --- |
| Decorate:装饰器 | Passed in favor of "Fascade", which seems to be more common. |
| Mask:掩膜 | Overloaded as a computer science term<br />[6]<br />. |
| Map:映射 | Expresses the idea well, but has a lot of meanings in math and programming. |
| Use: 别名 | Possibly confusing with C++'s<br />using<br />keyword; also not terribly descriptive. |
| Wrap: 包装器 | Passed in favor of "Adapt", which seems to be more common. |

## rclcpp实现

### 在rclcpp实现还是单独插件

Adding this feature in rclcppPlacing this feature in ROS 2's client support library, rcl, would allow this feature to be used in other client libraries, such as rclcpp and rclpy. However, we believe that the concrete benefits for C++ currently outweigh the potential benefits for existing or theoretical client libraries in other languages. <br />单独插件的好处是，rclcpp，rclpy等其他client都可以用上，但是我们认为rclcpp的重要性比别的都要高，所以优先集成到rclcpp，集成到rclcpp减少了复杂性<br />For example, placing this feature in rclcpp allows us to avoid type erasure (which would be necessary to place this functionality into rcl) and to use ownership mechanics (unique and shared pointer) to ensure it is safely implemented. Another added advantage of placing this feature in rclcpp is that it reduce the number of function calls and calls that potentially are to functions in separate shared libraries.Perhaps we can support a form of this feature in other languages in rcl or rmw in the future. One challenge in doing this is that it may require custom type support, which may be middleware specific. This possibility will be further explored in the future.

### On the Location for Specifying the Type Adapter

It was suggested that we only template the Publisher::publish method, but in addition to being more convenient, specifying a type adapter for the publisher at instantiation rather than in Publisher::publish allows the intra process system to be setup to expect a custom type. Similarly, it is preferable to specify the adapted type at instantiation for subscriptions, service clients, service servers, action clients, and action servers.

### 对比ros1的type-adaption

Although intended to be similar in functionality, the proposed feature and ROS 1's type adaptation support [[5]](https://ros.org/reps/rep-2007.html#id12) have a few important differences:

- This feature will support both convert and (de)serialize functions, and require at least one or the other, but also allow both. The reason for this is that convert is superior for intra-process communication and the (de)serialization functions are better for inter-process communication.
  - convert对于intra-process通信的意义更加重大
  - serialization对于IPC通信的意义更加重大
- This feature will also require the user to write less code when creating an adapter, as compared to the ROS 1 implementation.
- An advantage of following the ROS 1 approach is that **an extra copy can be avoided**; although it is likely much more challenging to implement this feature the ROS 1 way because of the middleware.

# FeatureProgress

The type adaptation API has been implemented for publishers and subscribers ([ros2/rclcpp#1557](https://github.com/ros2/rclcpp/pull/1557)). Examples ([ros2/examples#300](https://github.com/ros2/examples/pull/300)) and demos ([ros2/demos#482](https://github.com/ros2/demos/pull/482)) for using type adaptation have also been created.<br />There are several other features specified in this REP that have not been implemented. You can check the issues below to see the state of the reference implementation.

- integrate into intra-process manager ([ros2/rclcpp#1664](https://github.com/ros2/rclcpp/issues/1664)) - 完成
- support serialize/deserialize functions in addition to the convert functions ([ros2/rclcpp#1665](https://github.com/ros2/rclcpp/issues/1665))- 未完成
- support services ([ros2/rclcpp#1666](https://github.com/ros2/rclcpp/issues/1666))- 未完成
- support actions ([ros2/rclcpp#1667](https://github.com/ros2/rclcpp/issues/1667))- 未完成

很多工作还没实现！

# demo

## 最小demo

[https://github.com/ros2/examples/pull/300](https://github.com/ros2/examples/pull/300)

## image-container

[https://github.com/ros2/demos/pull/482](https://github.com/ros2/demos/pull/482)

## image-container-cvbridge

[https://github.com/ros2/demos/issues/583](https://github.com/ros2/demos/issues/583)<br />[https://github.com/ros-perception/vision_opencv/pull/441](https://github.com/ros-perception/vision_opencv/pull/441)

### pointcloud-container-pcl_conversion

# 讨论

## TypeAdagtion当前没有省略type-conversion

而是简化了type-conversion，~~避免了过多的copy~~，压根没有避免copy<br />最后还是转化成了ros_message_type

## TypeConversion过程能省么？

没有cv-bridge，就不能在图片里面加时间戳<br />但是opencv处理的cv_image就是没有时间戳的<br />这个过程是必要的，不能省

## TypeConversion的过程产生了拷贝没有？

我认为没有产生（重大数据的）拷贝，cv_<br />cv-bridge中的拷贝能否减少？

## 非标准类型的序列化性能如何？

之所以要标准数据类型，就是为了序列化性能<br />为了zero-copy，转换成POD数据类型，如果去除了type-adaption，则zero-copy不能实现

## 没有序列化能否IPC

typeadaption的目标除了避免类型转换，还要避免串行化<br />如果避免了序列化，如何进行IPC？<br />基于SHM的IPC不需要序列化，那倒是个可行的地方

## 当前typeadaption功能没避免序列化

所以对性能优化没提升作用，在performance_test中进行测试

# 参考
<https://github.com/ros-infrastructure/rep/blob/master/rep-2007.rst><br />[https://github.com/ros2/rclcpp/pull/1849](https://github.com/ros2/rclcpp/pull/1849)<br />[https://github.com/ros2/examples/pull/300](https://github.com/ros2/examples/pull/300)<br />[https://github.com/ros2/demos/pull/482](https://github.com/ros2/demos/pull/482)<br />[https://github.com/ros2/demos/issues/583](https://github.com/ros2/demos/issues/583)<br />[https://github.com/ros-perception/vision_opencv/pull/441](https://github.com/ros-perception/vision_opencv/pull/441)
