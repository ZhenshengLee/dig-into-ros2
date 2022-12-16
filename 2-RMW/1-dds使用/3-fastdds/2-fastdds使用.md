2.3.4

# 编译选项

SECURITY，激活信息安全支持，默认OFF，手动开启<br />SHM_TRANSPORT_DEFAULT，默认传输方式为SHM，默认开启<br />STRICT_REALTIME，严格实时配置，默认关闭

# 传输模式

fastdds定义了一个帮助配置通信通道的locator_t，A Locator_t uniquely identifies a communication channel with a remote peer for a particular transport. For example, on UDP transports, the Locator will contain the information of the IP address and port of the remote peer.<br />The Locator class is not abstract, and no specializations are implemented for each transport type. Instead, transports should map the data members of the Locator class to their own channel identification concepts. For example, on Shared Memory Transport the address contains a unique ID for the local host, and the port represents the shared ring buffer used to communicate buffer descriptors.<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565301296-97eb8fce-a855-4907-bf3f-f2d62534b813.png#clientId=ub9f0550b-c05a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u5fb211a8&margin=%5Bobject%20Object%5D&originHeight=327&originWidth=973&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u62dfd589-b124-42a3-83bb-e4fe0fb8160&title=)

## shm-default

SHM transport provides better performance than other network transports like UDP / TCP, even when these transports use loopback interface. This is mainly due to the following reasons:<br />Large message support: Network protocols need to fragment data in order to comply with the specific protocol and network stacks requirements, increasing communication overhead. SHM transport allows the copy of full messages where the only size limit is the machine’s memory capacity.<br />Reduce the number of memory copies: When sending the same message to different endpoints, SHM transport can directly share the same memory buffer with all the destination endpoints. Other protocols require to perform one copy of the message per endpoint.<br />Less operating system overhead: Once initial setup is completed, shared memory transfers require much less system calls than the other protocols. Therefore, there is a performance/time consume gain by using SHM.<br />dashing版本要用上fastdds2.0, 需要重新编译ROS2版本<br />foxy 版本2.0.2, Shared memory transport enabled by default，但是还没有完全<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565301303-eb9603a9-1c04-4791-b1d2-e100f444bb34.png#clientId=ub9f0550b-c05a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u613a3790&margin=%5Bobject%20Object%5D&originHeight=635&originWidth=868&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u33eda1d4-8e4a-42ad-82a5-b6c60380bec&title=)

### 注意

默认的segsize是512kB<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663579969411-4976cec1-af09-4edb-af4a-864954a034d1.png#clientId=ud7edba3f-8036-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=354&id=u86715e34&margin=%5Bobject%20Object%5D&name=image.png&originHeight=354&originWidth=964&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44126&status=done&style=none&taskId=u47a94c48-bb44-4c03-bb12-5b5f74069ec&title=&width=964)<br />在不同版本混合运行之后可能会报错：

```bash
2022-03-27 12:15:30.282 [RTPS_TRANSPORT_SHM Error] Failed init_port fastrtps_port7412: open_and_lock_file failed -> Function open_port_internal
```

此时删除/dev/shm内容，重试

```bash
# 检查可配置了最大值
sudo sysctl -a | grep shmmax
# 检查shm空间
$ df -Th | grep '/dev/shm\|Used'
Filesystem     Type      Size  Used Avail Use% Mounted on
tmpfs          tmpfs      32G  2.4G   29G   8% /dev/shm
```

### 判断是否启用shm

```bash
# 通过ls /dev/shm来检查是否启用了shm
# 查看terminal是否报错，fastrtps是否报错
# 上述均正常，则说明启用了shm传输
```

### docker配置

```bash
ipc=host
```

参考<br />[https://github.com/pytorch/pytorch/issues/1158](https://github.com/pytorch/pytorch/issues/1158)

## shm-data-sharing

2.3.2-full zero-copy Shared Memory Transport <br />This feature is available only if the following requirements are met:

- The DataWriter and DataReader have access to the same shared memory.
- The Topic has a bounded TopicDataType, i.e., its is_bounded() member function returns true.
- The Topic is not keyed.
- The DataWriter is configured with PREALLOCATED_MEMORY_MODE or PREALLOCATED_WITH_REALLOC_MEMORY_MODE.

There is also a limitation with the DataReader’s HistoryQos. Using Data-sharing mechanism, the DataWriter’s history is shared with the DataReaders. This means that the effective HistoryQos depth on the DataReader is, at most, the Datawriter’s HistoryQos depth. To avoid confusions, set the DataReaders’ history depth to a value equal or less than the DataWriter’s.<br />必须要有界大小数据<br />Data-sharing delivery can be configured in the DataWriter and the DataReader using DataSharingQosPolicy. Four attributes can be configured:

- The data-sharing delivery kind
- The shared memory directory
- The data-sharing domain identifiers.
- The maximum number of data-sharing domain identifiers.

配置一个qos到datawriter和datareader

## shm-datasharing-zc

在data-sharing的基础上loan-api<br />要求data-sharing的基础上进行zero-copy<br />要求qos配置

## intra

默认打开，交换指针

## local-mode

去除广播通信<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565301316-2fe62fbc-1b33-4a81-bd8f-32eb8931af2f.png#clientId=ub9f0550b-c05a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u240c31fb&margin=%5Bobject%20Object%5D&originHeight=405&originWidth=786&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ud5dbffde-19fe-40d8-bb89-cda3ca1db65&title=)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles>
  <participant profile_name="participant_profile" is_default_profile="true">
    <rtps>
      <builtin>
        <metatrafficUnicastLocatorList>
          <locator/>
        </metatrafficUnicastLocatorList>
        <initialPeersList>
          <locator>
            <udpv4>
              <address>127.0.0.1</address>
            </udpv4>
          </locator>
        </initialPeersList>
      </builtin>
    </rtps>
  </participant>
</profiles>
```

# 实时选项

- 内存在初始化时候分配
- 阻塞函数调用建立超时机制

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565301307-8f8c9db1-13a0-45cb-be56-b7c23bdec8f5.png#clientId=ub9f0550b-c05a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uaab079f3&margin=%5Bobject%20Object%5D&originHeight=191&originWidth=377&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ud5eed68a-1f4f-40ec-98d7-ea292a3a0f2&title=)

# XML配置

An XML profile is defined by a unique name that is used to reference the XML profile during the creation of an Entity, the Trasport configuration, or the DynamicTypes definition.<br />用于决定实体的创建，传输层配置，动态类型的定义<br />配置不是全局粒度，而是参与者为粒度，支持多种配置，在程序中初始化环境时可加载不同的profile.xml

## 加载机制

- 加载当前目录的FASTRTPS_DEFAULT_PROFILES_FILE.xml
- FASTRTPS_DEFAULT_PROFILES_FILE环境变量指定的文件
- 载入class定义的参数，不加载文件SKIP_DEFAULT_XML

## 模板

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565301295-66e86838-04fb-4fab-a7c7-6a063cd197ef.png#clientId=ub9f0550b-c05a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=udd988e05&margin=%5Bobject%20Object%5D&originHeight=496&originWidth=629&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u8259c5bd-a51c-4e45-8fb9-217c151452e&title=)

# 环境变量

export FASTRTPS_DEFAULT_PROFILES_FILE=/home/user/profiles.xml

# 场景策略

## OverWifi

为了去除发现协议带来的不必要的流量，采用配置固定IP的方式，initialpeerlist设置本机就可以了

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <participant profile_name="disable_multicast" is_default_profile="true">
        <rtps>
            <builtin>
                <metatrafficUnicastLocatorList>
                    <locator/>
                </metatrafficUnicastLocatorList>
                <initialPeersList>
                    <locator>
                        <udpv4>
                            <address>192.168.0.1</address>
                        </udpv4>
                    </locator>
                </initialPeersList>
            </builtin>
        </rtps>
    </participant>
</profiles>
```

## LargeDataRates

### 提高缓冲区的大小

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <participant profile_name="participant_xml_profile_qos_socketbuffers">
        <rtps>
            <sendSocketBufferSize>1048576</sendSocketBufferSize>
            <listenSocketBufferSize>4194304</listenSocketBufferSize>
        </rtps>
    </participant>
</profiles>
```

找到和配置系统最大值<br />For socket buffers used to send data, use the following command:<br />$> sudo sysctl -a | grep net.core.wmem_max net.core.wmem_max = 1048576 <br />For socket buffers used to receive data the command is:<br />$> sudo sysctl -a | grep net.core.rmem_max net.core.rmem_max = 4194304 <br />However, these maximum values are also configurable and can be increased if needed. The following command increases the maximum buffer size of sending sockets:<br />$> sudo sysctl -w net.core.wmem_max=12582912 <br />For receiving sockets, the command is:<br />$> sudo sysctl -w net.core.rmem_max=12582912

### 提高接口传输队列长度txqueuelen

linux only，新功能，暂时不用

### 使用流量控制机制

避免流量burst，

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <publisher profile_name="publisher_profile_qos_flowcontroller">
        <throughputController>
            <bytesPerPeriod>300000</bytesPerPeriod>
            <periodMillisecs>1000</periodMillisecs>
        </throughputController>
    </publisher>
</profiles>
```

### qos机制

使用best_effort_reliable机制，减少发包过程中的ACK确认机制<br />使用keep_last_history订阅，允许丢包<br />对于变长数据类型，使用PREALLOCATED_WITH_REALLOC<br />使用reliable_reliable机制，not strict reliable qos, <br />When HistoryQosPolicyKind is set as KEEP_ALL_HISTORY_QOS, all samples have to be received (and acknowledged) by all subscribers before they can be overridden by the DataWriter. If the message rate is high and the network is not reliable (i.e., lots of packets get lost), the history of the DataWriter can be filled up, blocking the publication of new messages until any of the old messages is acknowledged by all subscribers.<br />If this strictness is not needed, HistoryQosPolicyKind can be set as KEEP_ALL_HISTORY_QOS. In this case, when the history of the DataWriter is full, the oldest message that has not been fully acknowledged yet is overridden with the new one. If any subscriber did not receive the discarded message, the publisher will send a GAP message to inform the subscriber that the message is lost forever.

## ManySubscribers

每一次datawriter发布一次变更，便广播一次变更给所有订阅该消息的datareader，如果datareader很多，推荐使用组播而非广播，在这种情况下，对于每一个sample只有一个网络包发送，减少cpu和网络负载<br />解决方案可以是udp transport或者是shm transport，shm在intra-machine是默认启用的

# 通用策略

## 避免广播

If all the peers are known beforehand, it is possible to disable the multicast meta traffic completely. This is done using the configuration attribute metatrafficUnicastLocatorList. By defining a custom metatrafficUnicastLocatorList, the default metatraffic multicast and unicast locators to be employed by the participant are avoided, which prevents the participant from listening to any discovery data from multicast sources. The local participant creates a meta traffic receiving resource per address-port pair specified in the metatrafficUnicastLocatorList.<br />提前指定好peer的IP地址，就可以避免广播发现机制

```xml
<participant profile_name="initial_peers_multicast_avoidance" is_default_profile="true" >
  <rtps>
    <builtin>
      <!-- Choosing a specific unicast address -->
      <metatrafficUnicastLocatorList>
        <locator>
          <udpv4>
            <address>192.168.10.13</address>
            <port>7412</port>
          </udpv4>
        </locator>
      </metatrafficUnicastLocatorList>
    </builtin>
  </rtps>
</participant>
```

diabling all multicast，指定initial peer list即可

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <participant profile_name="disable_multicast" is_default_profile="true">
        <rtps>
            <builtin>
                <metatrafficUnicastLocatorList>
                    <locator/>
                </metatrafficUnicastLocatorList>
                <initialPeersList>
                    <locator>
                        <udpv4>
                            <address>192.168.0.1</address>
                        </udpv4>
                    </locator>
                </initialPeersList>
            </builtin>
        </rtps>
    </participant>
</profiles>
```

注意：该配置影响默认的shm传输，导致传输异常<br />经过社区确认，使用下列配置可同时保证shm和disable-multicast

## 限制订阅者数量

可以增强性能，但效果未知

```xml
<!-- publisher profile for topic async_topic -->
<publisher profile_name="/async_topic">
    <historyMemoryPolicy>DYNAMIC</historyMemoryPolicy>
    <qos>
        <publishMode>
            <kind>ASYNCHRONOUS</kind>
        </publishMode>
    </qos>
    <matchedSubscribersAllocation>
        <initial>0</initial>
        <maximum>1</maximum>
        <increment>1</increment>
    </matchedSubscribersAllocation>
</publisher>
```

## 使用话题分区partitions

This Qos Policy allows the introduction of a logical partition inside the physical partition introduced by a domain. For a DataReader to see the changes made by a DataWriter, not only the Topic must match, but also they have to share at least one logical partition (see PartitionQosPolicy).<br />datareader要能够接受到change的通知，除了订阅话题一致外，partitions分区信息还需要一致，这个是为了减轻网络负载。<br />Partitions introduce a logical entity isolation level concept inside the physical isolation induced by a Domain ID. For a publisher to communicate with a subscriber,**they have to belong at least to one common partition**. Partitions represent another level to separate publishers and subscribers beyond domain and topic. Unlike domain and topic, an endpoint can belong to several partitions at the same time. For certain data to be shared over different domains or topics, there must be a different publisher for each, sharing its own history of changes. However, a single publisher can share the same data sample over different partitions using a single topic data change, thus reducing network overload.

```xml
<!-- publisher profile for topic sync_topic -->
<publisher profile_name="/sync_topic">
    <historyMemoryPolicy>DYNAMIC</historyMemoryPolicy>
    <qos>
        <publishMode>
            <kind>SYNCHRONOUS</kind>
        </publishMode>
        <partition>
            <names>
                <name>part1</name>
            </names>
        </partition>
    </qos>
</publisher>

<!-- subscriber profile for topic sync_topic -->
<subscriber profile_name="/sync_topic">
    <historyMemoryPolicy>DYNAMIC</historyMemoryPolicy>
    <qos>
        <partition>
            <names>
                <name>part2</name>
            </names>
        </partition>
    </qos>
</subscriber>
```

# 弃用-FastRTPS-API层

# 参考

**DDS API — Fast DDS 2.3.4 documentation**<br /><https://fast-dds.docs.eprosima.com/en/latest/index.html>

[https://docs.ros.org/en/rolling/Tutorials/Advanced/FastDDS-Configuration.html#limiting-the-number-of-matching-subscribers](https://docs.ros.org/en/rolling/Tutorials/Advanced/FastDDS-Configuration.html#limiting-the-number-of-matching-subscribers)
