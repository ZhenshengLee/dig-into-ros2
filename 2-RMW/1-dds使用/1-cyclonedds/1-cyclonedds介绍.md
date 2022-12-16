# 使用cyclone-dds

cyclonedds是ros2默认的dds, adlink是autoware基金会的高级成员, 所以促成了cyclonedds对fastrtps的替代

## 参考

<http://www.robotandchisel.com/2020/08/12/cyclonedds/>

<https://rosindustrial.org/news/2020/7/8/developing-a-ros2-collaborative-industrial-scan-n-plan-application>

<https://www.adlinktech.com/en/CycloneDDS>

<https://github.com/eclipse-cyclonedds/cyclonedds#configuration>

<https://github.com/ros2/rmw_cyclonedds>

## 介绍

![](https://tcs.teambition.net/storage/3126b439a9f01bc66eb9bf4d46aabaa5e5fc?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDc2OCwiaWF0IjoxNjcxMTg5OTY4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjZiNDM5YTlmMDFiYzY2ZWI5YmY0ZDQ2YWFiYWE1ZTVmYyJ9.sSVDi3W-25_FzDpWspKtNr59ijCw14Y6R0AiUAUwwbU&download=image.png "")

ROS-I显示cyclone-dds使用起来比fastrtps和opensplice要好

## 性能

官方给出了两个性能度量: 吞吐量对带宽, 吞吐量对延迟

变量为: 消息个数, 消息尺寸, 吞吐量, 延迟, 带宽

可靠消息吞吐量:

对于小的消息吞吐量可达到1MS/s, 对于100byte的消息可以达到90%带宽,

延迟测试-ddsperf:

使用ddsperf测试延迟达到30us, 计算机性能为Intel(R) Xeon(R) CPU E3-1270 V2 @ 3.50GHz

延迟测试-消息订阅发布模式:

随着消息尺寸增大, 吞吐量增大, 延迟时间逐渐增加, 在吞吐量在10^6byte即1MB时, 时延达到10000us即10ms

![](https://tcs.teambition.net/storage/312681bd71c43df89f71eadb5d5fd6593384?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDc2OCwiaWF0IjoxNjcxMTg5OTY4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY4MWJkNzFjNDNkZjg5ZjcxZWFkYjVkNWZkNjU5MzM4NCJ9.gRLndDMY4Fda_Xy0iro3oMl2M54PefA5cRqY-RcH34s&download=image.png "")

## 配置

### 基础配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<CycloneDDS xmlns="https://cdds.io/config" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="https://cdds.io/config https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/master/etc/cyclonedds.xsd">
    <Domain id="any">
        <General>
            <NetworkInterfaceAddress>auto</NetworkInterfaceAddress>
            <AllowMulticast>default</AllowMulticast>
            <MaxMessageSize>65500B</MaxMessageSize>
            <FragmentSize>4000B</FragmentSize>
        </General>
        <Internal>
            <Watermarks>
                <WhcHigh>500kB</WhcHigh>
            </Watermarks>
        </Internal>
        <Tracing>
            <Verbosity>config</Verbosity>
            <OutputFile>stdout</OutputFile>
        </Tracing>
    </Domain>
</CycloneDDS>
$ export CYCLONEDDS_URI=file://$PWD/cyclonedds.xml

```

$ cat cyclonedds.xml

- NetworkInterfaceAddress

- 只能使用一个网络接口

- AllowMulticast

- 什么情况下使用组播

- 有线网络, 使用组播用于网络发现和多节点数据传输

- wifi网络, 只用于网络发现spdp, 传输数据不使用

- MaxMessageSize and FragmentSize

- 最大消息尺寸和段尺寸

- WhcHigh

- 发送方需要等多久ack报文

```text
# With large samples (100s of kilobytes), excessive latency can be caused by running out of space in the OS-level receive buffer. For this reason, on Linux, we recommend increasing the buffer size:

sudo sysctl -w net.core.rmem_max=8388608 net.core.rmem_default=8388608
echo "net.core.rmem_max=8388608\nnet.core.rmem_default=8388608\n" | sudo tee /etc/sysctl.d/60-cyclonedds.conf
sudo sysctl -w net.core.rmem_max=2147483647
net.core.rmem_max=2147483647

```

```text
<CycloneDDS xmlns="https://cdds.io/config" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="https://cdds.io/config
https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/master/etc/cyclonedds.xsd">
    <Domain id="any">
        <Internal>
            <MinimumSocketReceiveBufferSize>10MB</MinimumSocketReceiveBufferSize>
        </Internal>
    </Domain>
</CycloneDDS>

```

## 对SHM的支持

0.7.0没有，后面会集成iceoyrx，是ROS2默认DDS

0.8.0 可能会有，预计于2021.6发布到foxy

## 对Security支持

已经支持

## 配置选项

<https://github.com/eclipse-cyclonedds/cyclonedds/blob/master/docs/manual/options.md>
