ros2中的性能测试

时延，总体内存占用

由于ROS2中的component开发移植比较困难，暂时不测试intra性能，主要测试zero_copy-ipc性能。

主要测试点云传输性能！还要测试ros1的特性！

# 安装绘图工具

# 画图工具

注意：测量cpu占用率需要删除第一行异常值

```bash
sudo apt-get install python3 python3-pip texlive texlive-pictures texlive-luatex
sudo -H pip3 install performance_test/helper_scripts/apex_performance_plotter
perfplot <logfile_name>

```

# 中间件测试

不包含rclcpp性能测试，不包含ros2 层开销的通信性能测试，即不包括rclcpp和rmw层的开销

## 选择中间件

```bash
./install/performance_test/lib/performance_test/perf_test -c iceoryx -l log -t Array1k --max_runtime 10
./install/performance_test/lib/performance_test/perf_test -c FastRTPS -l log -t Array1k --max_runtime 10
./install/performance_test/lib/performance_test/perf_test -c  CycloneDDS -l log -t Array1k --max_runtime 10

```

## 选择transport

例子：iceoryx-zero-copy

```bash
./colcon/install/performance_test/lib/performance_test/perf_test -c iceoryx --zero-copy --msg Array1k -p 0 -s 1 -l log_sub
./colcon/install/performance_test/lib/performance_test/perf_test -c iceoryx --zero-copy --msg Array1k -p 1 -s 0 -l log_pub
```

## 非dds测试

NOTE: The iceoryx plugin is not a DDS implementation. The DDS-specific options (such as domain ID, durability, and reliability) do not apply. For the iceoryx plugin, RouDi must be running.

仿照iceoryx做eCAL的测试开发

# rclcpp性能测试

理论上是rclcpp+rmw的开销，但是rmw主要做只是接口转换

## 切换中间件

```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
export RMW_IMPLEMENTATION=rmw_fastrtps_dynamic_cpp
export RMW_IMPLEMENTATION=rmw_iceoryx_cpp
export RMW_IMPLEMENTATION=rmw_ecal_dynamic_cpp
export RMW_IMPLEMENTATION=rmw_ecal_proto_cpp
```

## 默认qos

```bash
# -p publisher num
# -s subscriber num
# -t topic_name
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --zero-copy --msg Array1k -p 0 -s 1 -l log_sub
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --zero-copy --msg Array1k -p 1 -s 0 -l log_pub
```

## 切换qos配置（最终脚本）

先sub后pub

默认reliable, volatile durability, keep last history(10)

```bash
# 相机工况
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg Array1m -r 30 -p 0 -s 1 --history-depth 10 --keep-last -l log_sub_array1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg Array1m -r 30 -p 1 -s 0 -l log_pub_array1m

# 指令工况
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg Struct16 -r 50 -p 0 -s 1 --reliable -l log_sub_struct16
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg Struct16 -r 50 -p 1 -s 0 --reliable -l log_pub_struct16

# 点云工况
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg PointCloud1m -r 10 -p 0 -s 1 --history-depth 1 --keep-last -l log_sub_pc1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg PointCloud1m -r 10 -p 1 -s 0 -l log_pub_pc1m

# zerocopy专用
# 相机工况
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --zero-copy --msg Array1m -r 30 -p 0 -s 1 --history-depth 10 --keep-last -l log_sub_array1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --zero-copy --msg Array1m -r 30 -p 1 -s 0 -l log_pub_array1m
# 指令工况
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --zero-copy --msg Struct16 -r 50 -p 0 -s 1 --reliable -l log_sub_struct16
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --zero-copy --msg Struct16 -r 50 -p 1 -s 0 --reliable -l log_pub_struct16
# 点云工况
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --zero-copy --msg PointCloud1m -r 10 -p 0 -s 1 --history-depth 1 --keep-last -l log_sub_pc1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --zero-copy --msg PointCloud1m -r 10 -p 1 -s 0 -l log_pub_pc1m

./colcon/install/performance_test/lib/performance_test/perf_test
```

## 切换executor

```bash
# 默认工况
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg Array1k -p 0 -s 1 -l log_sub
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg Array1k -p 1 -s 0 -l log_pub

./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-static-single-threaded-executor --msg Array1k -p 0 -s 1 -l log_sub
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-static-single-threaded-executor --msg Array1k -p 1 -s 0 -l log_pub

./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-waitset --msg Array1k -p 0 -s 1 -l log_sub
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-waitset --msg Array1k -p 1 -s 0 -l log_pub

# 点云工况depth=1
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg PointCloud1m -r 10 -p 0 -s 1 --history-depth 1 --keep-last -l log_sub_pc1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg PointCloud1m -r 10 -p 1 -s 0 -l log_pub_pc1m

./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-static-single-threaded-executor --msg PointCloud1m -r 10 -p 0 -s 1 --history-depth 1 --keep-last -l log_sub_pc1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-static-single-threaded-executor --msg PointCloud1m -r 10 -p 1 -s 0 -l log_pub_pc1m

./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-waitset --msg PointCloud1m -r 10 -p 0 -s 1 --history-depth 1 --keep-last -l log_sub_pc1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-waitset --msg PointCloud1m -r 10 -p 1 -s 0 -l log_pub_pc1m


# 点云工况depth=10
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg PointCloud1m -r 10 -p 0 -s 1 --history-depth 10 --keep-last -l log_sub_pc1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg PointCloud1m -r 10 -p 1 -s 0 -l log_pub_pc1m

./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-static-single-threaded-executor --msg PointCloud1m -r 10 -p 0 -s 1 --history-depth 10 --keep-last -l log_sub_pc1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-static-single-threaded-executor --msg PointCloud1m -r 10 -p 1 -s 0 -l log_pub_pc1m

./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-waitset --msg PointCloud1m -r 10 -p 0 -s 1 --history-depth 10 --keep-last -l log_sub_pc1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-waitset --msg PointCloud1m -r 10 -p 1 -s 0 -l log_pub_pc1m

# 默认工况小数据
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg Struct16 -p 0 -s 1 -l log_sub_s16
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg Struct16 -p 1 -s 0 -l log_pub_s16

./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-static-single-threaded-executor --msg Struct16 -p 0 -s 1 -l log_sub_s16
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-static-single-threaded-executor --msg Struct16 -p 1 -s 0 -l log_pub_s16

./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-waitset --msg Struct16 -p 0 -s 1 -l log_sub_s16
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-waitset --msg Struct16 -p 1 -s 0 -l log_pub_s16

# events-executor
# 默认工况
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-events-executor --msg Array1k -p 1 -s 10 -l log_sub
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg Array1k -p 1 -s 10 -l log_sub

# 点云工况depth=1
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-events-executor --msg PointCloud1m -r 10 -p 1 -s 10 --history-depth 1 --keep-last -l log_sub_pc1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg PointCloud1m -r 10 -p 1 -s 10 --history-depth 1 --keep-last -l log_sub_pc1m

# 点云工况depth=10
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-events-executor --msg PointCloud1m -r 10 -p 1 -s 10 --history-depth 10 --keep-last -l log_sub_pc1m
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg PointCloud1m -r 10 -p 1 -s 10 --history-depth 10 --keep-last -l log_sub_pc1m

# 默认工况小数据
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-events-executor --msg Struct16 -p 1 -s 10 -l log_sub_s16
./colcon/install/performance_test/lib/performance_test/perf_test -c rclcpp-single-threaded-executor --msg Struct16 -p 1 -s 10 -l log_sub_s16



```

## 带上graph信息

```text
ros2 run
```

# 自动测试脚本

## 环境

```bash
# python3.8
sudo -H pip install bokeh selenium pandas
sudo apt install firefox-geckodriver

sudo -H pip3 install --upgrade requests
# sudo -H pip install panda==1.0.3 --upgrade -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install panda==1.0.3
```

全局目录是/usr/local/lib/python3.8/dist-packages

site-packages

pip.conf可配置user安装目录，默认是~/.local

[install]

install-option=--prefix=~/.local

查看安装目录

```bash
python -m site
python -m site --user-site
```

## 命令

```bash
rm -rf perf_logs/

# Run perf_test for each experiment in the yaml file
# generate the log files
ros2 run performance_report runner \
  --log-dir perf_logs \
  --test-name experiments \
  --configs ./ga_deps/performance_test/performance_report/cfg/runner/run_one_experiment.yaml

# generate plots
ros2 run performance_report plotter \
  --log-dir perf_logs \
  --configs ga_deps/performance_test/performance_report/cfg/plotter/plot_one_experiment.yaml

# generate reports
ros2 run performance_report reporter \
  --log-dir perf_logs \
  --configs ga_deps/performance_test/performance_report/cfg/reporter/report_one_experiment.yaml
```

[__https://github.com/ros-realtime/performance_test_rt_cfg__](https://github.com/ros-realtime/performance_test_rt_cfg)

## 自动脚本示例

[ROS 2 Real-Time benchmarks](https://ros-realtime.github.io/ros2_realtime_benchmarks/)

# jetson环境

仅包括构建和自动化运行脚本

```bash
sudo apt install default-jre
sudo -H pip3 install --ignore-installed PyYAML

```

## 编译问题

```bash
export COLCON_EXTENSION_BLOCKLIST=colcon_core.event_handler.desktop_notification
```

# 参考

<https://www.apex.ai/post/performance-testing-in-ros-2>

<https://github.com/ZhenshengLee/performance_test>

<https://discourse.ros.org/t/ros2-latency-using-different-node-setups/18335>
