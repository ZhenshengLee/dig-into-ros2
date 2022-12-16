# 背景

机器人算法和数据同步对时间是有严格依赖的。所以我们希望在ROS系统中的节点有一个同步过的时钟，用于报告各种事件

此处的real time指的是真实时间，而不是实时系统

# 抽象时间的动机

当回放数据的时候，通常需要支持时间的加速，减速，或者跳跃。这种控制允许暂停时间和系统，允许深入调试程序。通常可以通过记录传感器数据的方式做到，但是如果传感器数据与系统的时间不同步，则算法系统就会失效。

另一个场景是机器人仿真中播放真实数据。根据仿真的情况，仿真器可能需要加速或满速播放数据。加速播放对于高层仿真用例执行是有用的。减速播放对于深度调试复杂系统是有用的。有时算法过于复杂，必须减速播放数据，否则无法运行。有时仿真需要暂停，此时时间也需要暂停。

# 挑战

第一个挑战：控制时间。很多算法都有一个前提假设，时间是自然连续增长的。使用抽象时间的目的是加速，减慢，暂停时间，用于算法调试。

第二个挑战：传输时延。时间的变化在分布式网络中传输也会有时延的困扰。每一个时间变更都要迅速传递到整个分布式网络中。

第三个挑战：回到过去。回放过去的数据，会输出负数的时间，算法如何在负数时间下仍然正常工作，处理不连续的时间。

# 时间源类型

至少有三种时间类型，SystemTime, SteadyTime and ROSTime。这个理念与std::chrono system_clock and steady_clock是一致的。

默认选择的时间源类型是ROS_TIME,

Objects of class `system_clock` represent wall clock time from the system-wide realtime clock.

Objects of class `steady_clock` represent clocks for which values of `time_point` never decrease as physical time advances and for which values of `time_point` advance at a steady rate relative to real time. That is, the clock may not be adjusted.

Objects of class `high_resolution_clock` represent clocks with the shortest tick period. `high_resolution_clock` may be a synonym for `system_clock` or `steady_clock`.

从上述标准内容可看到，system_time其实就是wall_time

## System Time

For convenience in these cases we will also provide the same API as above, but use the name `SystemTime`.`SystemTime` will be directly tied to the system clock.

当输入一个时间的时候，默认是SYSTEM_TIME

```c
Time(int32_t seconds, uint32_t nanoseconds, rcl_clock_type_t clock_type = RCL_SYSTEM_TIME);
```

## ROS Time

The `ROSTime` will report the same as `SystemTime` when a ROS Time Source is not active. When the ROS time source is active `ROSTime` will return the latest value reported by the Time Source. `ROSTime` is considered active when the parameter `use_sim_time` is set on the node.

所以消息的时间戳默认是以ROS_TIME为时间源的

```cpp
  Time(
    const builtin_interfaces::msg::Time & time_msg,
    rcl_clock_type_t clock_type = RCL_ROS_TIME);

```

## Steady Time

Example use cases for this include hardware drivers which are interacting with peripherals with hardware timeouts.

与硬件交互的时候，需要时间的严格单调递增。

In nodes which require the use of `SteadyTime` or `SystemTime` for interacting with hardware or other peripherals it is expected that they do a best effort to isolate any `SystemTime` or `SteadyTime` information inside their implementation and translate external interfaces to use the ROS time abstraction when communicating over the ROS network.

```cpp
 * RCL_SYSTEM_TIME reports the same value as the system clock.
 *
 * RCL_STEADY_TIME reports a value from a monotonically increasing clock.
 */
typedef enum rcl_clock_type_t
{
  /// Clock uninitialized
  RCL_CLOCK_UNINITIALIZED = 0,
  /// Use ROS time
  RCL_ROS_TIME,
  /// Use system time
  RCL_SYSTEM_TIME,
  /// Use a steady clock time
  RCL_STEADY_TIME
} rcl_clock_type_t;

```

# 默认时间源

```c
TEST_F(TestTime, clock_type_access) {
  rclcpp::Clock ros_clock(RCL_ROS_TIME);
  EXPECT_EQ(RCL_ROS_TIME, ros_clock.get_clock_type());

  rclcpp::Clock system_clock(RCL_SYSTEM_TIME);
  EXPECT_EQ(RCL_SYSTEM_TIME, system_clock.get_clock_type());

  rclcpp::Clock steady_clock(RCL_STEADY_TIME);
  EXPECT_EQ(RCL_STEADY_TIME, steady_clock.get_clock_type());
}

```

```c
TEST_F(TestTime, time_sources) {
  using builtin_interfaces::msg::Time;
  rclcpp::Clock ros_clock(RCL_ROS_TIME);
  Time ros_now = ros_clock.now();
  EXPECT_NE(0, ros_now.sec);
  EXPECT_NE(0u, ros_now.nanosec);

  rclcpp::Clock system_clock(RCL_SYSTEM_TIME);
  Time system_now = system_clock.now();
  EXPECT_NE(0, system_now.sec);
  EXPECT_NE(0u, system_now.nanosec);

  rclcpp::Clock steady_clock(RCL_STEADY_TIME);
  Time steady_now = steady_clock.now();
  EXPECT_NE(0, steady_now.sec);
  EXPECT_NE(0u, steady_now.nanosec);
}

```

# 自定义时间源

It is possible that the user may have access to an out of band time source which can provide better performance than the default source the `/clock` topic. It might be possible that for their use case a more advanced algorithm would be needed to propagate the simulated time with adequate precision or latency with restricted bandwidth or connectivity. The user will be able to switch out the time source for the instance of their Time object as well as have the ability to override the default for the process.

It is possible to use an external time source such as GPS as a ROSTime source, but it is recommended to integrate a time source like that using standard NTP integrations with the system clock since that is already an established mechanism and will not need to deal with more complicated changes such as time jumps.

For the current implementation a `TimeSource` API will be defined such that it can be overridden in code. If in the future a common implementation is found that would be generally useful it could be extended to optionally dynamically select the alternative TimeSource via a parameter similar to enabling the simulated time.

可支持NTP，GPS外部时间源

# /clock和/use_sim_time

The time abstraction can be published by one source on the `/clock` topic. The topic will contain the most up to date time for the ROS system. If a publisher exists for the topic, it will override the system time when using the ROS time abstraction. If `/clock` is being published, calls to the ROS time abstraction will return the latest time received from the `/clock` topic. If time has not been set it will return zero if nothing has been received. A time value of zero should be considered an error meaning that time is uninitialized.

与ROS1保持一致，在galactic中支持 ros2 bag play --clock 发送模拟的时间

如果/use_sim_time=1，ROS_TIME时间是0，除非收到/clock时间信息，在模拟场景下，时间会被模拟时间替换。

如果时间是0，则可以认为时间系统发生错误。

For calculations of time durations when using simulation time, clients should always wait until the first non-zero time value has been received before starting, because the first simulation time value from /clock topic may be a high value.

# 报错

不同的时间源的时间是不能比较的，但是这个错误只能是运行时错误，编译期不能检查到该错误

```text
[object_collision_estimator_node_exe-18] terminate called after throwing an instance of 'std::runtime_error'
[object_collision_estimator_node_exe-18]   what():  can't subtract times with different time sources

```

# rosbag

rosbag录制的时候将/clock录制下来呢？实际运行的系统中是没有/clock的，/clock是仿真使用的时间

ROS_TIME在没有/clock的时候使用SYS_TIME，在有/clock的时候使用/clock的时间

# 与ROS1不同之处

ROS1通过 ros::time和ros::wall_time来提供不同种类的时间，没有rclcpp-walltime

ROS2通过时间源类型来提供不同种类的时间

# API

为了提供一个时间接口，ROS提供ROSTime和Duration的数据类型，通过clock模块提供时间数值，Timesource时间源

galactic/src/ros2/rclcpp/rclcpp/include/rclcpp/time.hpp

galactic/src/ros2/rclcpp/rclcpp/include/rclcpp/clock.hpp

galactic/src/ros2/rclcpp/rclcpp/test/rclcpp/test_time.cpp

galactic/src/ros2/rcl/rcl/include/rcl/time.h

galactic/src/ros2/rcutils/include/rcutils/time.h

## Public API

The implementation from client library will provide `Time`, `Duration`, and `Rate` datatypes, for all three time source abstractions.The `Clock` will support a `sleep_for` function as well as a `sleep_until` method using a `Duration` or `Time` argument respectively. The implementation will also provide a `Timer` object which will provide periodic callback functionality for all the abstractions.

It will also support registering callbacks for before and after a time jump. The first callback will be to allow proper preparations for a time jump. The latter will allow code to respond to the change in time and include the new time specifically as well as a quantification of the jump.

Any API which is blocking will allow a set of flags to indicate the appropriate behavior in case of time jump. This will allow the user to choose to error immediately on a time jump or choose to ignore. When registering a callback for jumps a filter for the minimum backwards or forwards distance will be possible and well as whether a clock change is to be included.

### RCL implementation

In `rcl` there will be datatypes and methods to implement each of the three time abstractions for each of the core datatypes. However at the `rcl` level the implementation will be incomplete as it will not have a threading model and will rely on the higher level implementation to provide any threading functionality which is required by sleep methods. It also will require appropriate threading to support the reception of `TimeSource` data.

It will provide implementations parallel to the public datastructures and storage which the client library can depend upon and to which it can delegate. The underlying datatypes will also provide ways to register notifications, however it is the responsibility of the client library implementation to collect and dispatch user callbacks.

# 开发者建议

在编程中尽量使用ROS提供的时间和时钟接口，不建议使用其他接口，例如

- std::chrono

- unix-time

- 其他标准库时间

# 社区反馈

## autoware

[__https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/issues/806__](https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/issues/806)

![](https://tcs.teambition.net/storage/312i2b46dd1f707b7e1c4cf82f04fb201b24?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg0NjkzMCwiaWF0IjoxNjcxMjQyMTMwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmkyYjQ2ZGQxZjcwN2I3ZTFjNGNmODJmMDRmYjIwMWIyNCJ9.aYVCjh_WR894Rvj7u8vkY5r390CVVyvqeB0jEr3h064&download=image.png "")

![](https://tcs.teambition.net/storage/312ia6e3f70277846feefd695b85f5d0f9b4?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg0NjkzMCwiaWF0IjoxNjcxMjQyMTMwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmlhNmUzZjcwMjc3ODQ2ZmVlZmQ2OTViODVmNWQwZjliNCJ9.TWYF0PIWHJt_NUMUoDOuSFzxJSHBcF8E8mSJy_hGgqY&download=image.png "")

## robot_localization

[__https://github.com/cra-ros-pkg/robot_localization/issues/622__](https://github.com/cra-ros-pkg/robot_localization/issues/622)

![](https://tcs.teambition.net/storage/312i7a1462533935c21ad018f0220113fd22?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg0NjkzMCwiaWF0IjoxNjcxMjQyMTMwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmk3YTE0NjI1MzM5MzVjMjFhZDAxOGYwMjIwMTEzZmQyMiJ9.sHcSE3XR9O9_Ia233Ou4Zawtb-3foPZHC8knsx4o2rI&download=image.png "")

# 参考

[object_collision_estimator_node fails with 'can't subtract times with different time sources [1 != 2]' (#806) · Issues · Autoware Foundation / Autoware.Auto / AutowareAuto · GitLab](https://gitlab.com/autowarefoundation/autoware.auto/AutowareAuto/-/issues/806)

[Robot_Localization can't subtract times with different time sources - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/380328/robot_localization-cant-subtract-times-with-different-time-sources/)

[[ROS2] msg stamp time difference - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/321436/ros2-msg-stamp-time-difference/)

[Ros rolling different time sources · Issue #622 · cra-ros-pkg/robot_localization](https://github.com/cra-ros-pkg/robot_localization/issues/622)

[Difference between std::system_clock and std::steady_clock?](https://stackoverflow.com/questions/13263277/difference-between-stdsystem-clock-and-stdsteady-clock)
