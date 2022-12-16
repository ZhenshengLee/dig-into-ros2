# python-api

## rclpy

python-api-of-rcl，完全封装rcl

## ros2cli-api

ros2cli内部的接口封装，依赖rclpy<br />例如 galactic\src\ros2\ros2cli\ros2topic\ros2topic\api\__init__.py

```python
import argparse

from argparse import ArgumentTypeError
from time import sleep

import rclpy

from rclpy.expand_topic_name import expand_topic_name
from rclpy.topic_or_service_is_hidden import topic_or_service_is_hidden
from rclpy.validate_full_topic_name import validate_full_topic_name
from ros2cli.node.strategy import NodeStrategy
from rosidl_runtime_py import get_message_interfaces
from rosidl_runtime_py import message_to_yaml
from rosidl_runtime_py.utilities import get_message
```

galactic\src\ros2\ros2cli\ros2cli\ros2cli\node\strategy.py

```python
from ros2cli.node.daemon import add_arguments as add_daemon_node_arguments
from ros2cli.node.daemon import DaemonNode
from ros2cli.node.daemon import is_daemon_running
from ros2cli.node.daemon import spawn_daemon
from ros2cli.node.direct import add_arguments as add_direct_node_arguments
from ros2cli.node.direct import DirectNode
```

## rosidl_runtime_py

rosidl消息系统的python接口，与通信管理关系不大，这个主要是data-exchange-format的领域

# 哪些模块会依赖daemon

## ros2cli的node相关命令

使用到node的命令，通过node_strategy来设置是否使用daemon<br />galactic\src\ros2\ros2cli\ros2cli\ros2cli\node\strategy.py<br />[https://github.com/ros2/ros2cli/pull/514](https://github.com/ros2/ros2cli/pull/514)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1668563607673-c28d5c10-9020-45ec-906d-df43226187ac.png#averageHue=%23fefdfd&clientId=u108a1875-5280-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=201&id=uf031bc76&margin=%5Bobject%20Object%5D&name=image.png&originHeight=201&originWidth=295&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7790&status=done&style=none&taskId=u636c6753-05d1-4627-a97a-0bdaad370fb&title=&width=295)

## ros2-lifecycle

会报错node not found

# 不依赖daemon

## rclcpp

rviz2不会依赖daemon

## rqt-plugin

只会依赖rclpy，所以应该也不会出现这个问题

# 参考

[https://answers.ros.org/question/377666/how-to-list-the-ros2-topics-from-a-c-node/](https://answers.ros.org/question/377666/how-to-list-the-ros2-topics-from-a-c-node/)
