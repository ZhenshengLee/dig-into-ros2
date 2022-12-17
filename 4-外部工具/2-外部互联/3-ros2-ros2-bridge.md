# 基础功能

# Docker

I am trying to use ROS1 Bridge to work with ROS1 nodes from within ROS2 (Dashing) inside the docker environment (ADE).

PS. I want to use ROS2 inside the docker environment and communicate with ROS1 nodes outside the docker.

I am using Ubuntu 18.04

Is there a good documentation available for enabling the use of the ROS bridge inside the docker environment?

Thank you.

@shokgoel There is nothing required inside the Docker container to enable the bridge outside of the container. However, whether you are running the bridge inside or outside of the container, the ROS1/ROS2 bridge needs access to both a ROS1 _and_ a ROS2 environment so you must have both installed in the same place (either inside or outside the Docker environment). Additionally, if you want to use messages that are not part of [common_interfaces](https://github.com/ros2/common_interfaces), you will have to build the package from source. You can get more details [here](https://github.com/ros2/ros1_bridge/blob/master/doc/index.rst).

# 参考

[https://github.com/ros2/ros1_bridge](https://github.com/ros2/ros1_bridge)

[Using ROS1 Bridge to Communicate between ROS1 and ROS2 - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/369014/using-ros1-bridge-to-communicate-between-ros1-and-ros2/)
