linux trace toolkit next generation

跟踪器

可以帮助调试！

The rcl and rclcpp instrumentation has been integrated into Eloquent, so we simply need to recompile ros2_tracing & compile tracetools_analysis.

rcl和rclcpp的源代码不需要foxy自带打桩了

## 参考

[https://lttng.org/docs/#doc-ubuntu-ppa](https://lttng.org/docs/#doc-ubuntu-ppa)

<https://gitlab.com/ros-tracing/ros2_perf_analysis/-/blob/master/executor_profiling/post.md>

<https://answers.ros.org/question/327477/ros2-uses-6-times-more-cpu-than-fastrtps/>

## 为什么需要Profiler

ROS2 uses 6 times more cpu than FastRTPS?

ROS2占用了过多CPU资源

## 为什么LTTng适合ROS2

由于ROS2客户端高频率发送小消息，所以对于 traditional profiling (which adds lots of overhead) and for sampling-based profiling (which may not notice these)，都不是很好。

尤其是GCC打桩，基本上程序就不能用了，谈何调试？gcc -finstrument-functions，后程序执行10s，没有的话只执行100ms

所以选择单独打桩，只在某些需要监控的函数位置手动打桩。（日志，ASSERT，TRACE都是手动打桩）

总之，LTTng是一个不需要-G就可以进行Profiler的工具。

使用LTTng两个条件，局部打桩，RelWithDbginfo

```text
rcl_ret_t

__attribute__((instrument_function))

rcl_wait(rcl_wait_set_t * wait_set, int64_t timeout)

{

  RCL_CHECK_ARGUMENT_FOR_NULL(wait_set, RCL_RET_INVALID_ARGUMENT);

```

## 安装

```text
sudo apt-add-repository ppa:lttng/stable-2.12
sudo apt update
sudo apt-get install lttng-tools lttng-modules-dkms liblttng-ust-dev python3-lttngust python3-babeltrace

sudo usermod -aG tracing $USER
reboot

```

## Trace概念

Linux软件开发需要很多工具来确保软件可靠性，具备好的性能。

Debugger, GDB并不能让你的软件更快，只能帮助你找到BUG。

Profiler, profiler一般用来找打性能瓶颈，适用于找到性能损耗最大的地方。profiler可以输出一个图，用数据表示观察到的事件，这样你就可以看到软件在哪个事件上消耗更多的时间。

不过，**Profiler无法报告为什么某个函数消耗很长或很大的资源**。同时，profiler也无法记录执行的历史。statistical profiler， event-based profiler。

这就引出了Tracing技术，tracing技术用来理解当前运行的软件系统在发生什么。Tracer可以理解为一个录音机。，当打开录制功能时，，软件中特定的打桩代码产生一系列事件，保存在一个文件中，这个文件叫做tracing file。通过这个文件可以追踪应用软件和操作系统软件的行为。

Tracing和logging也很像。**但是tracer和Logger是完全不同的，服务于不同的软件需求**。Tracer用于记录底层事件，记录更加频繁，每秒钟达到1000次，执行开销比较小。Logger对于高层分析更加适合，一般用于记录错误，用户访问，数据库访问，而这些需求只是Tracing众多需求中的一个。

Trace记录文件从高层分析，就像读log文件一样，但是由于tracing文件记录信息更多多元化，所以可以做到更多更有趣的分析，针对特定的需求生成数据和图像。Traceviewer和analyzers就是用来做这个的。

LTTng就是这样一个工具，可以分析内核和应用程序的工作！

## LTTng概念

总的过程是：lttng cli发送命令到session daemon

### tracing session

一个独立的跟踪会话

一个会话有四种工作模式：

- 本地模式

- 网络流模式

- 快照模式

- 现场模式

- 这个模式下允许实时查看事件

### tracing domain

跟踪域是事件的来源，有五种来源：

- Linux kernel

- User space

- java.util.logging

- log4j

- Python

必须明确跟踪域，因为命名的检测点可能会重名

可以在LinuxKernel和UserSpace中创建channel，其他域只有默认的channel

### channel and ringbuffer

channel是一系列环形缓冲区的集合，每个环形缓冲区分为若干个子缓冲区。当tracer发射出一个事件，将其记录到一个子缓冲区中。当缓冲区空间不足时，根据预定义的规则决定是否新建一个缓冲区。

channel拥有事件规则，作为将事件记录到缓冲区的条件。

### event rule

事件规则是事件满足记录标准的条件，事件规则属于Channel

在lttng2.12中，可用的条件有：

- 事件规则使能

- 打桩点类型匹配instrumentation point type

- 打桩点名字匹配event name

- 打桩点记录等级匹配

- 载荷的字段符合过滤器标准The fields of the payload of the event satisfy a filter expression F

过滤器标准是为了满足动态控制需求

![](https://tcs.teambition.net/storage/312656ada634e6e70d9c434c20ec195b0943?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MDY2NSwiaWF0IjoxNjcxMjQ1ODY1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY1NmFkYTYzNGU2ZTcwZDljNDM0YzIwZWMxOTViMDk0MyJ9.2Vjd51Ne1-yd1cfH3NxcWYsYqDnO4Abb0DImdjTmnQI&download=image.png "")

## 打桩

有多种打桩方法，最直接的方法是在代码中嵌入tracepoints，还有一种办法是在tracing domain中嵌入打桩点

如果只对linux kernel跟踪感兴趣，打桩点已经预先定义好了，在Linux kernel tracepoints中。下面介绍的是在用户态程序中打桩，这些方法包括：

- c/c++程序的打桩

- 预编译的用户态tracing helpers

- Java agent

- Python agent

- LTTng logger

- LTTng kernel tracepoints

此处重点介绍第一种

基于ust的trace的步骤：

- 创建tracepoint privider package

- 在源代码中添加tracepoints

- 编译链接

如果你需要快捷的方法，那么可以略过上述步骤直接使用`tracef()`或者`tracelog()`

### 创建privider package

tracepoint provider是一系列的函数，提供tracepoints，这些函数可以发射事件，`tracepoint()`调用这些函数。

provider是库，包含c和h文件，依赖`liblttng-ust`

![](https://tcs.teambition.net/storage/3126b106394468220dfe89bc4f79e28f7364?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MDY2NSwiaWF0IjoxNjcxMjQ1ODY1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZiMTA2Mzk0NDY4MjIwZGZlODliYzRmNzllMjhmNzM2NCJ9.WfXyLuj0Foi2vGi1aiWVwv4sl2yz1UMJni4XaKnoK4o&download=image.png "")

创建TD跟踪定义，需要包含下列属性：

- 输入参数

- 用户APP的输入数据

- 输出事件字段

- 额外信息

provider_name:tracepoint_name共同组成event name

有了这些定义，就可以在用户代码中添加了

```text
tracepoint(my_provider, my_tracepoint,
           my_structure, some_ratio, the_query);

```

还可以设置Log_level

### tracef和tracelog快捷方法和局限性

局限性：

- provider和tracepoint name是预定义的，不能更改

- 没有类型检查

- 使用了printf，所以会影响性能

所以只能用于暂时使用，在正式代码中不能使用。

### tracing helpers

## CLI

<https://lttng.org/docs/v2.12/#doc-start-sessiond>

```text
lttng create my-session
lttng list
列出所有会话
lttng set-session new-session
更改当前会话
lttng destroy
销毁当前会话
lttng list --userspace
lttng list --kernel --syscall
创建事件规则
lttng enable-event --kernel sched_switch
ttng enable-event --kernel --syscall open,write,read,close
lttng enable-event --kernel --all \

                 --filter='$ctx.tid == 1988 || $ctx.tid == 1534'
满足某个provider的
lttng enable-event --userspace my_app:'*' --loglevel=TRACE_INFO
禁止某个事件规则
lttng disable-event --python my-logger
lttng status
开始记录
lttng start
lttng stop
清除会话记录内容
lttng clear
lttng clear --all

```

## 技巧

### channel

### context fields

额外的上下文信息帮助你过滤

lttng add-context --list

![](https://tcs.teambition.net/storage/312644e14b5e968ea1e68eb73bf00505ed4c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MDY2NSwiaWF0IjoxNjcxMjQ1ODY1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY0NGUxNGI1ZTk2OGVhMWU2OGViNzNiZjAwNTA1ZWQ0YyJ9.KVecwE9vaDUfXh5hXTEsrqK2BEeRLj15DEog3eF8ULw&download=image.png "")

```text
lttng add-context --userspace --type=vpid --type=perf:thread:cpu-cycles
lttng add-context --userspace --type=perf:thread:raw:r0110:test

lttng add-context --kernel --type=perf:cpu:raw:r0013c:x86unhalted

```

### 额外的进程信息

```text
lttng track --userspace --vpid=3,4,7,10,13
```

![](https://tcs.teambition.net/storage/31269f94564129a08b98ec5e6c6d8c273647?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MDY2NSwiaWF0IjoxNjcxMjQ1ODY1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY5Zjk0NTY0MTI5YTA4Yjk4ZWM1ZTZjNmQ4YzI3MzY0NyJ9.Ey8yOG3LWdZDstgWGQurU2Nz7X_2bRFHfFUwO7gpGpM&download=image.png "")

### 保存和加载会话配置

```text
lttng save my-session
lttng load my-session
LTTng saves tracing session configurations to $LTTNG_HOME/.lttng/sessions by default. Note that the LTTNG_HOME environment variable defaults to $HOME if not set. Use the --output-path option to change this destination directory.
```

### 快照

```text
lttng create my-session --snapshot
lttng snapshot record --name=my-first-snapshot
```

### MI

<https://lttng.org/viewers/>

前端

### 录制离线分析

dump and record the state dump

## 分析示例

<https://gitlab.com/ros-tracing/ros2_perf_analysis/-/blob/master/executor_profiling/post.md>
