# 命令行发布tf

```bash
ros2 run tf2_ros static_transform_publisher 1 2 3 0.5 0.1 -1.0 foo bar
# That tool will publish a static transform from the parent frame foo to the child frame bar with (X, Y, Z) translation (1, 2, 3) and (yaw, pitch, roll) body-fixed axis rotation sequence (0.5, 0.1, -1.0).
```

# 查询tf

```
ros2 run tf2_ros tf2_echo foo bar
```

# tf_tree

```
sudo apt install ros-foxy-rqt-tf-tree
```

# tf不全会丢包

# 参考

[https://docs.ros.org/en/foxy/Tutorials/Intermediate/Tf2/Introduction-To-Tf2.html](https://docs.ros.org/en/foxy/Tutorials/Intermediate/Tf2/Introduction-To-Tf2.html)<br /><https://index.ros.org/doc/ros2/Tutorials/tf2/>
