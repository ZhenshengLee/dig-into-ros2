# iceoryx-dds介绍

bosch出品于2020年

## 参考

<https://www.eclipse.org/community/eclipse_newsletter/2019/december/4.php>

<https://github.com/ros2/rmw_iceoryx>

<https://github.com/eclipse-iceoryx/iceoryx>

## 介绍

iceoryx is an inter-process-communication (IPC) middleware for various operating systems (currently we support Linux, MacOS and QNX).

是一个IPC中间件, 起源于汽车行业软件, 意味着其不支持网络通信.

号称true-zero-copy

![](https://tcs.teambition.net/storage/312666492db1a8dd1f0c369d4815b8cf8561?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDgzMCwiaWF0IjoxNjcxMTkwMDMwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY2NjQ5MmRiMWE4ZGQxZjBjMzY5ZDQ4MTViOGNmODU2MSJ9.4_zOW3fw0htz3q_9ceRlUUeeU_jx7mGHLD_h-UBzqNI&download=image.png "")

In addition, we have already identified the potential for synergies within the Eclipse family. For example, combining Eclipse Cyclone DDS and iceoryx creates open and powerful communications middleware for IPC and network communications.

目前开发团队正在把iceroyx和cyclone-dds结合起来, 发挥优势, 获得更好的IPC和网络通信能力

## 对ROS2的支持

目前功能不全

只包含topic传输接口

lgsvl不支持iceroyx通信, 所以也不能做仿真

## 性能

![](https://tcs.teambition.net/storage/312624bc0eb96b037b1a63be6445bc0ba3a2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDgzMCwiaWF0IjoxNjcxMTkwMDMwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjYyNGJjMGViOTZiMDM3YjFhNjNiZTY0NDViYzBiYTNhMiJ9.AEaL0e7IMZ4zuQNZohWFgUiVIP7lpXVuciGlX_2wO7M&download=image.png "")

在4M 以下数据, 传输时延稳定在us级别

## 安装配置

```bash
sudo apt install libacl1-dev
# 18.04
sudo apt install maven openjdk-11-jdk-headless
# 20.04
sudo apt install maven openjdk-14-jdk-headless
cd ~/Downloads/cpptoml-0.1.1
mkdir build
cd ./build
cmake ..
make
sudo make install
cd ~/Downloads/iceoryx-master
# !!! tip To build all iceoryx components add -DBUILD_ALL to the CMake command
cmake -Bbuild -Hiceoryx_meta -DBUILD_ALL=1
# cmake -Bbuild -Hiceoryx_meta -DBUILD_ALL=1 -DCMAKE_INSTALL_PREFIX=/opt/iceoryx/iceoryx
cmake --build build  -j10
sudo cmake --build build --target install
# /usr/local/lib/libiceoryx_introspection.a

# 卸载
cd /usr/local/lib
sudo rm ./libiceoryx_*
cd /usr/local/include
sudo rm -rf iceoryx_*
cd /usr/local/lib/cmake
sudo rm -rf iceoryx_*
cd /usr/local/bin
sudo rm iox-* iceperf-*
```

rmw还没更新

iceoryx不支持dashing, 主要支持foxy

## SHM支持

有

## 安全支持

没有

## 网络支持

通过iceoryx_bridge？

通过dds_gataway？

## 测试

```bash
s@zs-3630:/usr/local/bin$ ./iceperf-laurel

******   MESSAGE QUEUE    ********
waiting for follower
Measurement for: 1 kB, 2 kB, 4 kB, 8 kB, 16 kB, 32 kB, 64 kB, 128 kB, 256 kB, 512 kB, 1024 kB, 2048 kB, 4096 kB,

#### Measurement Result ####
10000 round trips for each payload.

| Payload Size [kB] | Average Latency [µs] |
|------------------:|---------------------:|
|                 1 |                  4.4 |
|                 2 |                  2.4 |
|                 4 |                  2.8 |
|                 8 |                    4 |
|                16 |                  5.7 |
|                32 |                  9.5 |
|                64 |                   17 |
|               128 |                   33 |
|               256 |                   63 |
|               512 |              1.2e+02 |
|              1024 |              2.4e+02 |
|              2048 |              4.7e+02 |
|              4096 |              9.4e+02 |

Finished!

****** UNIX DOMAIN SOCKET ********
waiting for follower
Measurement for: 1 kB, 2 kB, 4 kB, 8 kB, 16 kB, 32 kB, 64 kB, 128 kB, 256 kB, 512 kB, 1024 kB, 2048 kB, 4096 kB,

#### Measurement Result ####
10000 round trips for each payload.

| Payload Size [kB] | Average Latency [µs] |
|------------------:|---------------------:|
|                 1 |                  8.6 |
|                 2 |                  3.6 |
|                 4 |                  3.9 |
|                 8 |                  4.7 |
|                16 |                  7.5 |
|                32 |                   13 |
|                64 |                   25 |
|               128 |                   46 |
|               256 |                   89 |
|               512 |              1.8e+02 |
|              1024 |              3.5e+02 |
|              2048 |              7.1e+02 |
|              4096 |              1.4e+03 |

Finished!
2021-06-30 21:33:15.522 [ Debug ]: Application registered management segment 0x7f61ac5d2000 with size 60126248 to id 1
2021-06-30 21:33:15.523 [ Debug ]: Application registered payload data segment 0x7f61a373c000 with size 149264720 to id 2

******      ICEORYX       ********
Waiting for: subscription, subscriber [ success ]
Measurement for: 1 kB, 2 kB, 4 kB, 8 kB, 16 kB, 32 kB, 64 kB, 128 kB, 256 kB, 512 kB, 1024 kB, 2048 kB, 4096 kB,
Waiting for: unsubscribe  [ finished ]

#### Measurement Result ####
10000 round trips for each payload.

| Payload Size [kB] | Average Latency [µs] |
|------------------:|---------------------:|
|                 1 |                 0.82 |
|                 2 |                 0.82 |
|                 4 |                 0.81 |
|                 8 |                 0.82 |
|                16 |                 0.81 |
|                32 |                 0.78 |
|                64 |                 0.78 |
|               128 |                 0.78 |
|               256 |                 0.81 |
|               512 |                 0.81 |
|              1024 |                 0.81 |
|              2048 |                 0.79 |
|              4096 |                 0.79 |

Finished!

******   ICEORYX C API    ********
Waiting for: subscription, subscriber [ success ]
Measurement for: 1 kB, 2 kB, 4 kB, 8 kB, 16 kB, 32 kB, 64 kB, 128 kB, 256 kB, 512 kB, 1024 kB, 2048 kB, 4096 kB,
Waiting for: unsubscribe  [ finished ]

#### Measurement Result ####
10000 round trips for each payload.

| Payload Size [kB] | Average Latency [µs] |
|------------------:|---------------------:|
|                 1 |                  1.2 |
|                 2 |                  1.3 |
|                 4 |                  1.2 |
|                 8 |                  0.9 |
|                16 |                 0.83 |
|                32 |                 0.82 |
|                64 |                 0.82 |
|               128 |                 0.82 |
|               256 |                 0.83 |
|               512 |                 0.82 |
|              1024 |                 0.82 |
|              2048 |                 0.84 |
|              4096 |                 0.83 |

Finished!
2021-06-30 21:33:15.974 [Verbose]: RouDi cleaned up resources of laurel. Shutting down gracefully.

```

## 策略

进行性能对比测试

```text
ros2 daemon stop
```
