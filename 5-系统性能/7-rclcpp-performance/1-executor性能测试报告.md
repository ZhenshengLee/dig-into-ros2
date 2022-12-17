# 参考

<https://gitlab.com/ros-tracing/ros2_perf_analysis/-/blob/master/executor_profiling/post.md>

# 基准测试

需要backport static executor 的功能到foxy分支

## dell-3630-foxy测试结果

| node | executor                                                                                                                                  | ROS2-foxydefault executor | ROS2-foxystatic-single-threaded-executor | ROS2-foxyevents-executor | ROS2-Galacticwait-set based executor |
| ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------- | ---------------------------------------- | ------------------------ | ------------------------------------ |
| Struct16默认工况                                                                                                                                     | 16.4552%                  | -                                        | 10.0932%                 | -                                    |
| Array1k默认工况                                                                                                                                      | 18.7503%                  | -                                        | 10.0958%                 | -                                    |
| pc1m点云工况depth=1                                                                                                                                  | 0.7047%                   | -                                        | 0.6551%                  | -                                    |
| pc1m点云工况depth=10                                                                                                                                 | 0.7551%                   | -                                        | 0.6531%                  | -                                    |
| 配置：传输层使用默认fast-rtpscpu占用率取平均值，去除第一个异常值dell-3630工作站默认工况：消息发送频率=1000，qos为keepall点云工况：消息发送频率=10，qos为keeplast，depth=1, 10为了突出对比cpu占用率，1个发送节点，10个订阅节点 |                           |                                          |                          |                                      |

测试结果表明

- 在高频率，小数据的情况下效果明显，低频率，订阅量较少传输情况下，executor开销不是性能瓶颈。

- 在高频率，小数据的情况下效果明显，使用events_executor最高可以减少约40%的cpu占用率

- 在keeplast, depth=1的qos配置下，不同executor性能差异不大

使用static-single_threaded-executor最高可以减少约20%的cpu占用率

# 系统测试

## dell-3630-foxy测试结果

| node | executor                                | ROS1default | ROS2-Foxydefault executor | ROS2-Foxystatic-single-threaded-executor |
| ---------------------------------------------- | ----------- | ------------------------- | ---------------------------------------- |
| rslidar_sdk                                    |             | 10.5%                     | 10%                                      |
| 配置：传输层使用默认dds，cycloneddscpu占用率取平均值dell-3630工作站 |             |                           |                                          |

# 附录：官方测试结果

尽管static的设计者宣称有明显的性能改良，但是复现结果的环境很苛刻

## static

![](https://tcs.teambition.net/storage/31270dbe0174e7f1685bb914262c5683ea66?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg2MTMwMiwiaWF0IjoxNjcxMjU2NTAyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcwZGJlMDE3NGU3ZjE2ODViYjkxNDI2MmM1NjgzZWE2NiJ9.cguv01KCDkBMW4e39r5BG8oIxLL61NxZVaaL3GGJqRY&download=image.png "")

实际测试中，并未发现不同executor之间明显的性能差异

## events

![](https://tcs.teambition.net/storage/3127207133f577f28f32282f4bbbecccc883?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg2MTMwMiwiaWF0IjoxNjcxMjU2NTAyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcyMDcxMzNmNTc3ZjI4ZjMyMjgyZjRiYmJlY2NjYzg4MyJ9.ncRRGCJR9NN52HhmaDiM3PJ5ena5t4sX68TO2Vu3Fpo&download=image.png "")

![](https://tcs.teambition.net/storage/312792210a7ecb75e841b052ada081a906ce?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg2MTMwMiwiaWF0IjoxNjcxMjU2NTAyLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc5MjIxMGE3ZWNiNzVlODQxYjA1MmFkYTA4MWE5MDZjZSJ9.jBXlHqQMaeaPW9OqGAH6Oe32hXV1Y2oOGMGpNBNpDow&download=image.png "")
