# 单进程多节点

设计成component，方便单节点和进程内节点配置

## 参考

[https://index.ros.org/doc/ros2/Tutorials/Composition/](https://index.ros.org/doc/ros2/Tutorials/Composition/)

<http://docs.ros.org/en/rolling/Concepts/About-Composition.html>

## ROS1和ROS2的区别

ROS1主要是node 和 nodelet，node编译为可执行文件，nodelet编译成动态连接库，在运行时由一个容器进程动态加载

ROS2设计了统一的api，推荐的设计模式跟nodelet相似，我们称之为component。这样使得添加一些概念到代码中变得容易，比如life cycle。

统一的API也为编程带来了很大便利！

注意：

当然还是可以像之前的node风格进行代码设计，不过已经不推荐了。

## 进程设计process layouts

开发者可以选择：

- 多进程多节点

- 优势：各个模块独立处理，容错性好，易用，方便测试

- 单进程多节点

- 优势：数据传输开销小

## 编写一个组件

注意：组件没有main函数，组件时rclcpp::Node的子类，组件不驻留在线程上，所以在构造函数中不能执行长时间运行或阻塞的任务，不过组件可以使用定时器通知，创建发布器，订阅器等

定义了动态库后，需要由一个注册行为，使用rcl_components中的功能。这样使得组件可见，能够被进程加载，类似于一个entry point

组件定义后，需要在cmake中注册，不然资源管理系统不能发现文件

```

add_library(talker_component SHARED

   src/talker_component.cpp)

rclcpp_components_register_nodes(talker_component "composition::Talker")

# To register multiple components in the same shared library, use multiple calls

# rclcpp_components_register_nodes(talker_component "composition::Talker2")

```

为了能够搜索到这个动态库，必须source 工作环境

## 使用组件

有三种常见的使用方式

- 启动一个进程，请求load_node服务，提供包名，动态库名字，ros 服务服务端会加载对应的组件

- 这种模式下，从命令行请求服务也是可以的

- 创建一个可执行文件包含多个节点，这种方法需要组件在定义过程中包含头文件

- 通过ros2 launch创建一个容器进程来启动多个节点

### 发现已有的组件

```

$ ros2 component types

composition

  composition::Talker

  composition::Listener

  composition::Server

  composition::Client

```

### 方式1

```

ros2 run rclcpp_components component_container

```

```

ros2 component list

/ComponentManager

```

```

ros2 component load /ComponentManager composition composition::Talker

Loaded component 1 into '/ComponentManager' container node as '/talker'

# Now the first shell should show a message that the component was loaded as well as repeated message for publishing a message.

```

```

$ ros2 component load /ComponentManager composition composition::Listener

Loaded component 2 into '/ComponentManager' container node as '/listener'

```

### 方式2

方式2的方式和进程内部通信机制是一样的方式，这种方式比较简单

```

ros2 run composition manual_composition

```

Manually-composed components will not be reflected in the ros2 component list command line tool output.

这种是裸的，本质上不是组件，不会在组件列表中显示出来

### 方式3

```

ros2 launch composition composition_demo.launch.py

```

```py

"""Launch a talker and a listener in a component container."""



import launch

from launch_ros.actions import ComposableNodeContainer

from launch_ros.descriptions import ComposableNode





def generate_launch_description():

    """Generate launch description with multiple components."""

    container = ComposableNodeContainer(

            name='my_container',

            namespace='',

            package='rclcpp_components',

            executable='component_container',

            composable_node_descriptions=[

                ComposableNode(

                    package='composition',

                    plugin='composition::Talker',

                    name='talker'),

                ComposableNode(

                    package='composition',

                    plugin='composition::Listener',

                    name='listener')

            ],

            output='screen',

    )



    return launch.LaunchDescription([container])

```

### 额外方式：使用dlopen

Run-time composition using dlopen

This demo presents an alternative to 1. by creating a generic container process and explicitly passing the libraries to load without using ROS interfaces. The process will open each library and create one instance of each “rclcpp::Node” class in the library source code).

加载一个动态库，创建一个rclcpp::Node

这种是裸的，本质上不是组件，不会在组件列表中显示出来

## 高级话题

### 卸载组件

```

ros2 run rclcpp_components component_container

$ ros2 component load /ComponentManager composition composition::Talker

Loaded component 1 into '/ComponentManager' container node as '/talker'

$ ros2 component load /ComponentManager composition composition::Listener

Loaded component 2 into '/ComponentManager' container node as '/listener'

$ ros2 component unload /ComponentManager 1 2

Unloaded component 1 from '/ComponentManager' container

Unloaded component 2 from '/ComponentManager' container

```

### 重映射容器名和容器空间

```

ros2 run rclcpp_components component_container --ros-args -r __node:=MyContainer -r __ns:=/ns

```

### 重映射组件名和组件空间

```

ros2 run rclcpp_components component_container

# Remap node name

ros2 component load /ComponentManager composition composition::Talker --node-name talker2

# Remap namespace

ros2 component load /ComponentManager composition composition::Talker --node-namespace /ns

# Remap both

ros2 component load /ComponentManager composition composition::Talker --node-name talker3 --node-namespace /ns2

```
