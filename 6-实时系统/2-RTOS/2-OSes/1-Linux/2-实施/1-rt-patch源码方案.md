相比于xen微内核方案，rt-patch更简单，但是对资源消耗更加大，其要求所有Linux线程均可被抢占。

在该方案下，可以使dds线程，回调线程都配置实时性

# 参考

[Building realtime Linux for ROS 2 [community-contributed] — ROS 2 Documentation: Galactic  documentation](https://docs.ros.org/en/galactic/Tutorials/Building-Realtime-rt_preempt-kernel-for-ROS-2.html)

[rtwg_documentation/source/Guides/Real-Time-Operating-System-Setup at rolling · ros-realtime/rtwg_documentation](https://github.com/ros-realtime/rtwg_documentation/tree/rolling/source/Guides/Real-Time-Operating-System-Setup)
