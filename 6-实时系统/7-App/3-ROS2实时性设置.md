# 实时性配置

![](https://tcs.teambition.net/storage/312g4de1297aff640a616259398e2d3901ae?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMzQxMywiaWF0IjoxNjcxNDE4NjEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmc0ZGUxMjk3YWZmNjQwYTYxNjI1OTM5OGUyZDM5MDFhZSJ9.LzqY8hDAn5O6w9vaQ6GBxuAOqvk54cNAjJr1mrzl1VI&download=image.png "")

![](https://tcs.teambition.net/storage/312gaa30315dac45ea996f15b0164fdb159f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMzQxMywiaWF0IjoxNjcxNDE4NjEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdhYTMwMzE1ZGFjNDVlYTk5NmYxNWIwMTY0ZmRiMTU5ZiJ9.EXTcOlP649SjHKEjaWoTPox3DnZd3wOhth8nYmXAVb0&download=image.png "")

1. 配置线程profile，接下来新创建的线程都会遵循该profile

1. init()，创建rcl_context数据结构，初始化参数，日志系统，初始化dds实例，daemon，发现系统。

1. dds线程（child threads）初始化，创建node, publisher, subscriber, datawriter, datareader，这些会遵循上述profile

1. 配置线程Profile，接下来新创建的线程都会遵循该profile

1. exec在spin的时候运行，ros执行器线程(ros threads)创建运行，遵循上述profile

![](https://tcs.teambition.net/storage/312gbc356fbce24aa9b4595febd6c2aa4f72?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMzQxMywiaWF0IjoxNjcxNDE4NjEzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdiYzM1NmZiY2UyNGFhOWI0NTk1ZmViZDZjMmFhNGY3MiJ9.M04XipbFIXz8lz8kq2lJJo0WSA-oynTS6ugqDyqm_SQ&download=image.png "")

# 是否配置dds线程实时性

- dds规范自带实时性，通过qos来保证通信质量

- dds可能没有在rt-patch环境下测试过

- 所以一般不怡加上实时性配置

# 参考

[Utilities for real-time tests and applications - community](https://www.gitmemory.com/issue/ros-realtime/community/14/779363816)

在 launch中无法设置这些

[Real-time programming in ROS 2 — ROS 2 Documentation: Foxy  documentation](https://docs.ros.org/en/foxy/Tutorials/Real-Time-Programming.html)

[GitHub - ros2/realtime_support: Minimal real-time testing utility for measuring jitter and latency.](https://github.com/ros2/realtime_support)

可不可以一些节点按照实时策略调度，另一些节点按照普通策略调度？

我认为是可以的

[GitHub - ros2-realtime-demo/pendulum: ROS, ROS2, real-time, control, pendulum](https://github.com/ros2-realtime-demo/pendulum)

launch prefix

[[ros2] Prefix in launch file - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/343326/ros2-prefix-in-launch-file/)

[https://github.com/ros2/launch_ros/issues/37](https://github.com/ros2/launch_ros/issues/37)
