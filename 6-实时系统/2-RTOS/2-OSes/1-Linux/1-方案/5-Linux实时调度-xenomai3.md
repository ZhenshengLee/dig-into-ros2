问:

我可以在我的开发板PREEMPT_RT直接在内核环境中执行POSIX应用,　使用Xenomai3 这是什么原因它?答:

假设你的应用程序已经完全是POSIX,而且性能也满足，则,而且也没有理由去使用它Xenomai3.但是，你可能会考虑基于以下两点Xenomai3.

- **你想移植遗留的嵌入式应用到Linux上而不用改动API（*移植性更好）**.　比方说,你不想在POSIX接口上重写API,　这时候Xenomai就能发挥它的用场.由于它通过一个共通的实时性来支持多个编程接口,包含传统的RTOS API, Xenomai3也将在基于PREEMPT_RT的系统上支持这些API.

- **你的目标板的性能不足,或/而且你想要你的实时任务给系统追加最小的负载（对处理器/微控制器性能要求较低）**. 这就是双内核机制优于原生的抢占系统的一个地方. 后者的情况下,全部的Linux系统必须运行内部代码(如优先级继承,中断线程化)来防止实时处理被延迟, 可是在双内核系统里, 并没有这样的必要,由于实时内核是差别于Linux内核独立运行的,故而,通常的Linux动作并不会对实时动作有影响,它甚至都不须要知道实时内核.

总而言之, 对这类问题并没有一个统一的答案:它实际上依赖于你对性能的要求(**实时性上：Xenomai优于PREEMPT_RT**)和你的开发板的性能. 这是一个case-by-case的东西. 仅仅是说"我们能够在最坏情况下达到X微秒"却不指明板子的性能并没有什么意义.

# ROS外挂架构

![](https://tcs.teambition.net/storage/312g199d19125dc6de3906808bdb4960fef0?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMTM0NCwiaWF0IjoxNjcxNDE2NTQ0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmcxOTlkMTkxMjVkYzZkZTM5MDY4MDhiZGI0OTYwZmVmMCJ9.tGOjoFj040dZjs6GUKMWLSstNjvDf3aD-JFm6oSgkH0&download=image.png "")

# ROS2外挂架构

![](https://tcs.teambition.net/storage/312gdba3f2f44c36b692f24b2b128dfd6d73?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMTM0NCwiaWF0IjoxNjcxNDE2NTQ0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdkYmEzZjJmNDRjMzZiNjkyZjI0YjJiMTI4ZGZkNmQ3MyJ9.C_r6V2Jms6QKTUa9TKcX4AdQ8ihMgIbXDZCde_stRKs&download=image.png "")

# ROS原生架构

目前没有

RTI有connext在Xen上的发行版，也就是说完全跑在

# BROS实现

BROS支持ROS2的外挂架构，不过该架构没有推广开来，需要开发人员自己实现。

# 参考

[linux实时化改造：Preempt-rt 和 Xenomai - CodeAntenna](https://codeantenna.com/a/YezacSDnUv)
