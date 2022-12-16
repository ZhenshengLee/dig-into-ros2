# launch.action

## 简单风格

```python
from launch import LaunchDescription
from launch_ros.actions import Node


def generate_launch_description():
    return LaunchDescription([
        Node(
            package="sm_guardian",
            executable="sm_guardian_node",
            arguments=['--ros-args', '--log-level', 'warn'],
            output="screen"
        )]
    )
```

定义一个generate_launch_description函数，这是一个统一入口

## 复杂

IncludeLaunchDescription,  DeclareLaunchArgument<br />src\launch\autoware_demos\launch\avp_core.launch.py

```python
from ament_index_python import get_package_share_directory
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.actions import IncludeLaunchDescription
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.conditions import IfCondition
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import Node

prediction = Node(
        executable='prediction_nodes_node_exe',
        name='prediction',
        namespace='prediction',
        output="screen",
        package='prediction_nodes',
        parameters=[LaunchConfiguration('prediction_param_file')],
        remappings=[
            ("tracked_objects", "/perception/tracked_objects")
        ],
        condition=IfCondition(LaunchConfiguration('with_obstacles'))
    )

    return LaunchDescription([
        euclidean_cluster_param,
        ray_ground_classifier_param,
        scan_downsampler_param,
        with_obstacles_param,
        lanelet2_map_provider_param,
        lane_planner_param,
        costmap_generator_param,
        freespace_planner_param,
        object_collision_estimator_param,
        behavior_planner_param,
        off_map_obstacles_filter_param,
        vehicle_characteristics_param,
        vehicle_constants_manager_param,
        euclidean_clustering,
        ray_ground_classifier,
        scan_downsampler,
        point_cloud_fusion_node,
        lanelet2_map_provider,
        lanelet2_map_visualizer,
        global_planner,
        lane_planner,
        costmap_generator,
        freespace_planner,
        object_collision_estimator,
        behavior_planner,
        off_map_obstacles_filter,
        multi_object_tracker_param,
        multi_object_tracker,
        state_estimation_param,
        state_estimation,
        covariance_insertion_param,
        covariance_insertion,
        prediction_param,
        prediction
    ])
```

## OpaqueFunction

```python
from launch.actions import (
    OpaqueFunction,
    DeclareLaunchArgument,
    IncludeLaunchDescription,
)
```

## group_action

nav2_bringup\launch\bringup_launch.py

```python
from launch.actions import (DeclareLaunchArgument, GroupAction,
                            IncludeLaunchDescription, SetEnvironmentVariable)
```

# SetParameter

Add a SetParameter action that sets a parameter to all nodes in the same scope<br />对当前作用域声明的node节点有用

# launch.substitutions

[https://github.com/Serafadam/interbotix_ros_manipulators/blob/xsarm_control_galactic/interbotix_ros_xsarms/interbotix_xsarm_control/launch/xsarm_control.launch.py](https://github.com/Serafadam/interbotix_ros_manipulators/blob/xsarm_control_galactic/interbotix_ros_xsarms/interbotix_xsarm_control/launch/xsarm_control.launch.py#L31)

```python
from launch.substitutions import (
    Command,
    FindExecutable,
    LaunchConfiguration,
    PathJoinSubstitution,
    TextSubstitution,
)
```

# launch_description

## add_action

nav2_bringup\launch\bringup_launch.py
