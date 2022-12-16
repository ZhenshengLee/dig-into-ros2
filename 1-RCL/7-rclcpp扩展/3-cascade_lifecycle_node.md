# 参考

[https://index.ros.org/r/cascade_lifecycle/](https://index.ros.org/r/cascade_lifecycle/)<br />[https://github.com/fmrico/cascade_lifecycle](https://github.com/fmrico/cascade_lifecycle)

# cascade_lifecycle

[Managed nodes](https://design.ros2.org/articles/node_lifecycle.html) (or lifecycle nodes, LN) are an extremely useful concept in ROS2. It provides a mechanism to define states in a node so that its life cycle can be better controlled.<br />When an application is made up of multiple LNs, it is common to use a node to orchestrate the transitions of each one. This occurs, for example, in [Navigation2](https://github.com/ros-planning/navigation2/tree/master/nav2_lifecycle_manager) or in [ROS2 Planning System](https://github.com/IntelligentRoboticsLabs/ros2_planning_system/tree/master/plansys2_lifecycle_manager).<br />cascade_lifecycle provides a mechanism that can make managing LNs easier. This idea is based on my developments with [BICA](https://github.com/IntelligentRoboticsLabs/BICA/tree/ros2). This mechanism allows defining dependencies between LNs. When an LN A establishes an LN B as a dependency, when an A enters a state, B automatically enters this state. This allows creating configuration/activation/deactivation trees.<br />The class rclcpp_cascade_lifecycle::CascadeLifecycleNode extends the rclcpp_lifecycle::LifecycleNode API with next operations:<br />void add_activation (const std::string & node_name); void remove_activation (const std::string & node_name); void clear_activation (); <br />Using rclcpp_cascade_lifecycle in the next example, node_b makes the same state transitions as node_a:

```cpp
auto node_a = std::make_shared<rclcpp_cascade_lifecycle::CascadeLifecycleNode>("node_A");
auto node_b = std::make_shared<rclcpp_cascade_lifecycle::CascadeLifecycleNode>("node_B");

rclcpp::executors::SingleThreadedExecutor executor;
executor.add_node(node_a->get_node_base_interface());
executor.add_node(node_b->get_node_base_interface());

node_a->add_activation("node_B");
node_a->trigger_transition(lifecycle_msgs::msg::Transition::TRANSITION_CONFIGURE);
{
    rclcpp::Rate rate(10);
    auto start = node_a->now();
    while ((node_a->now() - start).seconds() < 0.5) {
        executor.spin_some();
        rate.sleep();
    }
}

ASSERT_EQ(node_a->get_current_state().id(), lifecycle_msgs::msg::State::PRIMARY_STATE_INACTIVE);
ASSERT_EQ(node_b->get_current_state().id(), lifecycle_msgs::msg::State::PRIMARY_STATE_INACTIVE);
```

Hope it helps!!!
