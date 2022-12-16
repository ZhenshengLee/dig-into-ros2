# 概述

如何配置ros2 with fastdds，参考<https://fast-dds.docs.eprosima.com/en/latest/fastdds/ros2/ros2_configure.html>

# rmw接口

## rmw-shm-default
<https://fast-dds.docs.eprosima.com/en/latest/fastdds/transport/shared_memory/shared_memory.html><br />shm传输用于在进程内和单机进程中提升数据传输性能<br />通过配置xml文件来使能shm，目前fastdds默认使能shm传输

## rmw-shm-data-sharing
<https://fast-dds.docs.eprosima.com/en/latest/fastdds/transport/datasharing.html><br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663569941793-e3e87ca8-22e8-49eb-ac3c-7a3697647974.png#clientId=u0295de33-335e-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u42e80634&margin=%5Bobject%20Object%5D&originHeight=96&originWidth=738&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u79eb9ae3-1bad-4f51-b803-8b47735eb14&title=)

## rmw-shm-zc

目前还未能完全启用data-sharing，还有bug没解决

# rclcpp接口

## rclcpp-shm-default

默认接口<br />默认api使能shm

## rclcpp-shm-datasharing

loaned-api使能datasharing，<br /><https://github.com/ros2/rmw_fastrtps/issues/534>

```bash
RMW_FASTRTPS_USE_QOS_FROM_XML=1 FASTRTPS_DEFAULT_PROFILES_FILE=PUB.xml RMW_IMPLEMENTATION=rmw_fastrtps_cpp ros2 run bug_reproduce data_sharing_talker
RMW_FASTRTPS_USE_QOS_FROM_XML=1 FASTRTPS_DEFAULT_PROFILES_FILE=SUB.xml RMW_IMPLEMENTATION=rmw_fastrtps_cpp ros2 run bug_reproduce data_sharing_listener
```

要求数据类型pod，bounded<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663569941782-76f79332-5857-4b1d-82a1-2abed4b738c4.png#clientId=u0295de33-335e-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u4eccb3cb&margin=%5Bobject%20Object%5D&originHeight=140&originWidth=912&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ucc6c2548-4e9f-4106-84fa-2fb5a8e12c4&title=)

<https://github.com/ros2/rmw_fastrtps/issues/579><br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663569941791-e20d21e9-5f52-45a1-9dc3-0432dc475a0a.png#clientId=u0295de33-335e-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u3eca3e58&margin=%5Bobject%20Object%5D&originHeight=343&originWidth=834&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u5a3bf0c4-be5b-4847-b32f-7bc971b7f8e&title=)

通过ls /dev/shm 查看是否存在datasharing

## rclcpp-shm-datasharing-zc

目前还不支持

# 支持情况

可以使用shm

# 性能

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663569941803-63792614-6d25-4a19-bae3-87056acd6cda.png#clientId=u0295de33-335e-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u5ea93a71&margin=%5Bobject%20Object%5D&originHeight=575&originWidth=707&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ud4daa14b-c2db-450a-9afc-adb240cc52b&title=)

# 参考

Fast DDS now with Zero-Copy<br />Specialized on high performance middleware. Check out our comparatives such as Apache Thrift vs Protocol Buffers vs Fast Buffers or ZMQ vs Fast RTPS.<br /><https://www.eprosima.com/index.php/company-all/news/186-fast-dds-now-with-zero-copy>

Example for using Zero Copy with Galactic · Issue #596 · ros2/rmw_fastrtps<br />We are trying to find an example on how to use zero copy with Fast RTPS. Is there a simple example somewhere that we can refer? We tried to use Galactic to do it but saw this error. Currently used ...<br />![](https://cdn.nlark.com/yuque/0/2022/svg/23125517/1663569942498-ea1a0c89-6bea-466f-b5fe-05a0425f9669.svg#clientId=u0295de33-335e-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u570107de&margin=%5Bobject%20Object%5D&originHeight=32&originWidth=32&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u4a99f26f-2b74-4f2c-b980-99335f31c0e&title=)<https://github.com/ros2/rmw_fastrtps/issues/596>

How to enable LoanedMessages · Issue #579 · ros2/rmw_fastrtps<br />Bug report Required Info: Operating System: Ubuntu 20.04 Installation type: source Version or commit hash: ros2/ros2@4a36f31 DDS implementation: Fast-DDS Client library (if applicable): rclcpp Step...<br />![](https://cdn.nlark.com/yuque/0/2022/svg/23125517/1663569942780-0348b77c-85fb-4c73-9d89-0ad2f20204f9.svg#clientId=u0295de33-335e-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub70b324f&margin=%5Bobject%20Object%5D&originHeight=32&originWidth=32&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uf7c509a0-34c2-489f-b253-ef3b53aa026&title=)<https://github.com/ros2/rmw_fastrtps/issues/579><br />16.1. Configuring Fast DDS in ROS 2 — Fast DDS 2.6.0 documentation

<https://fast-dds.docs.eprosima.com/en/latest/fastdds/ros2/ros2_configure.html>

# 相关工作

## rmw_sony_cpp

这个未开源，不过是sone和eprosima合作订制开发的<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663569942942-738676ec-edbf-4666-bce0-fd187b2ba1d6.png#clientId=u0295de33-335e-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u0c2b7cbe&margin=%5Bobject%20Object%5D&originHeight=573&originWidth=1063&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u5e494cbc-805d-4a52-8d18-0c9b28c4bda&title=)
