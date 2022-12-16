# 发现机制

通过底层的middleware来实现

1. 节点启动时，通过网络向同一个ROS域（其实是dds域）（ROS_DOMAIN_ID）所有节点广播。所有节点回应，然后建立相应的连接。（与二层交换网络类似）

2. 节点定期广播运行状态，所以连接可以被新建立的节点发现

3. 节点下线时向其他所有节点告知下线状态

节点只会与QOS配置兼容的节点建立连接！

# 参考

<http://docs.ros.org/en/rolling/Tutorials/Discovery-Server/Discovery-Server.html>

[Topological Discovery and Communication Negotiation](https://design.ros2.org/articles/discovery_and_negotiation.html)
