# 调度策略

从Linux 2.6.23开始，默认的调度器为CFS，即"完全公平调度器"(Completely Fair Scheduler)。CFS调度器取代了之前的"O(1)"调度器。

当前的调度器为CFS，以绝对公平为基本理念，也带有对实时进程的支持，其包括多种调度策略。

## Scheduling policies

内核模块使用调度器来决定下一个CPU时钟周期执行的线程。每个线程都包含一个调度策略以及一个静态的调度优先级`sched_priority`，调度器根据系统上所有线程的调度策略和静态优先级来决定如何进行调度。对于使用普通调度策略(`SCHED_OTHER`, `SCHED_IDLE`, `SCHED_BATCH`)的线程来说，`sched_priority`并不会影响调度结果，且**必须设置为0**。对于使用实时策略(`SCHED_FIFO`,`SCHED_RR`)的进程，其`sched_priority`取值为1到99(1为最低值)。实时线程的调度优先级总是高于普通线程。注：POSIX.1的系统在实现中，会要求实时调度策略有32个优先级设置，因此，为了可移植性，可以使用[sched_get_priority_min](http://man7.org/linux/man-pages/man2/sched_get_priority_min.2.html)和[sched_get_priority_max](http://man7.org/linux/man-pages/man2/sched_get_priority_max.2.html)来查找调度策略所支持的优先级范围。调度器会为每个`sched_priority`值维护一个可运行的线程列表。调度器通过查看非空且静态优先级最高的列表，并选择该列表首部的元素作为下一个运行的线程。

线程的调度策略决定了如何根据静态优先级来将一个线程插入到同静态优先级的线程列表(list of runnable threads)中，以及如何在该列表中调整线程的位置。

所有的调度都具有抢占性：如果一个具有更高静态优先级的线程准备运行，当前运行的线程会被抢占并返回到其静态优先级对应的等待列表中。调度策略仅根据具有相同静态优先级的可运行线程列表来决定调度顺序。

*进程调度中使用了2个队列：进程一开始会进入ready队列等待调度；当进程执行中遇到I/O阻塞，等待子进程结束或软中断等原因会进入wait队列，等阻塞结束后会返回到ready队列*

## SCHED_FIFO: First in-first out scheduling(实时线程)

`SCHED_FIFO`仅适用于静态优先级大于0的线程，即当一个`SCHED_FIFO`的线程变为可运行(runnable)状态时，它会立即抢占所有当前运行的`SCHED_OTHER`, `SCHED_BATCH`或`SCHED_IDLE` 线程。`SCHED_FIFO`不使用时间片进行调度，所有使用`SCHED_FIFO`调度策略的线程应该遵守如下规则：

1. 当一个运行中的`SCHED_FIFO`线程被其他有更高优先级的线程抢占后，该线程会返回到其优先级对应的列表的首部，当所有更高优先级的线程阻塞后，该线程将会立即恢复运行；

1. 当一个阻塞的`SCHED_FIFO` 线程变为可运行状态时，该线程会返回到其优先级对应的列表末尾；

`SCHED_FIFO` 线程将会一直运行，直到被更高优先级的线程抢占，或调用了[sched_yield(2)](http://man7.org/linux/man-pages/man2/sched_yield.2.html) 。

## SCHED_RR: Round-robin scheduling(轮询调度)

`SCHED_RR`对`SCHED_FIFO`做了简单增强。除每个线程仅允许运行在一个最大时间段下外，`SCHED_FIFO`中的所有规则都适用于`SCHED_RR`。如果一个`SCHED_RR`线程已经运行了等于或大于该最大时间段时，该线程会被放置到其优先级列表的末尾。当一个`SCHED_RR`线程被更高优先级的线程抢占，并在后续恢复运行后，会在先前未过期的时间段下运行。最大时间段可以通过[sched_rr_get_interval(2)](http://man7.org/linux/man-pages/man2/sched_rr_get_interval.2.html)获得。

抢占，但是可自动让出。

## SCHED_DEADLINE: Sporadic task model deadline scheduling

3.14版本之后的Linux提供了一个新的调度策略`SCHED_DEADLINE`。该策略结合了GEDF(Global Earliest Deadline First)和 CBS (Constant Bandwidth Server)。必须通[sched_setattr(2)](http://man7.org/linux/man-pages/man2/sched_setattr.2.html)和[sched_getattr(2)](http://man7.org/linux/man-pages/man2/sched_getattr.2.html)来设置和获取该策略。一个Sporadic task被定义为一系列任务，且每个任务每次仅激活一次。每个任务都有一个`relative deadline`(该任务应该在该相对时间前停止运行)，以及一个`computation time`(执行该任务需要的CPU时间，对应下图的`comp. time`)。一个新的任务开始执行时会唤醒(wakeup)一个Sporadic task，该时间点被称为`arrival time`，`start time`为一个任务开始执行的时间，`absolute deadline`(绝对截止时间)为`arrival time`加上`relative deadline`的时间点。

![](https://tcs.teambition.net/storage/31296864e8b828a1554db52a6aca4d8c55f5?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMTE3MywiaWF0IjoxNjcxNDE2MzczLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk2ODY0ZThiODI4YTE1NTRkYjUyYTZhY2E0ZDhjNTVmNSJ9.MquGYIYa0UBEbUjdO1F4fnqX2raQYuOLx6msXAwfT2A&download=image.png "")

当使用[sched_setattr(2)](http://man7.org/linux/man-pages/man2/sched_setattr.2.html)给一个线程设置`SCHED_DEADLINE` 策略时，可以设置3个参数：`Runtime`, `Dead‐line`和`Period`，对于上面提到的场景来说，通常的做法是将`Runtime`设置为大于平均计算时间的值(或更坏的场景下，设置为硬实时任务的执行时间)；将`Deadline`设置为对应的dead-line，将`Period`设置为任务的周期，此时对于`SCHED_DEADLINE`的调度如下：`Runtime`对应上图中的`comp.time`，`Dead-line`对应上图的`relative deadline`

![](https://tcs.teambition.net/storage/3129735f2481f5fc56d4c2927b353104c1b4?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMTE3MywiaWF0IjoxNjcxNDE2MzczLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjk3MzVmMjQ4MWY1ZmM1NmQ0YzI5MjdiMzUzMTA0YzFiNCJ9.qJdidj2uUCpyq4_GpNmwDCpFuQxeDG3yLSSE-uWcgEk&download=image.png "")

3个deadline调度参数对应`sched_attr`结构体中的`sched_run‐time`, `sched_deadline`, 和`sched_period`字段，参见[sched_setattr(2)](http://man7.org/linux/man-pages/man2/sched_setattr.2.html)。这些字段的单位为纳秒。如果`sched_period`的值为0，则它与`sched_deadline`相同。

内核要求：

```text
sched_runtime <= sched_deadline <= sched_period
```

此外，在当前实现中，所有参数的值至少为1024(即，大于1微秒)，小于2^63。如果有效性校验失败，[sched_setattr(2)](http://man7.org/linux/man-pages/man2/sched_setattr.2.html)返回EINVAL错误。

CBS通过阻止线程超出其运行时间`Runtime`来保证任务间不互相干扰。为了确保deadline调度，当`SCHED_DEADLINE`线程在给定的条件下不可运行时，此时内核必须阻止这些线程的运行。内核必须在设置或修改`SCHED_DEADLINE`策略和属性时执行准入测试。准入测试用于计算这些修改是否可行，如果不可行，[sched_setattr(2)](http://man7.org/linux/man-pages/man2/sched_setattr.2.html)将返回EBUSY错误。例如，总的CPU利用率应该小于或等于总的可用的CPU。由于每个线程可以在每个`Period`中最大运行`Runtime`时间，线程的CPU时间片使用率为`Runtime`除以`Period`。为了满足`SCHED_DEADLINE`的条件，**使用SCHED_DEADLINE策略的线程的优先级是系统中最高的**。当一个`SCHED_DEADLINE`线程运行时，该线程会抢占其他策略下调度的线程。对`SCHED_DEADLINE`策略调度的线程调用[fork(2)](http://man7.org/linux/man-pages/man2/fork.2.html)会返回EAGAIN错误(除非该线程设置了reset-on-fork标记)。当一个`SCHED_DEADLINE`线程调用了[sched_yield(2)](http://man7.org/linux/man-pages/man2/sched_yield.2.html)将会停止当前任务，并等待新的周期。

## SCHED_OTHER: Default Linux time-sharing scheduling(默认策略)

`SCHED_OTHER`只能在静态优先级为0时使用*(普通线程)*。`SCHED_OTHER`是标准的Linux分时调度策略(不需要实时机制)。如何从静态优先级为0的列表中选择运行的线程取决于列表中的`dynamic`优先级。`dynamic`优先级基于nice值，且在每次线程准备运行时增加。这种机制保证公平处理所有的`SCHED_OTHER`线程。在Linux内核源码中，`SCHED_OTHER`被称为`SCHED_NORMAL`。

### The nice value

nice值用于影响CPU调度器对进程的调度偏好。**适用于SCHED_OTHER和SCHED_BATCH调度处理**。可以通过[nice(2)](http://man7.org/linux/man-pages/man2/nice.2.html)，[setpriority(2)](http://man7.org/linux/man-pages/man2/setpriority.2.html)或[sched_setattr(2)](http://man7.org/linux/man-pages/man2/sched_setattr.2.html)修改nice值。

根据POSIX.1，nice值是一个单进程属性，即进程中的所有线程共享一个nice值。然而，在Linux中，nice值是一个单线程属性，相同进程中的不同线程可能使用不同的nice值。

nice值的取值范围根据UNIX系统的不同而不同。在现代Linux系统中，取值为-20(高优先级)到+19(低优先级)，而一些系统中的取值为-20..20。在一些非常早期的Linux 内核(Linux 2.0之前)中的取值为-infinity..15。

nice值对相应的`SCHED_OTHER` 进程的影响根据UNIX系统和Linux内核版本的不同而不同。2.6.23版本的Linux内核中引入了CFS调度，并采用了一种能根据nice的差值产生更显著影响的算法。在当前的实现下，两个进程的nice差值中，每单位的nice差值对CFS调度的影响因子为1.25 (参见[how-is-nice-working](https://unix.stackexchange.com/questions/89909/how-is-nice-working)，CFS根据vruntime进行CPU调度：`vruntime = 实际运行时间 * 1024 / 进程权重`，`进程权重`为`1.25 ^ nice_value`)。这种算法使得在有高优先级负载运行的情况下，只能给低nice值(+19)的负载提供很小的CPU；而为高nice值(-20)的负载提供其运行应用需要的绝大部分CPU(如音频应用)。

## SCHED_BATCH: Scheduling batch processes

从Linux 2.6.16开始，`SCHED_BATCH`可以用于静态优先级为0的线程。该策略类似`SCHED_OTHER`，并根据动态优先级(nice值)进行调度。区别是使用该策略时，调度器会假设线程是CPU密集型的，因此，该调度器会根据线程的唤醒行为施加调度惩罚，因此这种调度策略比较不受欢迎。

该策略比较适用于非交互且不期望降低nice值的负载，以及需要不因为交互而(在负载之间)造成额外抢占的调度策略的负载。*下面引用自*[*PHP-FPM on Linux, SCHED_BATCH or SCHED_OTHER?*](https://serverfault.com/questions/703180/php-fpm-on-linux-sched-batch-or-sched-other)*，更多参见[batch/idle priority scheduling, SCHED_BATCH](batch/idle priority scheduling, SCHED_BATCH)*

## SCHED_IDLE: Scheduling very low priority jobs

从Linux 2.6.23开始，`SCHED_IDLE`可以用于静态优先级为0的线程。nice值不会影响该策略。该策略用于运行非常低优先级的任务(低于nice值为+19的`SCHED_OTHER`或`SCHED_BATCH`策略)。

# 数据结构

优先级就是一个数值

- 优先级的范围是 0～99，对于实时进程

- 优先级的范围是 100～139，对于普通进程

- 数值越小，优先级越高

SCHED_FIFO、SCHED_RR、SCHED_DEADLINE 是实时进程的调度策略。

- SCHED_FIFO相同优先级的进程，遵循先来先得

- SCHED_RR轮流调度算法，采用时间片，相同优先级的任务当用完时间片会被放到队列尾部

- SCHED_DEADLINE按照任务的 deadline 进行调度，总是选择其 deadline 距离当前时间点最近的那个任务

SCHED_NORMAL、SCHED_BATCH、SCHED_IDLE

- SCHED_NORMAL普通的进程

- SCHED_BATCH后台进程，几乎不需要和前端进行交互

- SCHED_IDLE特别空闲的时候才跑的进程

完全公平调度算法

在 Linux 里面，实现了一个基于 CFS 的调度算法。CFS 全称 Completely Fair Scheduling，叫完全公平调度。

记录下进程的运行时间，CFS 会为每一个进程安排一个虚拟运行时间 vruntime。

> 虚拟运行时间 vruntime += 实际运行时间 delta_exec * NICE_0_LOAD/ 权重

虚拟运行时间相同，权重高的（优先级高）实际运行时间多，权重低的（优先级低）实际运行时间少，

vruntime 少的，需要给它补上，所以会优先运行这样的进程。

两个进程队列，普通进程队列和实时进程队列。

调度队列与调度实体

以完全公平调度算法为例

CFS 需要一个数据结构来对 vruntime 进行排序，找出最小的那个

- 这个数据结构在查询的时候，能够快速找到最小的，更新的时候也需要能够快速地调整排序

- 能够平衡查询和更新速度的是树，在这里使用的是红黑树

# 场景分析

实时进程将得到优先调用，实时进程根据实时优先级决定调度权值，普通分时进程则通过nice和counter值决定权值，nice越小，counter越大，被调度的概率越大，也就是曾经使用了cpu最少的进程将会得到优先调度。实时任务按照可抢占优先级调度算法进行，就绪态的实时任务可以立即抢占非实时任务。

1，SCHED_FIFO：First Input First Output，优先级范围1-99（最大）

SCHED_FIFO一旦占用cpu则一直运行。一直运行直到有更高优先级任务到达或自己放弃。

如果有相同优先级的实时进程（根据优先级计算的调度权值是一样的）已经准备好，FIFO时必须等待该进程主动放弃后才可以运行这个优先级相同的任务。而RR可以让每个任务都执行一段时间。

2，SCHED_RR：基于FIFO基础上的Round Robin时间片轮转，先入先出当采用SHCED_RR策略的进程的时间片用完，系统将重新分配时间片，并置于就绪队列尾。将进程放在队列尾保证了所有具有相同优先级的RR任务的调度公平。

3，SCHED_DEADLINE：该调度策略采用Global Earliest Deadline First(GEDF)算法，最早要到期的任务将被以最高优先级调度和分配CPU。根据此策略调度的任务可以抢占使用SCED FIFO、SCED RR或非实时调度的任何任务。参考<https://lwn.net/Articles/743740/和https://blog.csdn.net/zhuyong006/article/details/80980502>

<https://blog.csdn.net/xiaofeng_yan/article/details/48246101>

        调度参数（ns纳秒单位）：

        runtime：任务需要CPU执行的时长，要考虑最坏/最慢情况下的最大时长，要小于deadline

        period：任务执行的周期，即两次任务开始的时间间隔

        deadline：任务必须在该时长内执行完，不大于period，是从period开始时刻计算时长

所有任务都采用Linux分时调度策略时:

Linux采用CFS（Completely Fair Scheduler）完全公平调度算法实现分时进程的CPU资源分配，注意    ：非实时线程priority（100-139）由内核动态调控，不能人为设置，可以用修改nice值（-20~19 越低的值越高的优先级，这个在计算虚拟时间的时候放在分母上），从而间接影响优先级，PRI(new) = PRI(old) + nice

1）创建任务指定采用分时调度策略，并指定优先级nice值(-20~19)。

2）将根据每个任务的nice值确定在cpu上的执行时间(counter)。

3）如果没有等待资源，则将该任务加入到就绪队列中。

4）调度程序遍历就绪队列中的任务，通过对每个任务动态优先级的计算(counter+20-nice)结果，选择

计算结果最大的一个去运行，当这 个时间片用完后(counter减至0)或者主动放弃cpu时，该任务将被放在

就绪队列末尾(时间片用完)或等待队列(因等待资源而放弃cpu)中。

5）此时调度程序重复上面计算过程，转到第4步。

6）当调度程序发现所有就绪任务计算所得的权值都为不大于0时，重复第2步。

所有任务都采用FIFO的调度策略时：

1）创建进程时指定采用FIFO，并设置实时优先级rt_priority(1-99)。

2）如果没有等待资源，则将该任务加入到就绪队列中。

3）调度程序遍历就绪队列，根据实时优先级计算调度权值(1000+rt_priority),选择权值最高的任务使用cpu，该FIFO任务将一直占有cpu直到有优先级更高的任务就绪(即使优先级相同也不行)或者主动放弃(等待资源)。

4）调度程序发现有优先级更高的任务到达(高优先级任务可能被中断或定时器任务唤醒，再或被当前运行的任务唤醒，等等)，则调度程序立即在当前任务 堆栈中保存当前cpu寄存器的所有数据，重新从高优先级任务的堆栈中加载寄存器数据到cpu，此时高优先级的任务开始运行。重复第3步。

5）如果当前任务因等待资源而主动放弃cpu使用权，则该任务将从就绪队列中删除，加入等待队列，此时重复第3步。

所有任务都采用RR调度策略（轮转法调度）时：

1）创建任务时指定调度参数为RR，并设置任务的实时优先级和nice值(nice值将会转换为该任务的时间片的长度)。

2）如果没有等待资源，则将该任务加入到就绪队列中。

3）调度程序遍历就绪队列，根据实时优先级计算调度权值(1000+rt_priority),选择权值最高的任务使用cpu。,

4）如果就绪队列中的RR任务时间片为0（RR时间片是100ms），则会根据nice值设置该任务的时间片，同时将该任务放入就绪5队列的末尾。重复步骤3。

5）当前任务由于等待资源而主动退出cpu，则其加入等待队列中。重复步骤3。

所有任务都采用SCHED_DEADLINE调度策略时：

该调度策略在目前的 Linux 内核里面属于优先级最高的调度类，也就是说假设系统上面存在多个调度类，那么每次重新调度的时候，都是先从该调度类下面去找到一个可以投入运行的进程进行调度。归属于该调度类的进程其归一化优先级都是 -1，在该调度类内部不再区分多个优先级，统一归为 -1.

1）创建任务时指定调度参数为DEADLINE，设定runtime、period、deadline参数。

2）如果没有等待资源，则将该任务加入到就绪队列中。

3）调度程序遍历就绪列表，根据deadline到期时间，选择最短的任务执行

4）执行最长时间片由runtime指定，过时没有执行完也会被收回CPU资源

5）将任务加入到就绪队列中，重复步骤3

系统中既有分时调度，又有实时RR时间片轮转调度、FIFO先进先出调度和DEADLINE调度：

1）调度系统首先查看是否有DEADLINE任务将要到期，如有则会抢占CPU资源执行DEADLINE任务，即使当前CPU正在执行FIFO或RR任务。

2）RR调度和FIFO调度的进程属于实时进程，以分时调度的进程是非实时进程。

3）当实时进程准备就绪后，如果当前cpu正在运行非实时进程，则实时进程立即抢占非实时进程。

    普通进程完全捞不到CPU资源也不完全对，后来LINUX增加组调度策略后，有/proc/sys/kernel/sched_rt_period_us和/proc/sys/kernel/sched_rt_runtime_us两个参数，这两个参数表示的含义是：在sched_rt_peroid_us的周期内，所有的实时进程运行时间之和不得超过sched_rt_runtime_us。 默认值为1秒和0.95秒换句话说，在1秒的周期以内，所有实时进程运行时间之和不得超过0.95秒，剩下的0.5秒钟留给普通进程。

3）RR进程和FIFO进程都采用实时优先级做为调度的权值标准，RR是FIFO的一个延伸。FIFO时，如果两个进程的优先级一样，则这两个优先级一样的进程具体执行哪一个是由其在队列中的未知决定的，这样导致一些不公正性,如果将两个优先级一样的任务的调度策略都设为RR,则保证了这两个任务可以循环执行，保证了公平。

# 使用限制

特权进程

# 减少对交互的影响

## Resetting scheduling policy for child processes

每个线程都有一个reset-on-fork调度标识。当设置该标识后，使用[fork(2)](http://man7.org/linux/man-pages/man2/fork.2.html)创建的子进程不会继承特权调度策略。可以通过如下方式设置reset-on-fork：

- 在调用[sched_setscheduler(2)](http://man7.org/linux/man-pages/man2/sched_setscheduler.2.html)时，将SCHED_RESET_ON_FORK 标识作为`policy`参数，或

- 在调用[sched_setattr(2)](http://man7.org/linux/man-pages/man2/sched_setattr.2.html)时，将SCHED_FLAG_RESET_ON_FORK 设置为`attr.sched_flags`

注意上面两个函数的常量名称不一样。使用[sched_getscheduler(2)](http://man7.org/linux/man-pages/man2/sched_getscheduler.2.html)和[sched_getattr(2)](http://man7.org/linux/man-pages/man2/sched_getattr.2.html)获取reset-on-fork状态的用法与上面类似。

reset-on-fork特性用于媒体播放的应用，可以防止应用在创建多个子进程时规避RLIMIT_RTTIME设置的资源限制。

更精确地讲，如果设置了reset-on-fork，后续创建地子进程会遵循下面规则：

- 如果正在运行的线程使用了`SCHED_FIFO`或`SCHED_RR`调度策略，子进程地策略或被设置为`SCHED_OTHER`；

- 如果正在运行的进程的nice值为负值，子进程的nice值会被设置为0。

在设置reset-on-fork之后，只有线程拥有CAP_SYS_NICE的capability时才能重置reset-on-fork。使用[fork(2)](http://man7.org/linux/man-pages/man2/fork.2.html)创建的子进程会disable reset-on-fork标识。

## Limiting the CPU usage of real-time and deadline processes（限制实时进程或deadline进程）

SCHED_FIFO, SCHED_RR或SCHED_DEADLINE策略下调度的线程中的非阻塞无限循环处理可能会阻塞其他线程获取CPU。在Linux 2.6.25之前，阻止实时进程冻结系统的唯一方式是通过shell启动一个静态优先级更高的程序，如通过这种方式来停止实施程序，并释放CPU资源。

从Linux 2.6.25开始，引进了其他技术手段来处理实时(SCHED_FIFO，SCHED_RR)和deadline(SCHED_DEADLINE)进程。一种方式是通过RLIMIT_RTTIME 来限制实时进程可能使用到的CPU的上限。参见 [getrlimit(2)](http://man7.org/linux/man-pages/man2/getrlimit.2.html)获取更多信息。

从Linux 2.6.25开始，Linux提供了2个`/proc`文件来为非实时进程保留CPU时间。保留的CPU也可以为shell预留资源来停止正在允许的进程。两个文件中的值对应的单位为微秒：

- `/proc/sys/kernel/sched_rt_period_us`该文件中的值指定了等同于100% CPU的调度周期。取值范围为1到INT_MAX，即1微秒到35分钟。默认值为1000,000(1秒)。*定义了一个CPU使用周期，周期越短，可以越快开始下一个周期*

- `/proc/sys/kernel/sched_rt_runtime_us`该文件中的值指定了实时和deadline调度的进程可以使用的"period"。取值范围为-1到INT_MAX-1，设置为-1标识运行时间等同于周期，即没有给非实时进程预留任何CPU。默认值为950,000（0.95秒），表示给非实时或deadline调度策略保留5%的CPU。该参数需要结合`sched_rt_period_us`使用

## autogrouping

todo

# 参考

[linux线程调度策略 - charlieroro - 博客园](https://www.cnblogs.com/charlieroro/p/12133100.html#sched_deadline-sporadic-task-model-deadline-scheduling)

[linux进程调度算法:分时调度策略、FIFO调度策略、RR调度策略_共觞的博客-CSDN博客_分时调度策略](https://blog.csdn.net/qq_32811489/article/details/70768264)

[一步一步学linux操作系统: 12 进程调度一_调度策略与调度类介绍_墨1024-程序员宝宝 - 程序员宝宝](https://www.cxybb.com/article/leacock1991/107010226)

[一步一步学linux操作系统: 12 进程调度一_调度策略与调度类介绍_墨1024-程序员宝宝 - 程序员宝宝](https://www.cxybb.com/article/leacock1991/107010226)
