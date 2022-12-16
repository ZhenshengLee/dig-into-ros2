# 概述

Currently nodes can be written as [components](https://index.ros.org//doc/ros2/Tutorials/Composition/), and components can be configured and loaded in a container. You can do all that in a launch file (see for example [https://github.com/ros2/demos/blob/master/composition/launch/composition_demo.launch.py](https://github.com/ros2/demos/blob/master/composition/launch/composition_demo.launch.py)).<br />component-manager从executor中拿到节点进行管理

# with-lifecycle

是支持的，按照正常的lifecycle_node使用即可

# with-intra

## manual

i believe that you can do what you want as followings,

1. implement nodes with options.use_intra_process_comms(true)
2. compose these nodes as [compositions](https://index.ros.org//doc/ros2/Tutorials/Composition/).
3. ros2 launch your_package your_launch_file

if im missing something, let me know.<br />here are my examples, (not using lifecycle node)<br />[https://github.com/fujitatomoya/demos/blob/topic-20191007-composition-intra-launch/composition/launch/composition_demo.launch.py](https://github.com/fujitatomoya/demos/blob/topic-20191007-composition-intra-launch/composition/launch/composition_demo.launch.py)<br />[fujitatomoya/demos@664e73d](https://github.com/fujitatomoya/demos/commit/664e73db4e16fa7abf1bb2ea44ffb8e696b070e8)<br />[https://github.com/fujitatomoya/demos/blob/topic-20191007-composition-intra-launch/composition/launch/composition_demo.launch.py](https://github.com/fujitatomoya/demos/blob/topic-20191007-composition-intra-launch/composition/launch/composition_demo.launch.py)

## launch

[https://github.com/ros2/rclcpp/issues/880](https://github.com/ros2/rclcpp/issues/880)<br />通过LoadNode extra_arguments<br />use_intra_process_comms' boolean in extra_arguments.<br />[https://github.com/ros2/launch_ros/blob/rolling/launch_ros/launch_ros/descriptions/composable_node.py#L39-L60](https://github.com/ros2/launch_ros/blob/rolling/launch_ros/launch_ros/descriptions/composable_node.py#L39-L60)<br />[https://github.com/ros2/rclcpp/pull/871](https://github.com/ros2/rclcpp/pull/871)

# component-manager-with-executors

默认情况：

- （compile time compositon）manual composition可以使用各种executor
- runtime compositon

rclcpp支持for Run-time composition , i notice that rclcpp_components only provides component_container which use SingleThreadedExecutor for all component nodes in container and component_container_mt which use MultiThreadedExecutor.<br />2021-12合入到master分支的新功能支持a bunch of SinglethreadedExecutor for component_container<br />[https://github.com/ros2/rclcpp/issues/1774](https://github.com/ros2/rclcpp/issues/1774)<br />[https://github.com/ros2/rclcpp/pull/1781/files?w=1](https://github.com/ros2/rclcpp/pull/1781/files?w=1)

```cpp
int main(int argc, char * argv[])
{
    /// Component container with dedicated single-threaded executors for each components.
    rclcpp::init(argc, argv);
    // parse arguments
    bool use_multi_threaded_executor{false};
    std::vector<std::string> args = rclcpp::remove_ros_arguments(argc, argv);
    for (auto & arg : args) {
        if (arg == std::string("--use_multi_threaded_executor")) {
            use_multi_threaded_executor = true;
        }
    }
    // create executor and component manager
    auto exec = std::make_shared<rclcpp::executors::SingleThreadedExecutor>();
    rclcpp::Node::SharedPtr node;
    if (use_multi_threaded_executor) {
        using ComponentManagerIsolated =
        rclcpp_components::ComponentManagerIsolated<rclcpp::executors::MultiThreadedExecutor>;
        node = std::make_shared<ComponentManagerIsolated>(exec);
    } else {
        using ComponentManagerIsolated =
        rclcpp_components::ComponentManagerIsolated<rclcpp::executors::SingleThreadedExecutor>;
        node = std::make_shared<ComponentManagerIsolated>(exec);
    }
    exec->add_node(node);
    exec->spin();
}
```

# with-launch-events

[https://github.com/ros2/launch/issues/672](https://github.com/ros2/launch/issues/672)

# 参考

[https://docs.ros.org/en/foxy/Tutorials/Demos/Intra-Process-Communication.html](https://docs.ros.org/en/foxy/Tutorials/Demos/Intra-Process-Communication.html)<br />[https://navigation.ros.org/2021summerOfCode/projects/spinners.html](https://navigation.ros.org/2021summerOfCode/projects/spinners.html)
