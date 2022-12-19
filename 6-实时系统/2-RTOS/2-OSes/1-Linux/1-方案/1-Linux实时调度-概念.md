![](https://tcs.teambition.net/storage/3129b845fdda7f889979142d921e64d97dc6?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMTE4MiwiaWF0IjoxNjcxNDE2MzgyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjliODQ1ZmRkYTdmODg5OTc5MTQyZDkyMWU2NGQ5N2RjNiJ9.NXefvAR8FIt6S4NfLDMzBJ0uwbj8f-0qf0cre3UONws&download=image.png "")

我们都知道对于机器人应用来说实时性很重要, 以及Linux不是一个实时系统, 但是说:

- 实时性到底对我的程序有什么影响?

- 默认情况下, Linux的实时性怎么样?

- rt-linux 有什么用?

# 调度实时

一个简单ROS程序逻辑如下: 以100hz向外发送数据.

```text
while (1) {
  usleep(10000);
  process()；
}
```

我们接下来就用这个程序的`循环周期间隔`来评估这个程序的实时性.

PS: Robotics上常见的高频率基本也就集中在200hz~100hz, 比如说IMU传感器的周期和各种控制的采样周期

默认Linux的调度器为CFS.

在CFS调度器下, ROS应用跑了5分钟, 其中出现的最大延迟为50ms

```text
[ INFO] [1551423662.755795004]: dt: now 10893.000000 max 54892.000000
```

下面我们设置这个应用为rt调度策略

```text
chrt --rr  -p <priority between 1-99> pid
```

跑了5分钟, 其中出现的最大延迟为13ms

```text
[ INFO] [1551425162.716713135]: dt: now 10447.000000 max 13020.000000
```

结论

可以看到, 在默认cfs调度器的情况, 一个程序的周期运行时间在CPU高负载的情况下是得不到保障的.

对关键的程序来说设置rt调度非常有必要.

不然叠加起来, 就有可能出现数百毫秒的延迟.

如果这是一个刹车链路, 那在80km/h的速度下刹车距离就会上下浮动10几米。

RT-Linux

在上面rt调度的情况, 可以看到最大的周期间隔13ms, 而不是严格的10ms.

造成这个3ms的原因有很多, 包括内核不支持抢占等等.

而RT-Linux补丁, 就是为了解决这些原因而产生的.

这里下载补丁, 重新编译内核

```text
https://cdn.kernel.org/pub/linux/kernel/projects/rt/3.14/older/
```

上车继续执行测试程序(rt调度)

跑了5分钟, 其中出现的最大延迟为10.7ms

```text
[ INFO] [1551434073.434294415]: dt: now 10284.000000 max 10701.000000
```

结论

RT-Linux补丁可以解决默认Linux下毫秒级的调度误差

# 程序执行实时

上文研究`循环周期间隔`所针对的都是`调度`上的实时性.

对实时性而言, 像`process时间`也要考虑起来, 这针对的是`程序执行`上的实时性.

一些影响程序实时性的点:

- 内存缺页

  - Linux applications access memory by using virtual addresses. Each virtual address translates into a physical address with the help of hardware that uses translation tables. This feature makes it possible to address more virtual memory than there is physical memory available, assuming that not all applications need all their allocated memory at the same time.

- I/O阻塞

- 多线程同步阻塞

- 其他......

这里不再详细描述, 有兴趣可看以下参考:

- [http://linuxrealtime.org/index.php/Designing_Real-Time_Applications](http://linuxrealtime.org/index.php/Designing_Real-Time_Applications)

- [https://index.ros.org/doc/ros2/Tutorials/Real-Time-Programming/](https://index.ros.org/doc/ros2/Tutorials/Real-Time-Programming/)

- [https://wiki.linuxfoundation.org/realtime/documentation/howto/applications/application_base](https://wiki.linuxfoundation.org/realtime/documentation/howto/applications/application_base)

# 实时分类定量

硬实时环境（Safety-Critical，低延时，比如几十个us内）

软实时（几百个us~几个/几十个ms)

普通的低延迟桌面（几个ms到几百个ms）

# 参考

[实时优化: Linux实时的一些小概念-阿里云开发者社区](https://developer.aliyun.com/article/696449)

[实时 Linux - 泰晓科技](http://tinylab.org/rtlinux/)

[实时 Linux 抖动分析 Step by step - 泰晓科技](http://tinylab.org/rtlinux-latency-tracing/)

[嵌入式 Linux 系统怎样保证实时性 - 泰晓科技](http://tinylab.org/how-to-make-a-linux-system-real-time/)
