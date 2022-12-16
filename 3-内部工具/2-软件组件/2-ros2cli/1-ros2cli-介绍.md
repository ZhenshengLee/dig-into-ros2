## 参考

[https://github.com/ros2/ros2cli](https://github.com/ros2/ros2cli)

<https://github.com/ros2/rclpy>

<https://docs.ros2.org/latest/api/rcl/>

## 流程

ros2cli->rclpy->RCL->RMW

## 中间件开发以支持ros2cli

### ros2 node list

<https://github.com/ros2/ros2cli/blob/master/ros2node/ros2node/verb/list.py>

list-->

<https://github.com/ros2/ros2cli/blob/master/ros2node/ros2node/api/>__init__.py

get_node_names_and_namespaces->

<https://github.com/ros2/rclpy/blob/master/rclpy/src/rclpy/node.cpp>

Node::get_node_names_and_namespaces()->

get_names_impl()

rcl_get_node_names

<https://github.com/ros2/rcl/blob/master/rcl/src/rcl/graph.c>

rmw_get_node_names

## vscode调试

```json
            "name": "zs: gdbrun py",
            "type": "python",
            "request": "launch",
            "program": "${workspaceRoot}/colcon/install/ros2cli/bin/ros2",
            // "args" : ["topic", "list"],
            "args" : ["topic", "echo", "/tf"],
            "console": "integratedTerminal",
            "cwd": "${workspaceFolder}",
            "internalConsoleOptions": "openOnFirstSessionStart",
            "presentation": {
                "hidden": false,
                "group": "",
                "order": 1
```

```yaml
{
            "name": "zs: gdbrun ros2",
            "type": "cppdbg",
            "request": "launch",
            "program": "/usr/bin/python3",
            "args": [
                "/opt/ros/foxy/bin/ros2",
                "topic",
                "echo",
                "/tf_static"
            ],
            "stopAtEntry": false,
            "cwd": "${workspaceRoot}",
            "environment": [
                {
                    "name": "RMW_IMPLEMENTATION",
                    // "value": "rmw_fastrtps_cpp"
                    // "value": "rmw_ecal_proto_cpp"
                    "value": "rmw_iceoryx_cpp"
                }
            ],
            "externalConsole": false,
            // "preLaunchTask": "source_ros2_ws",
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        },

```

# 参考

[Introspection with command line tools — ROS 2 Documentation: Rolling  documentation](https://docs.ros.org/en/rolling/Concepts/About-Command-Line-Tools.html)
