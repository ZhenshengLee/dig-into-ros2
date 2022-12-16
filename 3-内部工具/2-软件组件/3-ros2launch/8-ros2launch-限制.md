# component-lifecycle

```python
drive_base_inactive_state_handler = RegisterEventHandler(
    OnStateTransition(
        target_lifecycle_node = drive_base_node_action,
        goal_state = 'inactive',
        entities = [
            # Log
            LogInfo( msg = "'drive_base_name' reached the 'INACTIVE' state, 'activating'." ),
            # Change State event ( inactive -> active )
            drive_base_activate_trans_event_by_name,
        ],
    )
)
```

component-lifecycle，不是普通的lifecycle node，而是CompososedNodeContainer，继承自Node，所以无法使用上述的RegisterEventHandler<br />[https://github.com/ros2/launch_ros/issues/41](https://github.com/ros2/launch_ros/issues/41)  社区已经有issue<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1667887114018-2a0f22fb-7250-4cdf-96ae-d3c8bd154344.png#averageHue=%23fdfcfa&clientId=ud517fdcb-b7b3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=403&id=ucff35f8c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=403&originWidth=908&originalType=binary&ratio=1&rotation=0&showTitle=false&size=52905&status=done&style=none&taskId=u50ce88f8-55fa-49d8-bada-87dd217d916&title=&width=908)<br />目前有自定义的解决方案，但是没有合入主分支<br />当前有一个临时方案

```python
import launch
import launch_ros
from launch_ros.actions import ComposableNodeContainer
from launch_ros.descriptions import ComposableNode

from launch.actions import EmitEvent
from launch.actions import LogInfo
from launch.actions import RegisterEventHandler

from launch.substitutions import FindExecutable
from launch.actions import ExecuteProcess

from launch.event_handlers import (OnExecutionComplete, OnProcessStart)

def generate_launch_description():
    """Generate launch description with multiple components."""
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

    perc_base_node = " apa_perception_fusion_lifecycle_component "
    camera_base_node = " camera_parking_lifecycle "
    lifecycle_configure = " configure "

    # Launch Description
    configure_perc_base = ExecuteProcess(
        cmd=[[
            FindExecutable(name='ros2'),
            " lifecycle set ",
            perc_base_node,
            lifecycle_configure,
        ]],
        shell=True
    )

    configure_camera_base = ExecuteProcess(
        cmd=[[
            FindExecutable(name='ros2'),
            " lifecycle set ",
            camera_base_node,
            lifecycle_configure,
        ]],
        shell=True
    )

    sleep_shell = ExecuteProcess(
        cmd=[[
            FindExecutable(name='sleep'),
            " 2 ",
        ]],
        shell=True
    )


    ld = launch.LaunchDescription([
        container,
        RegisterEventHandler(
            OnProcessStart(
                target_action=container,
                on_start=[
                    LogInfo(msg='sleep 2s, waitting component_container_mt to finish ComposableNode load'),
                    sleep_shell
                ]
            )
        ),
        RegisterEventHandler(
            OnExecutionComplete(
                target_action=sleep_shell,
                on_completion=[
                    configure_perc_base,
                    LogInfo(msg='apa_perception_fusion_lifecycle_component configure finished')
                ]
            )
        ),
        RegisterEventHandler(
            OnExecutionComplete(
                target_action=sleep_shell,
                on_completion=[
                    configure_camera_base,
                    LogInfo(msg='camera_parking_lifecycle configure finished')
                ]
            )
        )
    ])

    return ld
```

# launch传递node-options

不行<br />[https://github.com/ros2/launch_ros/issues/84](https://github.com/ros2/launch_ros/issues/84)

# use dedicated executor for component nodes

[https://github.com/ros2/rclcpp/issues/1774](https://github.com/ros2/rclcpp/issues/1774)

# 参考

[https://github.com/ros2/launch/issues/114](https://github.com/ros2/launch/issues/114)
