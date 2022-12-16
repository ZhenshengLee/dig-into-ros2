# 总体方法

```bash
RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
CYCLONEDDS_URI=~/.ros/cyclonedds.xml
```

# 配置

[__https://github.com/eclipse-cyclonedds/cyclonedds/blob/master/docs/manual/config.rst__](https://github.com/eclipse-cyclonedds/cyclonedds/blob/master/docs/manual/config.rst)

## 关闭多播

```text
export ROS_LOCALHOST_ONLY=1
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<CycloneDDS xmlns="https://cdds.io/config" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    xsi:schemaLocation="https://cdds.io/config https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/master/etc/cyclonedds.xsd">
    <Domain id="42">
        <General>
            <NetworkInterfaceAddress>lo</NetworkInterfaceAddress>
            <AllowMulticast>false</AllowMulticast>
        </General>
        <Discovery>
            <ParticipantIndex>auto</ParticipantIndex>
            <MaxAutoParticipantIndex>120</MaxAutoParticipantIndex>
            <Peers>
                <Peer address="localhost"/>
            </Peers>
        </Discovery>
    </Domain>
</CycloneDDS>

```

## 零拷贝

...

# 参考

[GitHub - ros2/rmw_cyclonedds: ROS 2 RMW layer for Eclipse Cyclone DDS](https://github.com/ros2/rmw_cyclonedds)

[cyclonedds/config.rst at master · eclipse-cyclonedds/cyclonedds](https://github.com/eclipse-cyclonedds/cyclonedds/blob/master/docs/manual/config.rst)
