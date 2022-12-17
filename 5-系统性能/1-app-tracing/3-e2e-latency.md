# 流程

![](https://tcs.teambition.net/storage/312h50bdf7ee53684f4ada429e242604759d?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5NywiaWF0IjoxNjcxMjQ2NTk3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg1MGJkZjdlZTUzNjg0ZjRhZGE0MjllMjQyNjA0NzU5ZCJ9.v0eqVMmliy7vPWzsvu0b7fJFe0k42nnvqiqAu5yT8Jw&download=image.png "")

# 几种延迟类型

![](https://tcs.teambition.net/storage/312h0813445ceebd0758af01641d118d6fe4?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5NywiaWF0IjoxNjcxMjQ2NTk3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgwODEzNDQ1Y2VlYmQwNzU4YWYwMTY0MWQxMThkNmZlNCJ9.njJNJfXT50EaVfXXpMDpr-EDNzPZzX3SA8ZJRtdxI94&download=image.png "")

# 如何计算e2e时延

从单点时延到e2e时延

![](https://tcs.teambition.net/storage/312h67cbc2295d2a3d7cfdc1f77c21ed88ec?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5NywiaWF0IjoxNjcxMjQ2NTk3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg2N2NiYzIyOTVkMmEzZDdjZmRjMWY3N2MyMWVkODhlYyJ9.NVksbCc65-4taUG1DHMFcKoXsXluXDLUM__K1ov4WEI&download=image.png "")

为了这个计算公式，需要在现有tracing基础上额外添加一些tp，rcl和rclcpp都要添加

还需要收集ros::time时间

# 安装

安装添加了tp的ros2

```bash
sudo -H pip install pandas fire pygraphviz

wget https://ghproxy.com/https://raw.githubusercontent.com/brendangregg/FlameGraph/master/flamegraph.pl -O ~/.local/bin/flamegraph.pl && chmod +x $_
```

# 版本管理

galactic融合foxy版本

- ros2_tracing: 增加trace point定义 要增加定义。

  - 遗留galactic

[__https://gitlab.com/reishikou/ros2_tracing.git__](https://gitlab.com/reishikou/ros2_tracing.git)  +  [__https://gitlab.com/HasegawaAtsushi/ros2_tracing/-/tree/devel_e2e_measurement__](https://gitlab.com/HasegawaAtsushi/ros2_tracing/-/tree/devel_e2e_measurement)  devel_e2e_measurement

- tracetools_analysis: 分析脚本，没有问题不需要添加

[__https://gitlab.com/reishikou/tracetools_analysis.git__](https://gitlab.com/reishikou/tracetools_analysis.git) + [__https://gitlab.com/HasegawaAtsushi/tracetools_analysis/-/tree/devel_e2e_measurement__](https://gitlab.com/HasegawaAtsushi/tracetools_analysis/-/tree/devel_e2e_measurement) devel_e2e_measurement

- rclcpp: 增加trace point使用， 要添加

  - 遗留galactic

[__https://github.com/reishikou/rclcpp.git__](https://github.com/reishikou/rclcpp.git) + [__https://github.com/hsgwa/rclcpp/tree/devel_e2e_measurement__](https://github.com/hsgwa/rclcpp/tree/devel_e2e_measurement)  devel_e2e_measurement

- rcl: 增加trace point使用， 要添加

  - 遗留foxy: rcl_publisher_init未添加gid(这个最新版没用到)

[__https://github.com/reishikou/rcl.git__](https://github.com/reishikou/rcl.git) + [__https://github.com/hsgwa/rcl.git__](https://github.com/hsgwa/rcl.git)  devel_e2e_measurement

rmw_cyclonedds:  要添加

- 完成

[__https://github.com/hsgwa/rmw_cyclonedds.git__](https://github.com/hsgwa/rmw_cyclonedds.git)   devel_timestamp_for_ros2

cyclonedds: 要添加

- 完成

[__https://github.com/hsgwa/cyclonedds.git__](https://github.com/hsgwa/cyclonedds.git)    devel_timestamp_for_ros2

# 架构图

![](https://tcs.teambition.net/storage/312h375781161d994884f6c4bf38cfa6081b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5NywiaWF0IjoxNjcxMjQ2NTk3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgzNzU3ODExNjFkOTk0ODg0ZjZjNGJmMzhjZmE2MDgxYiJ9.c7EwavVE-Vj-pblLA1cANLm-ptTk_pBt5DW2mrC5BXo&download=rosgraph.png "")

初次运行，打开trace，rqt_graph记录，检查基础功能，生成架构文件

topic1:11hz,  0.09s, 执行0.01秒

topic2:7hz,  0.14s，执行0.01s

topic3:  topic1, 执行0.1s

topic4: topic3, 执行0.1s

topic5: topic2, 执行0.1s

topic6: 3hz, 0.33s, 执行0.1s

# 初次运行

```bash
# t1
ros2 launch e2e_demo demo.launch.py
# t2
# 检查
cd ~/.ros/tracing
# 这三个检测点必须要
babeltrace ./e2e_demo/ | grep -e rclcpp_publish -e rclcpp_subscribe
babeltrace ./e2e_demo/ | grep -e rclcpp_timer_added
# 生产架构文件模板
trace_create_template ./e2e_demo ./arch.json
# 使用ipynb进行分析
# 需要对python脚本熟悉
```

# 生成架构模板

使用命令生成架构模板

```text
trace_create_template ./e2e_demo ./arch.json
```

galactic模板相比foxy模板而言，少了callback_dependency的自动生成，这个跟call_start和call_end有关？

# 手动编写架构文件

![](https://tcs.teambition.net/storage/312h44736d5bf00f1763aa4e379c4e55a54c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5NywiaWF0IjoxNjcxMjQ2NTk3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg0NDczNmQ1YmYwMGYxNzYzYWE0ZTM3OWM0ZTU1YTU0YyJ9.5ipjYjEsQ6X4H7EAFheXgmW57RB1BnIUzRZrFJ5kndA&download=image.png "")

![](https://tcs.teambition.net/storage/312h8e05022f851b92ab2ea423f7db632ab2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5NywiaWF0IjoxNjcxMjQ2NTk3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg4ZTA1MDIyZjg1MWI5MmFiMmVhNDIzZjdkYjYzMmFiMiJ9.XrONevOIgXOav2dikDum9SLTZkLrjXIBb6f61hzGjcg&download=image.png "")

根据rqt_graph生成架构文件

foxy需要手动更新下列信息

![](https://tcs.teambition.net/storage/312h3cc18cab60f207d6050064aa642423c5?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5NywiaWF0IjoxNjcxMjQ2NTk3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgzY2MxOGNhYjYwZjIwN2Q2MDUwMDY0YWE2NDI0MjNjNSJ9.J6rrD9fR-sFWkD71xeQ7Nb_fHklSrPLW9U4jCzDjD28&download=image.png "")

- start node

- end node

- 删除时间录制节点两个

- 每个节点发布主题名字的字符串表达

![](https://tcs.teambition.net/storage/312h441c2c4236fef0f948a669d75a946587?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5NywiaWF0IjoxNjcxMjQ2NTk3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg0NDFjMmM0MjM2ZmVmMGY5NDhhNjY5ZDc1YTk0NjU4NyJ9.cAR9TFNYa2UUN977-wZod0THx7dKk4H9-VogBBoiKy8&download=image.png "")

galactic除了上述新增，还需要增加

还缺少callback_dependency

使用下面命令查看是否正确

```text
trace_draw_node_graph ./arch.json arch.png
```

# 计算

# 局限性

![](https://tcs.teambition.net/storage/312h8506450c2b082fd5a2d0940f8352120c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5NywiaWF0IjoxNjcxMjQ2NTk3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg4NTA2NDUwYzJiMDgyZmQ1YTJkMDk0MGY4MzUyMTIwYyJ9.Iff5X2aW7BfAVOqAbLLjndX5HqLAn8TXrkvBRX4qSs0&download=image.png "")

复杂的路径，估计不行，得人工选取

不同的路径，目前应该是可以的

图形界面，估计不会有了

# 参考

[https://github.com/hsgwa/e2e_demo/tree/master/e2e_demo](https://github.com/hsgwa/e2e_demo/tree/master/e2e_demo)

[https://github.com/hsgwa/e2e_demo/issues/2](https://github.com/hsgwa/e2e_demo/issues/2)

# 相关工作

[Chain-Aware ROS Evaluation Tool (CARET)](https://tier4.github.io/CARET_doc/main/)

这个工作是论文作者hsgwa在tier4开发的框架，比论文更加完善
