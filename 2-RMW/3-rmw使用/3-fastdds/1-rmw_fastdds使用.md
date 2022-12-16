使用注意事项

# 总体思路

rmw会创建fastdds的实体，根据ros2的命名规则，在fastdds xml中根据对应的名字设置不同的配置，<br />fastdds程序启动后加载该xml文件到配置中，创建实体时查询配置中对应的名字是否已经有profile，如果有则按照该配置

# 同步与异步发布

RMW_FASTRTPS_PUBLICATION_MODE<br />rmw默认的配置<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663570090959-53ba9581-9c3f-45d4-989d-034c69524b0b.png#clientId=u59d527f1-82f4-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ua4f509c6&margin=%5Bobject%20Object%5D&originHeight=242&originWidth=929&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u756bee42-0a4a-4a1f-8d22-c1ce285f114&title=)

# 使用xml配置qos

xml配置和rmw_fastrtps的配置的关系？<br />Fast DDS QoS policies can be fully configured through a combination of the [rmw QoS profile](http://docs.ros2.org/latest/api/rmw/structrmw__qos__profile__t.html) API, and the [Fast DDS XML](https://fast-dds.docs.eprosima.com/en/latest/fastdds/xml_configuration/xml_configuration.html) file's QoS elements. Configuration depends on the environment variable RMW_FASTRTPS_USE_QOS_FROM_XML.<br />通过rmw qos 接口已经可以配置FastDDS的所有QOS选项，但是FastDDS还有很多可以配置的东西，可以通过xml来配置。

1. ROS 2 QoS contained in [rmw_qos_profile_t](http://docs.ros2.org/latest/api/rmw/structrmw__qos__profile__t.html) are always honored, unless set to *_SYSTEM_DEFAULT. In that case, XML values, or Fast DDS default values in the absence of XML ones, are applied. Setting any QoS in rmw_qos_profile_t to something other than*_SYSTEM_DEFAULT entails that specifying it via XML files has no effect, since they do not override what was used to create the publisher, subscription, service, or client.
   1. rmw_qos_profile_t 中包含的 ROS 2 QoS 始终得到尊重，除非设置为 *_SYSTEM_DEFAULT。 在这种情况下，将应用 XML 值或没有 XML 值的 Fast DDS 默认值。 将 rmw_qos_profile_t 中的任何 QoS 设置为*_SYSTEM_DEFAULT 以外的其他内容意味着通过 XML 文件指定它没有任何效果，因为它们不会覆盖用于创建发布者、订阅、服务或客户端的内容。
   2. 意思是，在rmw中指定的qos（不是system_default）是优先的，如果在rmw中设置了，则xml的配置不生效，但是如果rmw没配置，则xml的生效
2. In order to modify the history memory policy or publication mode using XML files, environment variable RMW_FASTRTPS_USE_QOS_FROM_XML must be set to 1 (it is set to 0 by default). This tells rmw_fastrtps that it should override both the history memory policy and the publication mode using the XML. Bear in mind that setting this environment variable but not setting either of these policies in the XML results in Fast DDS' defaults configurations being used.
   1. 配置了该环境变量后，如果rmw没配置某选项，xml也没配置某选项，则使用fastdds的默认值（而非rmw默认值，rmw默认值可能和fastdds默认值不一样）
   2. 配置了环境变量后，rmw配置了选项，依然优先于xml选项

~~当RMW_FASTRTPS_USE_QOS_FROM_XML=1，则读取xml，此时rmw默认配置失效，fastdds默认配置生效~~

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663570090899-88b60663-6912-4f10-9dde-72acec5d11ee.png#clientId=u59d527f1-82f4-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ua99cbe42&margin=%5Bobject%20Object%5D&originHeight=340&originWidth=925&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u4cc6dba4-0bc8-4bcb-abeb-8b2643aeff1&title=)<br />Note: Setting RMW_FASTRTPS_USE_QOS_FROM_XML to 1 effectively overrides whatever configuration was set with RMW_FASTRTPS_PUBLICATION_MODE. Furthermore, If RMW_FASTRTPS_USE_QOS_FROM_XML is set to 1, and history memory policy or publication mode are not specified in the XML, then the Fast DDS' default configurations will be used:<br />RMW_FASTRTPS_PUBLICATION_MODE这个环境变量会被xml环境变量覆盖<br />这个我一般不管，让ros开发者定

# xml文件位置

```bash
export FASTRTPS_DEFAULT_PROFILES_FILE=<path_to_xml_file>
```

# xml配置参与者

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663570090929-1384765e-9ad9-49bd-922d-6922b7300970.png#clientId=u59d527f1-82f4-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub8dd5298&margin=%5Bobject%20Object%5D&originHeight=474&originWidth=973&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u13db2fd0-d00a-405a-be25-9c8c19c0c25&title=)

# xml配置实体

- To configure a publisher, define a <publisher> profile with attribute profile_name=topic_name, where topic_name is the name of the topic before mangling, i.e., the topic name used to create the publisher. If such profile is not defined, _rmw_fastrtps_ attempts to load the <publisher> profile with attribute is_default_profile="true".
- To configure a subscriber, define a <subscriber> profile with attribute profile_name=topic_name, where topic_name is the name of the topic before mangling. If such profile is not defined, _rmw_fastrtps_ attempts to load the <subscriber> profile with attribute is_default_profile="true".

以数据为中心的中间件，定义qos时是以topic名字来配置的

# xml参考

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663570090965-a278baf3-268d-4b73-8807-1f944e4a8e8e.png#clientId=u59d527f1-82f4-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u27b00a32&margin=%5Bobject%20Object%5D&originHeight=919&originWidth=650&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ubb4ebd0a-e35f-4757-bcec-9cf13bd6663&title=)

1. Open one terminal and run:export RMW_IMPLEMENTATION=rmw_fastrtps_cppexport FASTRTPS_DEFAULT_PROFILES_FILE=path/to/xml/ros_example.xmlexport RMW_FASTRTPS_USE_QOS_FROM_XML=1ros2 run demo_nodes_cpp talker
2. Open one terminal and run:export RMW_IMPLEMENTATION=rmw_fastrtps_cppexport FASTRTPS_DEFAULT_PROFILES_FILE=path/to/xml/ros_example.xmlexport RMW_FASTRTPS_USE_QOS_FROM_XML=1ros2 run demo_nodes_cpp listener

# rclcpp

ROS_LOCALHOST_ONLY

# 节点内混合使用同步和异步发布

rmw_fastrtps uses asynchronous publication mode by default. When the publisher invokes the write operation, the data is copied into a queue, a background thread (asynchronous thread) is notified about the addition to the queue, and control of the thread is returned to the user before the data is actually sent. The background thread is in charge of consuming the queue and sending the data to every matched reader.<br />On the other hand, with synchronous publication mode the data is sent directly within the context of the user thread. This entails that any blocking call occurring during the write operation would block the user thread, thus preventing the application from continuing its operation. However, this mode typically yields higher throughput rates at lower latencies, since there is no notification nor context switching between threads.

## xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">

    <!-- default publisher profile -->
    <publisher profile_name="default_publisher" is_default_profile="true">
        <historyMemoryPolicy>DYNAMIC</historyMemoryPolicy>
    </publisher>

    <!-- publisher profile for topic sync_topic -->
    <publisher profile_name="/sync_topic">
        <historyMemoryPolicy>DYNAMIC</historyMemoryPolicy>
        <qos>
            <publishMode>
                <kind>SYNCHRONOUS</kind>
            </publishMode>
        </qos>
    </publisher>

    <!-- publisher profile for topic async_topic -->
    <publisher profile_name="/async_topic">
        <historyMemoryPolicy>DYNAMIC</historyMemoryPolicy>
        <qos>
            <publishMode>
                <kind>ASYNCHRONOUS</kind>
            </publishMode>
        </qos>
    </publisher>

 </profiles>
```

Note that several publisher profiles are defined. A default profile which is defined setting the is_default_profile to true, and two profiles with names that coincide with those of the previously defined topics: sync_topic and another one for async_topic. These last two profiles set the publication mode to SYNCHRONOUS or ASYNCHRONOUS accordingly. Note also that all profiles specify a historyMemoryPolicy value, which is needed for the example to work, and the reason will be explained later on this tutorial.

# 参考
<https://fast-dds.docs.eprosima.com/en/latest/fastdds/ros2/ros2_configure.html>
