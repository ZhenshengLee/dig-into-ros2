## 参考

[https://design.ros2.org/articles/ros2_security_enclaves.html](https://design.ros2.org/articles/ros2_security_enclaves.html)

<https://github.com/ros2/sros2/blob/master/SROS2_Linux.md>

## 创建安全文件

```bash
mkdir ~/sros2_demo
cd ~/sros2_demo
ros2 security create_keystore demo_keystore
# Generate keys and certificates for the talker and listener nodes
# for rolling
#ros2 security create_enclave demo_keystore /talker_listener/talker
#ros2 security create_enclave demo_keystore /talker_listener/listener
# for foxy
ros2 security create_key demo_keystore /talker_listener/talker
ros2 security create_key demo_keystore /talker_listener/listener

```

## 确定环境变量

```bash
export ROS_SECURITY_KEYSTORE=~/sros2_demo/demo_keystore
export ROS_SECURITY_ENABLE=true
export ROS_SECURITY_STRATEGY=Enforce

```

## 选择支持安全功能的rmw

```text
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

## 安全策略配置

```bash
zs@zs-3630:~/sros2_demo/demo_keystore/enclaves/talker_listener/listener$ ll
total 24
drwxrwxr-x 2 zs zs 4096 6月  18 09:16 ./
drwxrwxr-x 4 zs zs 4096 6月  18 09:16 ../
-rw-rw-r-- 1 zs zs  505 6月  18 09:16 cert.pem
lrwxrwxrwx 1 zs zs   20 6月  18 09:16 governance.p7s -> ../../governance.p7s
lrwxrwxrwx 1 zs zs   36 6月  18 09:16 identity_ca.cert.pem -> ../../../public/identity_ca.cert.pem
-rw-rw-r-- 1 zs zs  241 6月  18 09:16 key.pem
lrwxrwxrwx 1 zs zs   39 6月  18 09:16 permissions_ca.cert.pem -> ../../../public/permissions_ca.cert.pem
-rw-rw-r-- 1 zs zs 3800 6月  18 09:16 permissions.p7s
-rw-rw-r-- 1 zs zs 2191 6月  18 09:16 permissions.xml
```

可以看到，所有节点的安全策略中

- governance是公共的，是一些全局安全配置

- ca是公共的，这可以理解

- permissions.xml是p7s中的一个副本

  - p7s文件是加密通信文件

- key和cert是基于认证通信的证书，这两个文件是通信的关键

### governance全局安全配置

```xml
<dds xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://www.omg.org/spec/DDS-SECURITY/20170901/omg_shared_ca_governance.xsd">
    <domain_access_rules>
        <domain_rule>
            <domains>
              <id>42</id>
            </domains>
            <allow_unauthenticated_participants>false</allow_unauthenticated_participants>
            <enable_join_access_control>true</enable_join_access_control>
            <discovery_protection_kind>ENCRYPT</discovery_protection_kind>
            <liveliness_protection_kind>ENCRYPT</liveliness_protection_kind>
            <rtps_protection_kind>SIGN</rtps_protection_kind>
            <topic_access_rules>
                <topic_rule>
                    <topic_expression>*</topic_expression>
                    <enable_discovery_protection>true</enable_discovery_protection>
                    <enable_liveliness_protection>true</enable_liveliness_protection>
                    <enable_read_access_control>true</enable_read_access_control>
                    <enable_write_access_control>true</enable_write_access_control>
                    <metadata_protection_kind>ENCRYPT</metadata_protection_kind>
                    <data_protection_kind>ENCRYPT</data_protection_kind>
                </topic_rule>
            </topic_access_rules>
        </domain_rule>
    </domain_access_rules>
</dds>

```

### permission通信限制

设置该节点允许发布和订阅的消息，这个通道是dds通道

```xml
<dds xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://www.omg.org/spec/DDS-SECURITY/20170901/omg_shared_ca_permissions.xsd">
  <permissions>
    <grant name="/talker_listener/listener">
      <subject_name>CN=/talker_listener/listener</subject_name>
      <validity>
        <not_before>2021-06-17T01:16:05</not_before>
        <not_after>2031-06-16T01:16:05</not_after>
      </validity>
      <allow_rule>
        <domains>
          <id>42</id>
        </domains>
        <publish>
          <topics>
            <topic>rq/*/_action/cancel_goalRequest</topic>
            <topic>rq/*/_action/get_resultRequest</topic>
            <topic>rq/*/_action/send_goalRequest</topic>
            <topic>rq/*Request</topic>
            <topic>rr/*/_action/cancel_goalReply</topic>
            <topic>rr/*/_action/get_resultReply</topic>
            <topic>rr/*/_action/send_goalReply</topic>
            <topic>rt/*/_action/feedback</topic>
            <topic>rt/*/_action/status</topic>
            <topic>rr/*Reply</topic>
            <topic>rt/*</topic>
          </topics>
        </publish>
        <subscribe>
          <topics>
            <topic>rq/*/_action/cancel_goalRequest</topic>
            <topic>rq/*/_action/get_resultRequest</topic>
            <topic>rq/*/_action/send_goalRequest</topic>
            <topic>rq/*Request</topic>
            <topic>rr/*/_action/cancel_goalReply</topic>
            <topic>rr/*/_action/get_resultReply</topic>
            <topic>rr/*/_action/send_goalReply</topic>
            <topic>rt/*/_action/feedback</topic>
            <topic>rt/*/_action/status</topic>
            <topic>rr/*Reply</topic>
            <topic>rt/*</topic>
          </topics>
        </subscribe>
      </allow_rule>
      <allow_rule>
        <domains>
          <id>42</id>
        </domains>
        <publish>
          <topics>
            <topic>ros_discovery_info</topic>
          </topics>
        </publish>
        <subscribe>
          <topics>
            <topic>ros_discovery_info</topic>
          </topics>
        </subscribe>
      </allow_rule>
      <default>DENY</default>
    </grant>
  </permissions>
</dds>

```

### AC policies

permission意味着对指定主题的数据进行限制

access control意味着对指定节点的行为进行限制，比如对某主题的pub和sub是否允许

```bash
ros2 security create_permission demo_keys /talker_listener/talker policies/sample.policy.xml
ros2 security create_permission demo_keys /talker_listener/listener policies/sample.policy.xml

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<profile>
  <topics publish="ALLOW" >
    <topic>rosout</topic>
  </topics>
</profile>

```

详细xml定义见

## rosrun

```text
ros2 run demo_nodes_cpp talker --ros-args --enclave /talker_listener/talker
ros2 run demo_nodes_py listener --ros-args --enclave /talker_listener/listener
```

上述节点会使用加密和认证进行通信。

## roslaunch

![](https://tcs.teambition.net/storage/312690b5cafab2f361835913c438ed049084?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTY5NDA0OCwiaWF0IjoxNjcxMDg5MjQ4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY5MGI1Y2FmYWIyZjM2MTgzNTkxM2M0MzhlZDA0OTA4NCJ9.dHPIf73QAQx9-0BV6y_pHzpjRuEGyJPejXKom_8EWic&download=image.png "")

![](https://tcs.teambition.net/storage/31268dfc4b53efd569a5b756c5a593dc520f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTY5NDA0OCwiaWF0IjoxNjcxMDg5MjQ4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjY4ZGZjNGI1M2VmZDU2OWE1Yjc1NmM1YTU5M2RjNTIwZiJ9.Q9N6Tv998paWlFOLRHlsqSP8gMCtW7NSPWitpn62t-I&download=image.png "")
