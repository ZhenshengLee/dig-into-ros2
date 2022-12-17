# LTTng调试方法

目标是LTTng替代gdb进行调用栈调试，并能够查看进行性能测试

可以使用numpy, panda等传统工具进行数据可视化

也可以使用专用工具babletrace或者tracecompass进行可视化

本文档不使用ros2-tracing工具，直接在应用程序中打桩。

## 安装

```bash
$ sudo apt-add-repository ppa:lttng/stable-2.10
$ sudo apt-get update
$ sudo apt-get install lttng-tools lttng-modules-dkms liblttng-ust-dev

$ sudo usermod -aG tracing $USER

sudo apt-get install python3-babeltrace python3-lttng

```

## 权限使能

```bash
sudo sysctl kernel.perf_event_paranoid=0
```

## 代码准备

Compile your application with the GCC options -g O2 -finstrument-functions

```text
./configure CFLAGS="-g -O2 -finstrument-functions"
```

## 打桩

## 配置和执行检测

### 配置选项

指定进程

Enable the vpid, vtid, and procname LTTng userspace contexts

Preload the liblttng-ust-cyg-profile.so

session设置

python配置

lttng-cli

## 后处理

### babeltrace

```bash
cd ~/.ros/tracing/
babeltrace my-tracing-session/

babeltrace my-tracing-session/ust/

```

babeltrace将trace文件转化为可读的格式，再使用脚本进行处理，可以使用pandas进行绘图

raw Common Trace Format (CTF) data, which is a list of events. Each event has a timestamp, an event type, some information on the process that generated the event, and the fields corresponding to the event type.

### tracecompass

### panda

其他工具略，直接用别人的py脚本即可

# Trace文件格式

 Common Trace Format (CTF) is a binary trace format designed to be very fast to write without compromising great flexibility.

BTF Best Trace Format是Vector公司定义的Trace格式

# 参考

[https://gitlab.com/ros-tracing/ros2_perf_analysis](https://gitlab.com/ros-tracing/ros2_perf_analysis)

<http://archive.eclipse.org/tracecompass/doc/stable/org.eclipse.tracecompass.doc.user/LTTng-UST-Analyses.html#Call_Stack_View>

<https://lttng.org/viewers/>

[Trace Compass](https://projects.eclipse.org/proposals/trace-compass)
