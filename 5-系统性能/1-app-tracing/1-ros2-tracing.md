lttng-based tracing in ros2

借助ros2 tracing工具基于lttng调试ros2应用程序

# ros2-tracing概述

软件难以调试，其本质是软件内部状态是可变的。所以需要一个工具持续记录软件的内部状态。

为了确保软件行为是确定性的，不超过资源界限，而且可以后续分析。

Tracing是一个记录运行时数据的方法，并且已经集成到OS，比如我们可以记录进程调度，IO的时刻。

我们需要安装lttng工具

![](https://tcs.teambition.net/storage/3126fec3829afc0585d1ddb97ffee1c1c8f2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MDgyNSwiaWF0IjoxNjcxMjQ2MDI1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZmZWMzODI5YWZjMDU4NWQxZGRiOTdmZmVlMWMxYzhmMiJ9.Aka7gbgG9bHCNzUX4WUQmZFqKJojXlKl-nPmU_l5sFw&download=image.png "")

# tp添加剖析

```bash
ga_deps/ros2_tracing/tracetools/include/tracetools/tracetools.h
用户调用代码封装宏
ga_deps/ros2_tracing/tracetools/include/tracetools/tp_call.h
是打桩点定义
这么做是为了方便使能
```

# 命令大全

```bash
press enter to start...^Czs@zs-3630:~$ ros2 trace -h
usage: ros2 trace [-h] [-s SESSION_NAME] [-p PATH] [-u [EVENT [EVENT ...]]] [-k [EVENT [EVENT ...]]]
                  [-c [CONTEXT [CONTEXT ...]]] [-l]
Trace ROS nodes to get information on their execution

optional arguments:

  -h, --help show this help message and exit

  -s SESSION_NAME, --session-name SESSION_NAME

                        the name of the tracing session (default: session-YYYYMMDDHHMMSS)

  -p PATH, --path PATH path of the base directory for trace data (default: ~/.ros/tracing/)

  -u [EVENT [EVENT ...]], --ust [EVENT [EVENT ...]]

                        the userspace events to enable (default: see tracetools_trace.tools.names) [to

                        disable all UST events, provide this flag without any event name]

  -k [EVENT [EVENT ...]], --kernel [EVENT [EVENT ...]]

                        the kernel events to enable (default: see tracetools_trace.tools.names) [to

                        disable all kernel events, provide this flag without any event name]

  -c [CONTEXT [CONTEXT ...]], --context [CONTEXT [CONTEXT ...]]

                        the context names to enable (default: see tracetools_trace.tools.names) [to

                        disable all context names, provide this flag without any name]

  -l, --list display lists of enabled events and context names (default: False)

zs@zs-3630:~$ ros2 trace -l

UST tracing enabled (14 events)

    ros2:rcl_init
    ros2:rcl_node_init
    ros2:rcl_publisher_init
    ros2:rcl_subscription_init
    ros2:rclcpp_subscription_init
    ros2:rclcpp_subscription_callback_added
    ros2:rcl_service_init
    ros2:rclcpp_service_callback_added
    ros2:rcl_client_init
    ros2:rcl_timer_init
    ros2:rclcpp_timer_callback_added
    ros2:rclcpp_callback_register
    ros2:callback_start
    ros2:callback_end
kernel tracing enabled (4 events)
    sched_switch
    kmem_mm_page_alloc
    kmem_mm_page_free
    power_cpu_frequency
context (3 names)
    procname
    vpid
    vtid

```

# 配置和执行

```bash
export ROS_TRACE_DIR=${HOME}/zs_ws/
# Otherwise, use $ROS_HOME/tracing, using ~/.ros for ROS_HOME if not set or if empty.
ros2 run tracetools status

ros2 launch tracetools_launch example.launch.py

ros2 trace-analysis process ~/.ros/tracing/my-tracing-session/ust/

ros2 trace
ros2 launch tracetools_launch example.launch.py
ros2 trace-analysis process ~/.ros/tracing/my-tracing-session/ust/

```

# 参考

<https://lttng.org/>

<https://micro.ros.org/docs/tutorials/advanced/tracing/>

[https://github.com/ZhenshengLee/ros2_tracing-analysis-example](https://github.com/ZhenshengLee/ros2_tracing-analysis-example)

# 其他分析指标

![](https://tcs.teambition.net/storage/312hdc9479ae57a2e90672c9e11a4da78448?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MDgyNSwiaWF0IjoxNjcxMjQ2MDI1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhkYzk0NzlhZTU3YTJlOTA2NzJjOWUxMWE0ZGE3ODQ0OCJ9.ZxXX2DU6L38bdVWiMDF8dCl0IF1tl_RQkcqu8EoGr80&download=image.png "")

![](https://tcs.teambition.net/storage/312h2f63b05b1ccf855414f3ef1a74aa3987?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MDgyNSwiaWF0IjoxNjcxMjQ2MDI1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgyZjYzYjA1YjFjY2Y4NTU0MTRmM2VmMWE3NGFhMzk4NyJ9.M2Rul0nQJ68aWJ3iUOJKXzOaadm4Y5Fg3qQDJCffPsg&download=image.png "")

# 两个节点回调间的延迟

这是e2e-latency？不是

![](https://tcs.teambition.net/storage/312hbe1046ee47cb0d003bbcba5617ea16ac?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MDgyNSwiaWF0IjoxNjcxMjQ2MDI1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhiZTEwNDZlZTQ3Y2IwZDAwM2JiY2JhNTYxN2VhMTZhYyJ9.bdesvmGY8DYQDTKLRqRiCJc4-5NssRaLRfa-dXT5oh4&download=image.png "")

# 定时器回调执行时间

get_timer_callback_range

# 代码功能

data_util

data_util.get_publish_instances()

data_util.get_callback_durations(timer_obj)

貌似没发现别的功能了

# 参考

[ros2_tracing-analysis-example/analyze.py at main · ros2middleware/ros2_tracing-analysis-example](https://github.com/ros2middleware/ros2_tracing-analysis-example/blob/main/analyze.py)

[ros-world-2021-demo/demo.ipynb at main · ros2middleware/ros-world-2021-demo](https://github.com/ros2middleware/ros-world-2021-demo/blob/main/ros_world_2021_demo/analysis/demo.ipynb)
