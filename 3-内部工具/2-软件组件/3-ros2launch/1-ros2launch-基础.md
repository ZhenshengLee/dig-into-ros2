# 概述

ros2launch相对于ros1而言有很多改进点：提升解释力，强调使用事件来驱动行为，验证和确定性的启动<br />Python API（.launch.py）

```
# package://demo_nodes_cpp/launch/talker_listener.launch.py
$ # can be introspected with:
$ ros2 launch --print-description demo_nodes_cpp talker_listener.launch.py
$ # can be run with:
$ ros2 launch demo_nodes_cpp talker_listener.launch.py
```

# 特性

## 最简例子

定义一个def generate_launch_description()函数<br />return LaunchDescription 启动描述，参数是一个数组，每个数组是一个启动项<br />启动项格式：

```python
ExecuteNodeProcess(package='demo_nodes_cpp', node_executable='talker', output='screen',
                   node_description=NodeDescription(name='my_talker'))
```

如果一个可执行文件中设计了四个节点，则可以在一个进程中启动两个

```python
ExecuteMultiNodeProcess(package='composition', node_executable='manual_composition', output='screen', node_descriptions={
    'talker': NodeDescription(name='my_talker'),
    'listener': NodeDescription(name='my_listener'),
})
```

## substitution

替换主要是用在参数上，提供一些变量，在描述语言中分析。<br />这些变量可以是：启动配置，环境变量，表达式计算

```python
from launch_ros.substitutions import FindPackageShare

from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import PathJoinSubstitution, TextSubstitution


def generate_launch_description():
    colors = {
        'background_r': '200'
    }

    return LaunchDescription([
        IncludeLaunchDescription(
            PythonLaunchDescriptionSource([
                PathJoinSubstitution([
                    FindPackageShare('launch_tutorial'),
                    'example_substitutions.launch.py'
                ])
            ]),
            launch_arguments={
                'turtlesim_ns': 'turtlesim2',
                'use_provided_red': 'True',
                'new_background_r': TextSubstitution(text=str(colors['background_r']))
            }.items()
        )
    ])
```

```python
from launch.substitutions import LaunchConfiguration, PythonExpression

from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, ExecuteProcess, TimerAction
from launch.conditions import IfCondition
from launch.substitutions import LaunchConfiguration, PythonExpression


def generate_launch_description():
    turtlesim_ns = LaunchConfiguration('turtlesim_ns')
    use_provided_red = LaunchConfiguration('use_provided_red')
    new_background_r = LaunchConfiguration('new_background_r')

    turtlesim_ns_launch_arg = DeclareLaunchArgument(
        'turtlesim_ns',
        default_value='turtlesim1'
    )
```

## 带生命周期的节点

先创建一个

```python
talker_node = LifecycleNodeDescription(node_name='talker')
execute_talker_process_action = ExecuteNodeProcess(
    package='lifecycle', node_executable='lifecycle_talker', output='screen',
    node_description=talker_node)
```

## 事件处理器

launch系统是ros2启动和管理用户进程的系统，启动系统负责监控所启动进程的状态，在进程报告其状态改变时做出响应的处理，这些变更会触发一些状态，launch系统中定义对应的事件处理器。事件处理器可以用来监控进程的状态，可以定义复杂的规则来动态修改Launch文件。

```python
return LaunchDescription([
        turtlesim_ns_launch_arg,
        use_provided_red_launch_arg,
        new_background_r_launch_arg,
        turtlesim_node,
        RegisterEventHandler(
            OnProcessStart(
                target_action=turtlesim_node,
                on_start=[
                    LogInfo(msg='Turtlesim started, spawning turtle'),
                    spawn_turtle
                ]
            )
        ),
])
# 对于具体的action，注册其事件的处理器，目前的处理器

register_event_handler_for_talker_reaches_active_state = RegisterEventHandler(
    OnStateTransition(
        target_lifecycle_node=talker_node, goal_state='active',
        entities=[
            LogInfo(
                msg="node 'talker' reached the 'active' state, launching 'listener'."),
            ExecuteNodeProcess(
                package='lifecycle', node_executable='lifecycle_listener', output='screen',
                node_description=LifecycleNodeDescription(node_name='listener')),
        ],
    )
)
```

# 实践

写一个launch<br />必须以.launch.py结尾，<br />Your launch file should define the generate_launch_description() which returns a launch.LaunchDescription() to be used by the ros2 launch verb

## 最简单的launch

```cpp
from launch import LaunchDescription
from launch_ros.actions import Node
def generate_launch_description():
    return LaunchDescription([
        Node(
            package='turtlesim',
            node_namespace='turtlesim1',
            node_executable='turtlesim_node',
            node_name='sim'
        ),
        Node(
            package='turtlesim',
            node_namespace='turtlesim2',
            node_executable='turtlesim_node',
            node_name='sim'
        ),
        Node(
            package='turtlesim',
            node_executable='mimic',
            node_name='mimic',
            remappings=[
                ('/input/pose', '/turtlesim1/turtle1/pose'),
                ('/output/cmd_vel', '/turtlesim2/turtle1/cmd_vel'),
            ]
        )
    ])
```

node_space，允许运行多个相同的节点，并且主题名字不会发生冲突。两个节点都接受和发送相同的节点<br />launch的启动

```
ros2 launch turtlesim_mimic_launch.py
ros2 launch <package_name> <launch_file_name>
```

# launch调试

ros2 launch -d raise.launch.py<br />ros1到ros2的移植<br />launch system<br />ros2的launch system就是帮助用户描述系统配置并且按照描述执行系统。launch系统的配置包括运行何种程序，程序在哪儿，传递何种参数，并且按照ROS的准则复用ROS组件。<br />也可以检测进程运行的状态，上报并根据状态进行对应的反应。<br />ROS2-B版本发布了基于python的启动文件，可以根据特定事件启动和停止不同的节点。<br />launch文件监视多个节点状态

# 常见问题

## 找不到包

cmake 和package.xml改写好<br />找不到launch文件<br />修改launch之后需要build，<br />build之后需要source install/setup.bash<br />记得在cmakefile里面install launch files，启动时候是在install目录中找的

```python
from glob import glob
from setuptools import setup
package_name = 'my_package'
setup(
    # Other parameters ...
    data_files=[
        # ... Other data files
        # Include all launch files. This is the most important line here!
        (os.path.join('share', package_name), glob('launch/*.launch.py'))
    ]
)
```

cpp 包

```cmake
install(DIRECTORY
  launch
  DESTINATION share/${PROJECT_NAME}/
)
```

# lifecycle node

managed node，通过launch_ros自动启动，在状态切换的时候发射一个事件。事件可以在launch文件中被响应，比如触发另一个事件（比如切换到另一个状态，这个是通过事件处理者决定的），或者触发另外的行为，如启动新的节点。<br />向节点传递参数<br />加载参数

```
ros2 param dump <node_name>
ros2 run <package_name> <executable_name> --ros-args --params-file <file_name>
ros2 run turtlesim turtlesim_node --ros-args --params-file ./turtlesim.yaml
```

```
ros2 run my_package node_executable --ros-args ...
# name remapping
ros2 run demo_nodes_cpp talker --ros-args -r __ns:=/demo -r __node:=my_talker -r chatter:=my_topic
ros2 run package_name executable_name --ros-args -p param_name:=param_value
```

# 参考

[https://roscon.ros.org/2018/presentations/ROSCon2018_launch.pdf](https://roscon.ros.org/2018/presentations/ROSCon2018_launch.pdf)<br />[https://design.ros2.org/articles/roslaunch.html](https://design.ros2.org/articles/roslaunch.html)<br />[https://docs.ros.org/en/foxy/Tutorials/Intermediate/Launch/Creating-Launch-Files.html](https://docs.ros.org/en/foxy/Tutorials/Intermediate/Launch/Creating-Launch-Files.html)
