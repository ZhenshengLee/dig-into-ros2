# 实时系统开发指南

## 参考

[https://design.ros2.org/articles/realtime_background.html](https://design.ros2.org/articles/realtime_background.html)

[https://design.ros2.org/articles/realtime_proposal.html](https://design.ros2.org/articles/realtime_proposal.html)

APEX.AI PPT

## ROS2如何编写实时软件

机器人系统需要非常好的响应性，响应性意味着响应时间必须在毫秒级。

### 实时计算的定义

实时性包括

确定性(determinism)：系统是确定的，给定的输入，总是产生指定的输出，不具备确定性的系统具有随机变量，或者行为具有随机性

（ROS1的核心系统，比如launch过程是不确定的，其导航算法是基于概率的，所以也是不确定的）

截至日期（deadline）：一个任务必须在指定时刻前完成

服务质量QOS：网络通信的总体质量，包括带宽，速率，可用性，抖动，延迟，误码率

实时系统就是在正确的时间做正确的事情

硬实时系统，拥有严格的截至日期，一旦越过截至日期，则系统失效。典型的硬实时系统包括飞机飞行控制系统，航天器控制系统

软实时系统，有截至日期，但是越过截至日期系统不会失效，往往会通过降低服务质量的方式恢复响应性。例子包括：音视频系统

准实时系统，把越过截至日期的数据信息当作非法值，和软实时系统一样，会降低服务质量，例子有：金融预测系统，机器人装配线

### 实时系统与低延迟系统

往往是紧密关联的。但是实时系统和低延迟没有绝对关联关系，时间长短没有关系，但是时刻准确非常重要，不早不晚恰好。

但是实时系统一定要求延迟时间是可以测试的，是可以设置的。

### 实时系统依赖

实时系统依赖计算机系统自上而下都具有实时的特点。

用户代码，中间件，操作系统，CPU都需要支持实时。

### 实时系统环境

RT_PREEMPT Linux补丁，把linux调度器设置为完全可抢占

Xenomai, a POSIX-compliant co-kernel (or hypervisor) that provides a real-time kernel cooperating with the Linux kernel. The Linux kernel is treated as the idle task of the real-time kernel’s scheduler (the lowest priority task).

RTAI, an alternative co-kernel solution.

QNX Neutrino, a POSIX-compliant real-time operating system for mission-critical systems.

### 实时系统开发的最佳实践

操作系统无法保证用户代码是实时的，所以开发者需要接受培训。在这个小节，很多针对操作系统的实时策略开发出来，并且可以应用到ROS2中。这种模式可以扩展到基于Linux的实时系统c++开发。

一般把实时代码分成三个部分：

non real-time safe section：进程开始阶段，与分配内存，启动线程

real-time safe section：通常是循环

non-real-time safe teardown section：回收内存

real-time code path：执行阶段的中间段

#### 内存管理

适当的内存管理，在real-time code path应该避免page faults，在页错误中，CPU会中断从磁盘载入数据到RAM，这很慢。

动态内存分配会大大降低实时性能，Calls to malloc/new and free/delete will probably result in pagefaults

堆内存分配会造成内存碎片，造成读写效率低下。

##### 锁住内存，防止内存换到SWAP分区

mlockall is a Linux system call for locking the process’s virtual address space into RAM, preventing the memory that will be accessed by the process from getting paged into swap space.

```cpp

if (mlockall(MCL_CURRENT|MCL_FUTURE) == -1) {

  perror("mlockall failed");

  exit(-2);

}

unsigned char dummy[MAX_SAFE_STACK];

memset(dummy, 0, MAX_SAFE_STACK);

```

##### 使用动态内存池

```cpp

if (mlockall(MCL_CURRENT | MCL_FUTURE))

  perror("mlockall failed:");

/* Turn off malloc trimming.*/

mallopt(M_TRIM_THRESHOLD, -1);

/* Turn off mmap usage. */

mallopt(M_MMAP_MAX, 0);

page_size = sysconf(_SC_PAGESIZE);

buffer = malloc(SOMESIZE);



for (i=0; i < SOMESIZE; i+=page_size) {

  buffer[i] = 0;

}

free(buffer);

```

make dynamic memory allocation real-time safe (mostly). It locks the virtual address space to a fixed size, disallows returning deallocated memory to the kernel via sbrk, and disables mmap. This effectively locks a pool of memory in the heap into RAM, which prevents page faults due to malloc and free (3).

动态内存池在用户释放内存后不会将内存返回，而是存到一个池子里面

优点：Can use malloc/new, free/delete, and even STL containers

缺点：

平台相关

内存池大小是固定的，需要准确估计

一些STL类型大小是动态增加的，对于内存池来说比较危险

实践中只适用于对内存需求比较小的进程

##### 自定义的实时安全内存分配器

avoid dynamic memory allocation

One such alternative allocator is TLSF (Two-Level Segregate Fit). It is also called the O(1) allocator, since the time cost of malloc, free, and align operations under TLSF have a constant upper bound. It creates a low level of fragmentation. The disadvantages of TLSF are that it is not thread safe and that its current implementation is architecture specific: it assumes the system can make 4-byte aligned accesses.

优点：

安全分配内存

不同分配器有不同的优势

缺点：

分配器没有充分测试，可能会有异常

代码依赖，代码复杂度

不同分配器有不同的劣势

#### Global variables and (static) arrays

全局变量和静态数组，确实是实时安全的，但是全局变量有非常多的缺点，不赘述了

#### Cache friendliness for pointer and vtable accesses

Classes with many levels of inheritance may not be real-time safe because of vtable overhead access.

继承比较多的类，不是实时安全的，因为较大的虚表访问开销，函数分布在内存的各个地方，可能并不全在cache中，导致cache miss

In general, C++ patterns with poor cache locality are not well-suited to real-time environments. Another such pattern is the opaque pointer idiom (PIMPL), which is convenient for ABI compatibility and speeding up compile times. However, bouncing between the memory location for the object and its private data pointer causes the cache to “spill” as it loads one chunk of memory and then another, unrelated chunk for almost every function in the PIMPLized object.

cpp的模式一般来讲都不是实时安全的，但是有一个惯用法，PIMPL可以加速。然后有局限性

#### Exceptions

异常

Handling exceptions can incur a large performance penalty. Running into an exception tends to push a lot of memory onto the stack, which is often a limited resource in real-time programming. But if exceptions are used properly, they should not be a concern to real-time programmers (since they indicate a place in the program with undefined behavior and are integral to debugging) (6).

异常处理导致很大的性能开销，所以不要用异常，用错误码机制。

##### 总结：知道你的内存需求

Know your problem

Different programs have different memory needs, thus memory management strategies vary between applications.

Required memory size known at compile time

在编译时间就知道内存需求

Example: publishing a message of a fixed size.

发布固定大小的消息

Solution: use stack allocation with fixed-size objects.

使用栈分配内存

Required memory size known at runtime, before real-time execution.

在运行时间知道内存需求

Example: publishing a message of a size specified on the command line.

命令行发布消息

Preallocate variable size objects on the heap once required size is known, then execute real-time code.

预分配

Required memory size computed during real-time

在实时运行阶段才知道内存需求

Example: a message received by the robot’s sensors determines the size of the messages it publishes.

Multiple solutions exist

Object pools

TLSF O(1) memory allocation

Use stack allocation and fail if allocated memory is exceeded

#### 设备IO

与物理设备交互，比如磁盘IO，打印到屏幕

由于进程经常需要等待设备，而且IO调用如fopen经常导致页错误。

所以：将磁盘读写操作在程序开始时候或结束时候执行，不要在实时阶段执行！

日志打印线程不要在实时阶段调度！

#### 多线程编程和同步

实时编程需要改变多线程编程的模式。程序执行不能阻塞，线程必须在指定时间内调度。实时操作系统必须满足调度需求。但是开发者需要注意一些开发上的注意事项。

##### 线程创建

在程序开始时创建线程，因为创建线程有额外开销。

Create high priority (but not 99) threads with a FIFO, Round Robin, or Deadline scheduler

特定的线程要有高优先级

##### 避免优先级反转

Priority inversion can occur on a system with a preemptive task scheduler and results in deadlock. It occurs when: a low-priority task acquires a lock and is then pre-empted by a medium-priority task, then a high-priority task acquires the lock held by the low-priority task.

会造成死锁

Here are the some solutions to priority inversion:

Don’t use blocking synchronization primitives

Disable preemption for tasks holding locks (can lead to jitter)

Increase priority of task holding a lock

Use priority inheritance: a task that owns a lock inherits the priority of a task that tries to acquire the lock

Use lock-free data structures and algorithms

使用无锁数据结构

##### timing shots

One real-time synchronization technique is when a thread calculates its next “shot” (the start of its next execution period). For example, if a thread is required to provide an update every 10 milliseconds, and it must complete an operation that takes 3-6 milliseconds, the thread should get the time before the operation, do the operation, and then wait for the remaining 7-4 milliseconds, based on the time measured after the operation.

使用高精度时钟！！！不然系统会有漂移

The most important consideration for the developer is to use a high precision timer, such as **nanosleep** on Linux platforms, while waiting. Otherwise the system will experience drift.

##### 自旋锁

Spinlocks tend to cause clock drift. The developer should avoid implementing his own spinlocks. The RT Preempt patch replaces much of the kernel’s spinlocks with mutexes, but this might not be guaranteed on all platforms.

使用实时补丁提供的自旋锁

##### 不要用fork

fork is not real-time safe because it is implemented using copy-on-write. This means that when a forked process modifies a page of memory, it gets its own copy of that page. This leads to page faults!

### 测试，基准测试

#### cyclictest

命令行工具，测试实时环境。输入多个线程，添加优先级，配置一个调度类型。所有的线程轮盘赌的形式调度，对于每一个线程，cyclictest测量每个线程应该唤醒以及实际唤醒的实践差。时延的变化就是抖动。如果有进程具有不确定的阻塞行为，则平均时延会很大，因为调度器不能满足实践需求。

#### 测试插桩代码

rttest.h

### pagefaults

The Linux system call **getrusage** returns statistics about many resource usage events relevant to real-time performance

## ROS2的设计指南

### 实现策略

With judicious application of the performance patterns and benchmarking tests proposed in this document, implementing real-time code in C/C++ is feasible. The question of how ROS 2 will achieve real-time compatibility remains.

允许ROS节点在启动和结束阶段不是实时安全，但是ROS节点交互，特别是进程内通信，则需要是实时安全的。

### 创建一个配置选项，允许进入实时友好模式

### 实现一个实时的计算客户端rclrt, rmwrt

### 实现一个实时的语言客户端，rclrt, rclc

realtime-cpp

![](https://tcs.teambition.net/storage/312996b0b2e7d4b73e67d88b011f6fa56012?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMzM4NiwiaWF0IjoxNjcxNDE4NTg2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk5NmIwYjJlN2Q0YjczZTY3ZDg4YjAxMWY2ZmE1NjAxMiJ9.b4o_lBUklWGfKa6jvZ0RAXtiGWFCRS6nMYTqkeffjCM&download=image.png "")
