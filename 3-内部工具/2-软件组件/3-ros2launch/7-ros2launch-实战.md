如何利用ros2launch做

- 节点启动
- 节点监控和管理

# ros2cli还是ros2launch

所有管理类命令都可以通过execute_process来执行ros2cli来做到，但是这个是最后的办法<br />尽量在ros2launch库内解决问题

```python
from launch.substitutions import FindExecutable
from launch.actions import ExecuteProcess

...

ld.add_action(
    ExecuteProcess(
        cmd=[[
            FindExecutable(name='ros2'),
            " service call ",
            "/namespace/service_to_call ",
            "example_msgs/srv/ExampleMsg ",
            '"{param_1: True, param_2: 0.0}"',
        ]],
        shell=True
    )
)
```

# 普通节点

## 退出错误处理

[https://ubuntu.com/blog/ros2-launch-required-nodes](https://ubuntu.com/blog/ros2-launch-required-nodes)<br />利用事件机制

# lifecycle_node

## 启动

[https://answers.ros.org/question/304370/ros2-launch-how-to-correctly-create-a-lifecycle-launch-file/](https://answers.ros.org/question/304370/ros2-launch-how-to-correctly-create-a-lifecycle-launch-file/)<br />[https://docs.ros.org/en/dashing/Tutorials/Launch-system.html](https://docs.ros.org/en/dashing/Tutorials/Launch-system.html)<br />The launch file in [this example](https://github.com/ros2/launch_ros/blob/dashing/launch_ros/examples/lifecycle_pub_sub_launch.py) launches two nodes, one of which is a node with a [managed lifecycle](https://docs.ros.org/en/dashing/Tutorials/Managed-Nodes.html) (a “lifecycle node”). Lifecycle nodes launched through launch_ros automatically emit _events_ when they transition between states. The events can then be acted on through the launch framework. For example, by emitting other events (such as requesting another state transition, which lifecycle nodes launched through launch_ros automatically have event handlers for) or triggering other _actions_ (e.g. starting another node).<br />In the aforementioned example, various transition requests are requested of the talker lifecycle node, and its transition events are reacted to by, for example, launching a listener node when the lifecycle talker reaches the appropriate state.

```python
"""Launch a lifecycle ZED node and the Robot State Publisher"""

import os


import launch
from launch import LaunchIntrospector

from ament_index_python.packages import get_package_share_directory

from launch import LaunchDescription
from launch.actions import EmitEvent
from launch.actions import LogInfo
from launch.actions import RegisterEventHandler
from launch_ros.actions import Node
from launch_ros.actions import LifecycleNode
from launch_ros.events.lifecycle import ChangeState
from launch_ros.event_handlers import OnStateTransition

import lifecycle_msgs.msg

def generate_launch_description():

    # use: 'zed' for "ZED" camera - 'zedm' for "ZED mini" camera
    camera_model = 'zedm'

    # URDF file to be loaded by Robot State Publisher
    urdf = os.path.join(get_package_share_directory('stereolabs_zed'), 'urdf', camera_model + '.urdf')

    # ZED Configurations to be loaded by ZED Node
    config_common = os.path.join(get_package_share_directory('stereolabs_zed'), 'config', 'common.yaml')

    config_camera = os.path.join(get_package_share_directory('stereolabs_zed'), 'config', camera_model + '.yaml')

    # Set LOG format
    os.environ['RCUTILS_CONSOLE_OUTPUT_FORMAT'] = '{time}: [{name}] [{severity}]\t{message}'

    # Launch Description
    ld = launch.LaunchDescription()

    # Prepare the ZED node
    zed_node = LifecycleNode(
        node_namespace = 'zed',        # must match the namespace in config -> YAML
        node_name = 'zed_node',        # must match the node name in config -> YAML
        package = 'stereolabs_zed',
        node_executable = 'zed_wrapper_node',
        output = 'screen',
        parameters = [
            config_common,  # Common parameters
            config_camera,  # Camera related parameters
        ]
    )

    # Prepare the Robot State Publisher node
    rsp_node = Node(
        node_name = 'zed_state_publisher',
        package = 'robot_state_publisher',
        node_executable = 'robot_state_publisher',
        output = 'screen',
        arguments = [urdf, 'robot_description:=zed_description']
    )

    # Make the ZED node take the 'configure' transition
    zed_configure_trans_event = EmitEvent(
        event=ChangeState(
            lifecycle_node_matcher = launch.events.process.matches_action(zed_node),
            transition_id = lifecycle_msgs.msg.Transition.TRANSITION_CONFIGURE,
        )
    )

    # Make the ZED node take the 'activate' transition
    zed_activate_trans_event = EmitEvent(
        event = ChangeState(
            lifecycle_node_matcher = launch.events.process.matches_action(zed_node),
            transition_id = lifecycle_msgs.msg.Transition.TRANSITION_ACTIVATE,
         )
    )

    # When the ZED node reaches the 'inactive' state, make it take the 'activate' transition and start the Robot State Publisher
    zed_inactive_state_handler = RegisterEventHandler(
        OnStateTransition(
            target_lifecycle_node = zed_node,
            goal_state = 'inactive',
            entities = [
                # Log
                LogInfo( msg = "'ZED' reached the 'INACTIVE' state, start the 'Robot State Publisher' node and 'activating'." ),
                # Robot State Publisher
                rsp_node,
                # Change State event ( inactive -> active )
                zed_activate_trans_event,
            ],
        )
    )

    # When the ZED node reaches the 'active' state, log a message.
    zed_active_state_handler = RegisterEventHandler(
        OnStateTransition(
            target_lifecycle_node = zed_node,
            goal_state = 'active',
            entities = [
                # Log
                LogInfo( msg = "'ZED' reached the 'ACTIVE' state" ),
            ],
        )
    )

    # Add the actions to the launch description.
    # The order they are added reflects the order in which they will be executed.
    ld.add_action( zed_inactive_state_handler )
    ld.add_action( zed_active_state_handler )
    ld.add_action( zed_node )
    ld.add_action( zed_configure_trans_event)

    return ld
```

## 运行节点管理

[https://answers.ros.org/question/394608/event-handling-on-ros2-nodes-across-launch-files/](https://answers.ros.org/question/394608/event-handling-on-ros2-nodes-across-launch-files/)

```python
# When the talker node reaches the 'active' state, log a message and start the listener node.
register_event_handler_for_talker_reaches_active_state = launch.actions.RegisterEventHandler(
    launch_ros.event_handlers.OnStateTransition(
        target_lifecycle_node=talker_node, goal_state='active',
        entities=[
            launch.actions.LogInfo(
                msg="node 'talker' reached the 'active' state, launching 'listener'."),
            launch_ros.actions.LifecycleNode(
                name='listener', namespace='',
                package='lifecycle', executable='lifecycle_listener', output='screen'),
        ],
    )
) # Copyright 2018 Open Source Robotics Foundation, Inc. See original source at link for Apache 2 license
```

事件只能在一个launch范围内进行响应和发射<br />对于已经运行的节点，如何管理。。。<br />可能只能使用ros2 cli

# component

## 新建

```python
import launch
from launch_ros.actions import ComposableNodeContainer
from launch_ros.descriptions import ComposableNode


def generate_launch_description():
    """Generate launch description with multiple components."""
    container = ComposableNodeContainer(
        name='image_container',
        namespace='',
        package='rclcpp_components',
        executable='component_container',
        composable_node_descriptions=[
            ComposableNode(
                package='image_tools',
                plugin='image_tools::Cam2Image',
                name='cam2image',
                remappings=[('/image', '/burgerimage')],
                parameters=[{'width': 320, 'height': 240, 'burger_mode': True, 'history': 'keep_last'}],
                extra_arguments=[{'use_intra_process_comms': True}]),
            ComposableNode(
                package='image_tools',
                plugin='image_tools::ShowImage',
                name='showimage',
                remappings=[('/image', '/burgerimage')],
                parameters=[{'history': 'keep_last'}],
                extra_arguments=[{'use_intra_process_comms': True}])
        ],
        output='both',
    )

    return launch.LaunchDescription([container])
```

## load existing

```python
from launch import LaunchDescription
from launch_ros.actions import LoadComposableNodes, Node
from launch_ros.descriptions import ComposableNode

def generate_launch_description():
    container = Node(
        name='image_container',
        package='rclcpp_components',
        executable='component_container',
        output='both',
    )

    load_composable_nodes = LoadComposableNodes(
        target_container='image_container',
        composable_node_descriptions=[
            ComposableNode(
                 package='image_tools',
                plugin='image_tools::Cam2Image',
                name='cam2image',
                remappings=[('/image', '/burgerimage')],
                parameters=[{'width': 320, 'height': 240, 'burger_mode': True, 'history': 'keep_last'}],
                extra_arguments=[{'use_intra_process_comms': True}],
            ),
            ComposableNode(
                package='image_tools',
                plugin='image_tools::ShowImage',
                name='showimage',
                remappings=[('/image', '/burgerimage')],
                parameters=[{'history': 'keep_last'}],
                extra_arguments=[{'use_intra_process_comms': True}]
            ),
        ],
    )

    return LaunchDescription([container, load_composable_nodes])
```

# component-intra

I don't remember how to set up intra-process communication when using components/containers in launch files (maybe [@hidmic](https://github.com/hidmic) knows better).<br />Currently, if you manually compose nodes in one executable (as you're currently doing), there's no nice way to configure it in a launch file. There's a proposed refactor considering that, between other things, [ros2/launch#114](https://github.com/ros2/launch/issues/114).<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1667884505216-486db064-35c1-4b4f-9398-652a3478a849.png#averageHue=%23e9c897&clientId=u79ef8859-142e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=232&id=F4HSn&margin=%5Bobject%20Object%5D&name=image.png&originHeight=232&originWidth=890&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37426&status=done&style=none&taskId=ue48e79fa-b487-4e42-8eb0-c8b20a3256f&title=&width=890)

```python
container = ComposableNodeContainer(
            name='apa_container',
            namespace='',
            package='rclcpp_components',
            executable='component_container_mt',
            prefix='taskset 3f',
            composable_node_descriptions=[
                ComposableNode(
                    package='camera_service',
                    plugin='cam::LifecycleParking',
                    name='camera_parking_lifecycle',
                    extra_arguments=[{'use_intra_process_comms': True}]
                )
                ,
                ComposableNode(
                    package='apa_perception_fusion_lifecycle_component',
                    plugin='apa::PercetionFusionComponent',
                    extra_arguments=[{'use_intra_process_comms': True}]
                )
            ],
            output='screen',
    )
```
