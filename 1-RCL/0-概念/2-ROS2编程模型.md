# ROS1编程模型

## 架构图

![](https://tcs.teambition.net/storage/31272b0a4d776b207213ddac29a3a23a93c7?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjM2NCwiaWF0IjoxNjcxMDE3NTY0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcyYjBhNGQ3NzZiMjA3MjEzZGRhYzI5YTNhMjNhOTNjNyJ9.hhV1jkFFEoT9-CN2on7zRJzAHRObnE6c-mlkHoBu_7U&download=image.png "")

## 通信图

![](https://tcs.teambition.net/storage/312g4c1a9e56afc7f3df0f1d7b1175d5b5e5?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjM2NCwiaWF0IjoxNjcxMDE3NTY0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmc0YzFhOWU1NmFmYzdmM2RmMGYxZDdiMTE3NWQ1YjVlNSJ9.qRNXmY6E5io2Zsw0YZZmnW7FOIODcj0lZt04nkU_cro&download=image.png "")

# ROS2解耦

executor进一步

- 上层：机器人领域业务抽象（ROS Graph）

  - 各种业务的抽象（驱动，算法，交互，界面，仿真）

- 中层：事件处理机制（executor）

  - 逻辑与进程分离

- 底层：数据传输机制（dds）

  - IO分层，上面为应用层，下面为传输层

  - 逻辑与数据分离

## 架构图

![](https://tcs.teambition.net/storage/3127af210819549bb0e806b84d68a29083e8?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjM2NCwiaWF0IjoxNjcxMDE3NTY0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjdhZjIxMDgxOTU0OWJiMGU4MDZiODRkNjhhMjkwODNlOCJ9.caqIAN0dq9T6FrH-mgTucvTcdoYA91o5w9ke9sgwne4&download=image.png "")

## 通信图

![](https://tcs.teambition.net/storage/31274623d3d596e7461fec9478153a26679a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjM2NCwiaWF0IjoxNjcxMDE3NTY0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc0NjIzZDNkNTk2ZTc0NjFmZWM5NDc4MTUzYTI2Njc5YSJ9.p-z-WSAxS93aGfM9_pCsPqgGb4a1zEDZ8pajSG3RVUU&download=image.png "")

## 时序图

![](https://tcs.teambition.net/storage/312gf409888e0eafb6bcbc11b6d102a6e212?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjM2NCwiaWF0IjoxNjcxMDE3NTY0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdmNDA5ODg4ZTBlYWZiNmJjYmMxMWI2ZDEwMmE2ZTIxMiJ9.OcgUtRWqoMdfF5JZbKbxRE9iRjJMH_FwiWWfIjG7Jj8&download=image.png "")

## 部署图

![](https://tcs.teambition.net/storage/312gae90e3a43245f619c4e9d314bb0502d3?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjM2NCwiaWF0IjoxNjcxMDE3NTY0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdhZTkwZTNhNDMyNDVmNjE5YzRlOWQzMTRiYjA1MDJkMyJ9.fqbN7sWEdJvYHSk1C9b0P-sqVyP2l9Us_115WIm_e_E&download=image.png "")

## 代码

![](https://tcs.teambition.net/storage/312g4b3e86bbc59bc8a2586357f21af85afb?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjM2NCwiaWF0IjoxNjcxMDE3NTY0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmc0YjNlODZiYmM1OWJjOGEyNTg2MzU3ZjIxYWY4NWFmYiJ9.YRKgAFvqSlRTJKOtPYHOhVMXLsWp_RD-d6FenirW8m4&download=image.png "")

This application

model, seen in Figure 1.1, organizes executable code into callbacks. Callbacks are then

organised into nodes (ostensibly object-oriented programming (OOP) classes) which share

access to common resources. Finally, nodes are organised into executable entities, forming a

set of independent executables that (when executed concurrently) represents the complete

application run-time.

- 数据缓存在dds参与者的缓存区中

- wait的时候找dds要wait信号

- wait成功后找node要回调函数

- exe回调的时候找dds要serialized 数据

# 与DDS关系

![](https://tcs.teambition.net/storage/312geb9a2e2136951a8a15202c9041db58e2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjM2NCwiaWF0IjoxNjcxMDE3NTY0LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdlYjlhMmUyMTM2OTUxYThhMTUyMDJjOTA0MWRiNThlMiJ9.Lpz9IWwhWvnYEecE_DSbkOIC2qUGwV0UaDkgMz9CzEw&download=image.png "")

# RCL与RCLCPP

RCL包含三个部分：

- RCL接口聚焦ROS到DDS的概念映射

- 不含线程模型，主要是创建数据结构

- 维护对DDS的操作

RCLCPP包含三个部分：

- 语言客户端，cpp实现

- executor线程模型

- 基于RCL创建更复杂的组件，如类型系统，串行化，component，qos

- 基于RCL和类型系统，QOS生成更复杂灵活的应用程序

上述说明，其实rclcpp的核心是线程模型，当然也为其他的机制提供灵活性，比如generic publisher就可以直接发送串行化的信息

[2-dds中的线程优先级](https://thoughts.teambition.com/share/6268e5650c85220041ebb168#title=2-dds%E4%B8%AD%E7%9A%84%E7%BA%BF%E7%A8%8B%E4%BC%98%E5%85%88%E7%BA%A7)
