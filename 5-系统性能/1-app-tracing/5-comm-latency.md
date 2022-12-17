![](https://tcs.teambition.net/storage/312hca179788dab521143fe539846aa79076?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhjYTE3OTc4OGRhYjUyMTE0M2ZlNTM5ODQ2YWE3OTA3NiJ9.BwQNXlrggJxKDQiN4Mc_HX-MO7cRRiGxoypgE5htO3M&download=image.png "")

# 例子

![](https://tcs.teambition.net/storage/312h70675c798c7645a24785da7c0f0b3be6?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg3MDY3NWM3OThjNzY0NWEyNDc4NWRhN2MwZjBiM2JlNiJ9.npeQKHVoMnEljb6Cviwo8FaqvzZuzckO09LN4BUtkqg&download=architecture.png "")

初次运行，打开trace，rqt_graph记录，检查基础功能，生成架构文件

topic1:11hz,  0.09s, 执行0.01秒

topic2:7hz,  0.14s，执行0.01s

topic3:  topic1, 执行0.01s

topic4: topic3, 执行0.01s

topic5: topic2, 执行0.01s

topic6: 3hz, 0.33s, 执行0.01s

回调执行时间全部是0.01s，10ms

# 测试目标

测试topic1的执行时间，包括ros2和dds执行时间

# 启动例子

```bash
ros2 launch e2e_demo demo1.launch.py
```

# 传统方法

暂时没搞定

# e2e方法

e2e支持dds-latency和comm-latency

## 构建arch

略

## 运行程序

先确认lttng trace是否正常

```bash
babeltrace ./e2e_demo1/ | grep -e rclcpp_publish -e rclcpp_subscribe
babeltrace ./e2e_demo1/ | grep -e rclcpp_timer_added

babeltrace ./v4l2_trace/ | grep -e rclcpp_publish
babeltrace ./v4l2_trace/ | grep -e rclcpp_subscribe
# 检查时间有效性
```

## 分析脚本

app.comms

class CommCollection(collections.abc.Iterable):

![](https://tcs.teambition.net/storage/312h06216cbaac541171687f682e4b72ff43?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgwNjIxNmNiYWFjNTQxMTcxNjg3ZjY4MmU0YjcyZmY0MyJ9.hao6oVDzQFuvrX-vAhr_hYOs1jLSqXPbWofFYnSby54&download=image.png "")

18-查看coms的清单，由话题的数组组成

comm下有一个child为dds_topic

## 结果

![](https://tcs.teambition.net/storage/312hbe5f0a5a11dcdd7eb8137b2ef5a9d2f9?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhiZTVmMGE1YTExZGNkZDdlYjgxMzdiMmVmNWE5ZDJmOSJ9.XbqjGN6hp9ZwD_Xge9PQ72f9GbCdNclnEpc6u0j5ysw&download=image.png "")

![](https://tcs.teambition.net/storage/312hba012910bb98a1f0f39698bc718cf26a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhiYTAxMjkxMGJiOThhMWYwZjM5Njk4YmM3MThjZjI2YSJ9.OSgI0glV2mzT1HiQl4odPOZp2hIEkChEMCdM2Nv8DuI&download=image.png "")

# 内部机制

import_trace

```python
comm_instances = self._get_comm_instances(events, self.comms)
        if len(comm_instances) > 0:
            comm_instances = self._filter.remove(comm_instances, 'timestamp')
            assert len(comm_instances) > 0, 'all instance are removed'
        self.comm_instances = comm_instances
        self._import_comm_instances(comm_instances)

```

_get_comm_instances

```python
    def _get_comm_instances(self, events, comms):
        publish_instances = self.get_publish_instances(events)
        subscribe_instances = self.get_subscribe_instances(events)

        comm_instances = [self._get_specific_comm_instances(comm,
                                                  publish_instances,
                                                  subscribe_instances) \
                          for comm in comms]
        if len(comm_instances) > 0:
            comm_instances = pd.concat(comm_instances)
        return comm_instances

```

get_publish_instances(event)

![](https://tcs.teambition.net/storage/312i9843a95fe9fd8e216ae29e66ccecbce7?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmk5ODQzYTk1ZmU5ZmQ4ZTIxNmFlMjllNjZjY2VjYmNlNyJ9.6otcFdQdXyj-acRNeL7fumBC0P-TlbpGXRNiEB99o9A&download=image.png "")

get_subscribe_instances

![](https://tcs.teambition.net/storage/312ic049c9ac8cd38887abbf3e470843572c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmljMDQ5YzlhYzhjZDM4ODg3YWJiZjNlNDcwODQzNTcyYyJ9.AcC7RgEBhhHg4Z676THshODFCyeSGXHgTD1N5BnYw_0&download=image.png "")

_import_comm_instances

![](https://tcs.teambition.net/storage/312i9f87954d01cfc4bfba49fa858fc23a3f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmk5Zjg3OTU0ZDAxY2ZjNGJmYmE0OWZhODU4ZmMyM2EzZiJ9.3LRYwZ1L_rH1AQAENKRBFnnYSRqtX_f1WouWQnrXgAo&download=image.png "")

整体通信时间：duration是timestamp-source_stamp

dds通信时间：commun_latency=received_stamp-source_stamp

comm.get_stats()

dds.get_stats()

# Tp支持

rclcpp_subscriber

# rmw支持

rmw_node.cpp支持

create_subscription，增加listener，向底层传递listener

![](https://tcs.teambition.net/storage/312ifa3fa2f1140bd8b87c5dd97aa2c505ee?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmlmYTNmYTJmMTE0MGJkOGI4N2M1ZGQ5N2FhMmM1MDVlZSJ9.iB1gXPdxowFkEV1Rjtpi76384OSpXmKgDXm7d5DEbx8&download=image.png "")

![](https://tcs.teambition.net/storage/312i5629e0f48a7583eddc6182e60be72e44?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmk1NjI5ZTBmNDhhNzU4M2VkZGM2MTgyZTYwYmU3MmU0NCJ9.-sMlbMP5FwPYkIYmgiBrIgzp0NlvvHtxiinuRwgSSJQ&download=image.png "")

实现从dds获取reception_timestamp

![](https://tcs.teambition.net/storage/312ifacd1322b8d2a04c3bc03c6e16d65673?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmlmYWNkMTMyMmI4ZDJhMDRjM2JjMDNjNmUxNmQ2NTY3MyJ9.qlgRgFb1iRlSkmFDaNFOcLrJ7C6rAWZTSE55ae1Pdkc&download=image.png "")

## rmw_cyclonedds支持

message_info有source_stamp 和received_stamp

[__https://github.com/ros2/rmw_cyclonedds/pull/163#issuecomment-618826057__](https://github.com/ros2/rmw_cyclonedds/pull/163#issuecomment-618826057)

缺少received_stamp

## rmw_fastrtps_cpp支持

[__https://github.com/ros2/rmw_fastrtps/pull/369__](https://github.com/ros2/rmw_fastrtps/pull/369)

message_info缺少source_stamp

回退fastdds后可用，不知道时间准不准！

## rmw_ecal_dynamic支持

message_info缺少source_stamp和received_stamp

# 验证程序

开发subscriber验证测试的准确性

## rmw_cyclonedds

验证程序在rmw_cyclonedds下准确性较强，

![](https://tcs.teambition.net/storage/312i8c2813e9ede425511a03cec52fa00d17?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmk4YzI4MTNlOWVkZTQyNTUxMWEwM2NlYzUyZmEwMGQxNyJ9.y6MiH-yaSJj2v9XtscKlSVGezXeN6xeR5Gp780kZSk4&download=image.png "")

comm计算时间为sub的timestamp-source_stamp

![](https://tcs.teambition.net/storage/312i6e0b2631b82eb04a52cd58fab82f9455?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmk2ZTBiMjYzMWI4MmViMDRhNTJjZDU4ZmFiODJmOTQ1NSJ9.1k6bNtSm1WxHikk5AbEojnMI-LMxLKFqWV_xgqcGS9I&download=image.png "")

此时间为sub的timestamp-pub_time_stamp

## rmw_fastrtps_cpp

![](https://tcs.teambition.net/storage/312i260c891f3c73ffd338091b755a9af090?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmkyNjBjODkxZjNjNzNmZmQzMzgwOTFiNzU1YTlhZjA5MCJ9.g1QZx4uIjWZv0Z2U6B1ZSzJl93BKfjXAVN4s1Cq66-Q&download=image.png "")

![](https://tcs.teambition.net/storage/312i2c7f82c9e80ca1be0e8d429953057d1a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxMCwiaWF0IjoxNjcxMjQ2NjEwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmkyYzdmODJjOWU4MGNhMWJlMGU4ZDQyOTk1MzA1N2QxYSJ9.p4OnWd-xk_SjugqN7BVsSjYLRDg6vUm8pek5RRur8v0&download=image.png "")

时间不准

# 改进机制

通信时间改回原来的机制，sub_timestamp-pub_time_stamp，需要修改rclcpp的publish-tracepoint
