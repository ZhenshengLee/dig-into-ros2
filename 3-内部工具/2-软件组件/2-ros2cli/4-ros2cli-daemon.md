# 概述

dds使用分布式服务协议来发现各个参与者<br />ros2也使用分布式协议来发现各个节点，节点通过分布式协议发现所有节点，通常耗时较长，所以将信息缓存在ROS Graph中，有一个后台守护进程长期运行，储存ROS Graph的信息，用来响应一些查询，如ros2 topic list

# daemon本质

The solution is the ROS 2 daemon. This is a node that sits around all day just listening to discovery protocol traffic and making a record of which nodes are alive and where they are. Then, when a new node wants to know where other nodes are, it can bypass its own DDS discovery process and contact the ROS 2 daemon on a known port and ask it for the graph information.<br />daemon是一个节点，监听发现服务的报文，将节点间信息记录下来，当新节点启动，可以绕过DDS服务发现协议，联系daemon，通过已知的接口来请求已经缓存的信息<br />daemon不会默认启动，而是第一个使用到ros2-daemon的cli，比如ros2 topic list<br />daemon是一个node，galactic\src\ros2\ros2cli\ros2cli\ros2cli\node\daemon.py

# 与DDS发现协议的关系

daemon在停止状态下，也不影响节点间通信，这是为什么？<br />The nice thing about this scheme is that a new node still performs its standard DDS discovery process, so if the ROS 2 daemon does not exist, then it will still be able to find other new nodes and function properly. This is a significant difference from ROS 1, where if the master does not exist your node simply cannot function. Similarly, if the new node starts very shortly after the ROS 2 daemon, then the daemon won't yet have cached much information about existing nodes. However in this case as well the new node can still find the missing information from its DDS discovery process, so it is robust to that potential problem as well. This is different from ROS 1 where if you restarted the master after it died, any new nodes would function but they would have no way of finding nodes that were around before the master was restarted.<br />在daemon运行下，dds的发现协议依然会运行，所以daemon停止，节点依然能够正常工作。如果daemon启动后，一个新的节点马上启动，则可能由于daemon没有缓存足够多的信息，节点不会找到指定的接口，但是节点依然会通过dds服务发现协议来完成通信发现。

# 与ros2cli的关系

ros2cli的工具都是来侦测ROS2系统的信息，有一些命令通过daemon来获取信息，有一些通过新其自身节点来获取信息<br />比如ros2 topic list就是通过daemon来获取信息

# 数据一致性问题

ros2 topic list读的是cache的信息，而node通信读的是dds图信息<br />可能出现不一致<br />完美情况，应该直接在dds接口上写app，而不是在rclcpp上<br />这也是rti的解决方案

# 与localhost_only的关系

[https://github.com/ros2/rmw_cyclonedds/issues/370](https://github.com/ros2/rmw_cyclonedds/issues/370)<br />不明确

# 与ROS1 Master的关系

 The ROS 2 daemon fulfils the same role that the master fills in ROS 1. However, unlike ROS 1, it is not required, but an optimisation. ROS 2 nodes can function without it, unlike ROS 1 nodes which cannot function without the master.

# 规范

要不要默认避免启动ros2 daemon?<br />cli避免使用daemon的信息？

```bash
ros2 topic list --no-daemon
ros2 node info /talker --no-daemon --spin-time 2
```

# 参考

Introspection with command line tools — ROS 2 Documentation: Foxy documentation<br /><https://docs.ros.org/en/foxy/Concepts/About-Command-Line-Tools.html><br />What is ROS2 daemon? - ROS Answers: Open Source Q&A Forum<br />I am able to run ros2 topic pub /chatter std_msgs/String "data: Hello ROS Developer" this command and this command also able to publish data without starting ros2 daemon. What exactly is the function of ROS2 daemon?<br /><https://answers.ros.org/question/327348/what-is-ros2-daemon/><br />Restrict DDS middleware network traffic to localhost · Issue #798 · ros2/ros2<br />Feature request Feature description Currently and by default, most DDS-based ROS middlewares lookup and choose a suitable network interface on the machine that&#39;s hosting them. It makes for easy...<br /><https://github.com/ros2/ros2/issues/798>
