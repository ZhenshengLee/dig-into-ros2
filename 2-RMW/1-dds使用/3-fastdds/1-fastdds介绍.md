# 参考
<https://github.com/eProsima/Fast-DDS><br /><https://discourse.ros.org/t/new-features-for-fast-dds-ros2-foxy/17376><br /><https://www.eprosima.com/index.php/resources-all/performance/fast-dds-vs-cyclone-dds-performance><br /><https://www.eprosima.com/index.php/resources-all/performance/eprosima-fast-dds-performance><br /><https://discourse.ros.org/t/how-to-use-fast-2-0-x-on-dashing/15623><br /><https://discourse.ros.org/t/eprosima-fast-dds-from-shared-memory-to-zero-copy/18877><br /><https://discourse.ros.org/t/fast-dds-v2-2-0-latency-performance/18989>

# 介绍

开源, 企业支持, 开源版本不是阉割版本<br />2.2.0开始支持Extension APIs allowing zero-copy delivery (both intra-process and inter-process)

# 性能

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565280165-c7a7b4cf-4541-4de6-b5b0-f33b3c0b6bd3.png#averageHue=%23fbfafa&clientId=ub2eeef93-5a2d-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u202fdab9&margin=%5Bobject%20Object%5D&originHeight=877&originWidth=735&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u968ab9f6-5028-4e6a-afff-ea7a7cef220&title=)

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565280152-7702cd2e-6d19-42e5-a599-ab1a187b3632.png#averageHue=%23fafafa&clientId=ub2eeef93-5a2d-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uedb9fa30&margin=%5Bobject%20Object%5D&originHeight=842&originWidth=1142&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ubff08ce6-f2e1-4f76-b5d6-c40503fa3ae&title=)

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663565280167-48e77d85-a4e5-4692-936d-6d2d6c5e4b0b.png#averageHue=%23fcfbfb&clientId=ub2eeef93-5a2d-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ue82d2040&margin=%5Bobject%20Object%5D&originHeight=808&originWidth=1149&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uebf3b985-587d-4fb3-824e-5af7ef66332&title=)

# 配置

配置文件格式`https://github.com/eProsima/Fast-DDS/blob/master/resources/xsd/fastRTPS_profiles.xsd`

# 示例程序

如何从RCL用上SHM，要看不同中间件的实现。<br />rmw_iceoryx不管是普通通信还是loan都是SHM的<br />2. Fast DDS Monitor on Linux — Fast DDS Monitor 1.1.0 documentation<br /><https://fast-dds-monitor.readthedocs.io/en/latest/rst/installation/linux.html>

# 版本

[https://github.com/ZhenshengLee/rosdistro/blob/master/galactic/distribution.yaml](https://github.com/ZhenshengLee/rosdistro/blob/master/galactic/distribution.yaml)<br />中，到fastrtps下的source下查看<br />Foxy：2.1.x<br />Galactic: 2.3.x<br />Humble: 2.6.x
