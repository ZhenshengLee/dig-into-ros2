# 概述

localhost是回环接口，socket通信的地址可以设置成localhost<br />在A类IP地址中，有这样一项规定：网络号为127（即二进制01111111）保留作为本地软件环回测试本主机的进程之间的通信之用，若主机发送一个目的地址为环回地址（例如127.0.0.1）的IP数据报，则本主机中的协议软件就处理数据报中的数据，而不会把数据报发送到任何网络。<br />172.0.0.1是本地还回地址，专供自己访问自己，速度快(不用经过整个协议栈)，永远都不能出现在主机外部的网络中，所以只适合用在开发环境。<br />localhost是本地DNS解析的127.0.0.1的域名，在hosts文件里可以看到。<br />localhost：是不经网卡传输的，它不受网络防火墙和网卡相关的限制<br />127.0.0.1：是通过网卡传输的，它依赖网卡，并受到网络防火墙和网卡相关的限制

# 对发现协议行为的影响

P.S. ROS_LOCALHOST_ONLY causes it to use the loopback interface, that is:

- it advertises only loopback addresses
- it sets the multicast transmit interface to loopback
- it joins the multicast group only on the loopback interface

# 对组播行为的影响

不可以避免广播，而是将广播的地址换成了loopback

# 结论

应该用dds 配置避免组播<br />This is the Fast-DDS configuration that I’m currently using with ROS 2 Galactic and Fast-DDS 2.3.3.<br />It’s made of two slightly different XMLs, one for the robot and one for the laptop.<br />The advantage of this approach is that the robot can be configured in an extremely generic way, because it does not need to know in advance the IP address of the remote machines it will interact with.<br />On the other hand on the laptop, where it’s usually easier to interact/change configurations and it is expected that you always know the IP address of the robot you want to communicate with, the IP address is specified.

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
              <address>127.0.0.1</address>
            </udpv4>
          </locator>
        </initialPeersList>
      </builtin>
    </rtps>
  </participant>
</profiles>
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <participant profile_name="unicast_connection" is_default_profile="true">
        <rtps>
            <builtin>
                <metatrafficUnicastLocatorList>
                    <locator/>
                </metatrafficUnicastLocatorList>
                <initialPeersList>
                    <locator>
                        <udpv4>
                            <address>$ROBOT_IP</address>
                        </udpv4>
                    </locator>
                </initialPeersList>
            </builtin>
        </rtps>
    </participant>
</profiles>
```

# 参考

Restrict DDS middleware network traffic to localhost · Issue #798 · ros2/ros2<br /><https://github.com/ros2/ros2/issues/798><br /><https://github.com/ros2/rmw_cyclonedds/issues/370><br /><https://answers.ros.org/question/365051/using-ros2-offline-ros_localhost_only1/><br />Unconfigured DDS considered harmful to Networks<br /><https://discourse.ros.org/t/unconfigured-dds-considered-harmful-to-networks/25689?page=2><br />[https://github.com/ros2/rmw_cyclonedds/issues/318](https://github.com/ros2/rmw_cyclonedds/issues/318)<br />[https://answers.ros.org/question/365051/using-ros2-offline-ros_localhost_only1/](https://answers.ros.org/question/365051/using-ros2-offline-ros_localhost_only1/)<br />[https://blog.csdn.net/m0_46360532/article/details/114449788](https://blog.csdn.net/m0_46360532/article/details/114449788)
