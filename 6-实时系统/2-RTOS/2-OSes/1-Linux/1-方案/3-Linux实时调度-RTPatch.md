# 介绍

从Linux 2.6.18开始，Linux逐渐具备实时功能，其中大部分来源于`realtime-preempt`补丁集。在这些补丁最终合并到内核主线之前，它们必须通过安装才能达到实时性能。这些补丁命名为：

```text
patch-kernelversion-rtpatchversion
```

可以从[这里](http://www.kernel.org/pub/linux/kernel/projects/rt/)下载。

如果没有补丁且在这些补丁完全合并到主线之前的内核提供了3中抢占类，CONFIG_PREEMPT_NONE，CONFIG_PREEMPT_VOLUNTARY和CONFIG_PREEMPT_DESKTOP，分别表示没有，部分，和考虑降低最坏情况下的调度延迟。

如果没有补丁且在这些补丁完全合并到主线之前的内核还提供了额外的配置表项CONFIG_PREEMPT_RT，如果选择该表项，Linux会转变为正式的实时操作系统。FIFO和RR调度策略会用于运行具有实时优先级且最小调度延迟的线程。

# 补丁作用

The RT-Preempt patch converts Linux into a fully preemptible kernel. The magic is done with:

- Making in-kernel locking-primitives (using [spinlocks](http://en.wikipedia.org/wiki/Spinlock)) preemptible though reimplementation with rtmutexes.

- Critical sections protected by i.e. spinlock_t and rwlock_t are now preemptible. The creation of non-preemptible sections (in kernel) is still possible with raw_spinlock_t (same APIs like spinlock_t).

- Implementing [priority inheritance](https://rt.wiki.kernel.org/index.php/Priority_inheritance "Priority inheritance") for in-kernel spinlocks and semaphores. For more information on [priority inversion](https://rt.wiki.kernel.org/index.php/Priority_inversion "Priority inversion") and priority inheritance please consult Introduction to Priority Inversion.

- Converting interrupt handlers into preemptible kernel threads: The RT-Preempt patch treats soft interrupt handlers in kernel thread context, which is represented by a task_struct like a common user space process. However it is also possible to register an IRQ in kernel context.

- Converting the old Linux timer API into separate infrastructures for high resolution kernel timers plus one for timeouts, leading to user space POSIX timers with high resolution.

# 测试

```bash
sudo apt install rt-tests

sudo cyclictest -p 80 -t5 -n

```

# 参考

[Linux RT(1)－硬实时Linux(RT-Preempt Patch)在PC上的编译、使用和测试_宋宝华-CSDN博客](https://blog.csdn.net/21cnbao/article/details/8038279)

[Frequently Asked Questions - RTwiki](https://rt.wiki.kernel.org/index.php/Frequently_Asked_Questions)
