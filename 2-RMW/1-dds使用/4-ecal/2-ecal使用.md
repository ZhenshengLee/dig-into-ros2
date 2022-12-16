5.10.0

# 构建选项

ECAL_LAYER_ICEORYX，iceoryx作为本体传输层，默认关，保持关闭

# 传输模式

5.10.0开始，默认为本地模式local mode

![](https://tcs.teambition.net/storage/312h8b59418f909333b0279f185a79991edd?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk1OCwiaWF0IjoxNjcxMTkwMTU4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg4YjU5NDE4ZjkwOTMzM2IwMjc5ZjE4NWE3OTk5MWVkZCJ9.8_FPX8jNiu10yiFGqGVuRA0BaBB4APVMkEw39J-yO_A&download=image.png "")

## shm模式概述

Initialization phase:

- The publisher creates a memory file fitting to the size of the current payload.

- The publisher creates a named shared mutex and a named event used as synchronization primitives.

- The publisher informs all matching subscribers about the name of the memory file, the matching mutex and the update event.

- The subscriber starts observing the memory file for updates by listening to the update event.

Communication phase (default configuration):

- The publisher acquires the memory file access-mutex and opens the memory file.

- The publisher writes the payload content and some header information into the memory file.

- The publisher fires the update event, closes the memory file and unlocks the mutex.

- The subscribers are signaled by the update event and acquire the access-mutex.

- The subscribers open the memory file and copy the payload into their process memory.

- The subscribers close the memory file and release the access-mutex.

To support one to many publisher/subscriber connections the publisher creates in fact one named update event per connection.

## shm-multi-buffer

*Multi-buffering has been added in eCAL 5.10. Multi-buffered topics cannot be received by older eCAL versions. The feature is turned off by default.*

As described in the previous sections, eCAL uses one shared memory file per publisher. This can lead to performance reduction if

- the memory file is blocked by a subscriber that is copying (non-zero copy) or working on (zero-copy) the content

- the memory file is blocked because many connected subscribers acquire read access

- a combination of all

This issue can be relaxed by using multiple memory files per topic, so the publisher can already write the next payload, while some subscribers still keep the previous memory file locked. These files work like an SHM-file ring buffer.

多个流，多个shm-file，支持同时读和写！

## shm-zc模式

*Zero-copy has been added in eCAL 5.10. It is turned off by default. When turned on, old eCAL Version can still receive the data but will not use zero-copy.**

The “normal” eCAL Shared memory communication results in the payload being copied at least twice:

1. Into the SHM file by the publisher

1. From the SHM file the private memory of each subscriber

Usually there is no issue with that. Copying the payload from the memory file before executing the subscriber callback results in better decoupling, so the publisher can update the memory file with the next message while the subscriber is still processing the last one. Small messages will be transmitted in a few microseconds and will not benefit from zero-copy.

If it comes to very large messages (e.g. high resolution images) however, copying really matters and it can make sense to activate eCAL’s zero-copy mode. With zero-copy, the communication would look like this:

1. The publisher still has to copy the data into the memory file.

1. The subscriber executes its callback directly on the memory file. The memory file is blocked, while being used.

The memory file is blocked for new publications as long as the user’s callback is processing its content. It will also block other subscribers from reading the same SHM file.

Even though it is called zero-copy, only the subscribers are zero-copy. Publishers still have to copy the data into the memory file, as they have to also support other layers like UDP or TCP and therefore cannot directly work on the memory file.

shm模式比较成熟，就是memfile，zc模式作为shm模式的扩充

默认情况下，shm模式下，接受方收到消息，打开memfile，拷贝到用户区，关闭memfile，调用回调

回调不管结束与否，均不影响写入方。

在zc模式，则一个拷贝都没有，接收方打开memfile，不做拷贝（直接使用指针），调用回调，回调结束后，关闭memfile

由于回调期间memfile都不关闭，则会阻塞写入方

![](https://tcs.teambition.net/storage/312hdbc80bbaaaac38c585345eb0a5cb3483?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk1OCwiaWF0IjoxNjcxMTkwMTU4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhkYmM4MGJiYWFhYWMzOGM1ODUzNDVlYjBhNWNiMzQ4MyJ9.ej9bpZ9zqHYjrevXAuKEhoYPWSYT1DGNf7XRkjTqQDQ&download=image.png "")

[__https://github.com/continental/ecal/pull/358__](https://github.com/continental/ecal/pull/358)

![](https://tcs.teambition.net/storage/312h49086206c253bbeefd15e1bef33e61da?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk1OCwiaWF0IjoxNjcxMTkwMTU4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg0OTA4NjIwNmMyNTNiYmVlZmQxNWUxYmVmMzNlNjFkYSJ9.H_93FfEboFtrMj4QrkE6b3sA93aH2nZllOXmg52gRCE&download=image.png "")

## shm-zc-iceoryx模式

ecal最初没有自己的zc模式，而是借用了iceoryx的，因为iceoryx机制与ecal不同，所以不建议选用

![](https://tcs.teambition.net/storage/312h9b6b47f0c0f5e9c708679211b7224acb?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk1OCwiaWF0IjoxNjcxMTkwMTU4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg5YjZiNDdmMGMwZjVlOWM3MDg2NzkyMTFiNzIyNGFjYiJ9.lvYLG0Ij8wiIIPq4eCwd2wHogAP2G-BwGfOenM_Wwxk&download=image.png "")

[__https://github.com/continental/ecal/issues/326__](https://github.com/continental/ecal/issues/326)

## tcp模式

最重要的特点是保证数据不丢失，适用于大和重要数据场景

# ini配置

## 加载顺序

1. `$ECAL_DATA/ecal.ini`

1. `/etc/ecal/ecal.ini` (from `CMAKE_INSTALL_SYSCONFDIR`)

1. `/etc/ecal/ecal.ini` (fallback)

Official builds are configured to use `/etc`. If you are compiling eCAL yourself and don’t provide the SYSCONFDIR, CMake will usually use `/usr/local/etc/ecal/ecal.ini`.

deb安装的在/etc

### 配置文件

默认是网络模式cloud mode

```text
/etc/ecal/ecal.ini

[network]
network_enabled           = false
multicast_ttl             = 0
inproc_rec_enabled        = true
shm_rec_enabled           = true
tcp_rec_enabled           = true
udp_mc_rec_enabled        = true

[publisher]
use_inproc                = 0
use_shm                   = 2
use_tcp                   = 0
use_udp_mc                = 2

memfile_minsize           = 4096
memfile_reserve           = 50
memfile_ack_timeout       = 0
memfile_buffer_count      = 1
memfile_zero_copy         = 0

```

## 默认配置

![](https://tcs.teambition.net/storage/312h026a1e21bce1224db2250ea06ca06619?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk1OCwiaWF0IjoxNjcxMTkwMTU4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgwMjZhMWUyMWJjZTEyMjRkYjIyNTBlYTA2Y2EwNjYxOSJ9.4JpG2JeXOBm3Jpwsu6fV6k1GXap0d3qe8HWf5kRcvbo&download=image.png "")

# 测试程序

### 基准测试

```bash
# t1
ecal_sample_latency_rec
# t2
ecal_sample_latency_snd  -s 1024 -r 10000
ecal_sample_latency_snd  -s 1024 -r 10000 -z
```

![](https://tcs.teambition.net/storage/312h3d87259ff5246adfd415505f92a68a7d?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk1OCwiaWF0IjoxNjcxMTkwMTU4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgzZDg3MjU5ZmY1MjQ2YWRmZDQxNTUwNWY5MmE2OGE3ZCJ9.nIpTE3zKnUrpH5gcZfPG2Gt8q76vPSiCHxN6smNt-zI&download=image.png "")

# 工具app

ecal_sys_gui，启动停止监控程序的工具

ecal_mon_gui，监控程序的工具

recorder, player是录制播放工具

# 使能zc方法

## ini

更改配置，memfile_zero_copy         = 1

## 代码

[__https://github.com/continental/ecal/pull/358/files#diff-a3fd43ad698146c33c14a04041bde77ba4d17b5dfbf6dcfec2f703adc30f8286__](https://github.com/continental/ecal/pull/358/files#diff-a3fd43ad698146c33c14a04041bde77ba4d17b5dfbf6dcfec2f703adc30f8286)

# 限制

## 有限QoS

![](https://tcs.teambition.net/storage/312h0faef323a1ba9aa38128d8b45ca31703?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDk1OCwiaWF0IjoxNjcxMTkwMTU4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgwZmFlZjMyM2ExYmE5YWEzODEyOGQ4YjQ1Y2EzMTcwMyJ9.vLvu9Jr2mu_J5lI_ATg7k6HZEP5rCudpeKqf3qULeOA&download=image.png "")

目前作为单机中间件来讲，不需要qos

[__https://github.com/continental/ecal/issues/245__](https://github.com/continental/ecal/issues/245)

# 参考

[Welcome to eCAL’s documentation! — eCAL Documentation  documentation](https://continental.github.io/ecal/)

[ecal.ini — eCAL Documentation  documentation](https://continental.github.io/ecal/configuration/options.html)
