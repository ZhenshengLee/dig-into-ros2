不用arch文件分析e2e

# 为什么no-arch

arch无法描述一些复杂场景

此时需要直接使用脚本分析即可

# 方法

1. Analyze the trace datamkdir -p ~/tracelogros2 trace-analysis process /path/to/trace/directory > tracelog.txtbabeltrace /path/to/trace/directory | grep callback_start > callback_start.txtbabeltrace /path/to/trace/directory | grep callback_end > callback_end.txt// Write the path of tracelog into "/path/to/trace/directory"cd ~/autoware_perf_galactic/tracetools_analysis/tracetools_analysis/analysis/galacticjupyter notebook// Click the e2elatency.ipynb

1. Enter the jupyter interface// Write the path of ~/tracelog into path_prefix = " "// Use rqt and cb function to find the path of end-to-end// Write the node name into node1, node2...// Write the topic name that the node subscribes into topic1, topic2...// Run the code

# 参考

[https://github.com/azu-lab/ROS2-E2E-Evaluation/tree/main/autoware_perf_galactic](https://github.com/azu-lab/ROS2-E2E-Evaluation/tree/main/autoware_perf_galactic)

# 理论

## 选择关键路径

![](https://tcs.teambition.net/storage/312h9d1b634f04da01cb32ffc997b3d70315?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQwNCwiaWF0IjoxNjcxMjQ2NjA0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg5ZDFiNjM0ZjA0ZGEwMWNiMzJmZmM5OTdiM2Q3MDMxNSJ9.kbIyRzg8_wEhNCY5NbiR7abZJjWXgvIsbn38XMUUBKA&download=image.png "")

计算指定节点的回调时延，ros2会将所有回调和节点信息记录，这些信息被分析，汇聚到ros2_tracing python软件的data-model

## 端到端时延的概念

![](https://tcs.teambition.net/storage/312h840961b7664c4756e626fcb89f8839e2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQwNCwiaWF0IjoxNjcxMjQ2NjA0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg4NDA5NjFiNzY2NGM0NzU2ZTYyNmZjYjg5Zjg4MzllMiJ9.TXfk4nOqw2_0fkGMiKwp7xBj0B90nYiVKwAUKjqMy3A&download=image.png "")

端到端时延，由node-execution-time和pub/sub-comm-latency累加而成，我们可以通过选择节点来计算

node-execution-time包括callback-duration 和inter-callback-latency

![](https://tcs.teambition.net/storage/312hdcdff1894ee03fc1a23b219adca2acd5?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQwNCwiaWF0IjoxNjcxMjQ2NjA0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhkY2RmZjE4OTRlZTAzZmMxYTIzYjIxOWFkY2EyYWNkNSJ9.MVOf2-yQ-dxlMFpWFJQNHKLWXluVyyRQ4Ei1MNhOx7g&download=image.png "")

## 时延计算

![](https://tcs.teambition.net/storage/312h9aaee8ce4d70c73267f7c005aea82304?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQwNCwiaWF0IjoxNjcxMjQ2NjA0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg5YWFlZThjZTRkNzBjNzMyNjdmN2MwMDVhZWE4MjMwNCJ9._YrKKZ0Q6m5OhZxyQvq-3drG1YKEZ1p7Wf6PBPq2kic&download=image.png "")

复杂场景下，节点一般都不是单输入单输出的，而且存在丢包现象，导致trace不能跟踪单独的一个消息。

由于节点间消息不是一对一关系，所以不能将node latency简单相加。所以e2e延迟通过将node latency的离散分布图的概率模型进行卷积

离散概率分布的卷积积分

卷积积分是计算两个独立随机变量和的一个通用方法

![](https://tcs.teambition.net/storage/312h96bffcfac7639f272cf3001580f35fbe?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQwNCwiaWF0IjoxNjcxMjQ2NjA0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg5NmJmZmNmYWM3NjM5ZjI3MmNmMzAwMTU4MGYzNWZiZSJ9.8W_gNkOqLP9dnyL_hAinEzyyD-sSwBjmK76V03P_cNM&download=image.png "")
