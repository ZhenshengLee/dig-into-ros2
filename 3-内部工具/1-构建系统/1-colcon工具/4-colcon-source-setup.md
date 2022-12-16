# setup.bash作用

## setup的作用

设置一些环境变量，用于ros2的启动和运行

It is important that we have sourced the environment for an existing ROS 2 installation that will provide our workspace with the necessary build dependencies for the example packages. This is achieved by sourcing the setup script provided by a binary installation or a source installation

### workspace概念

工作空间是ros源代码组织的空间

一个空间和另一个空间应该是独立的

一个空间也可以依赖另一个空间中的包

another colcon workspace (see [Installation](https://docs.ros.org/en/foxy/Installation.html)). We call this environment an **underlay**.

Our workspace, `ros2_examples_ws`, will be an **overlay** on top of the existing ROS 2 installation. In general, it is recommended to use an overlay when you plan to iterate on a small number of packages, rather than putting all of your packages into the same workspace.

## local的作用

Both scripts can be used to setup the environment, but their effect is slightly different.

- `local_setup.*` will add to the path only the directories built in the current workspace

- `setup.*` will add to the path the directories built in the current workspace AND the directories of the packages that were sourced while building the current workspace. This last operation will be done by sourcing the `local_setup.*` files for those workspaces.

Some examples:

- If you want to source the ROS2 SDK, the two scripts will have the same effect as there are no workspaces which are on top of the current one.

## 实践

I create a workspace for my new package. In order to build it I have to source the ROS2 SDK. When I will run something from this package I will have two options for setting up the environment: I can `source setup.*` OR I can `source /opt/ros2/bouncy/setup.*` and `source local_setup.*`

## 找不到local

删除下列描述内容

```text
 if(BUILD_TESTING)
      find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

```

## 参考

[could not source the setup.bash in ROS 2? - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/320968/could-not-source-the-setupbash-in-ros-2/)

[What is the difference between local_setup.bash and setup.bash - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/292566/what-is-the-difference-between-local_setupbash-and-setupbash/)

[what is difference between local_setup.bash and setup.bash in ros2 - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/307764/what-is-difference-between-local_setupbash-and-setupbash-in-ros2/)

[Using colcon to build packages — ROS 2 Documentation: Foxy  documentation](https://docs.ros.org/en/foxy/Tutorials/Colcon-Tutorial.html)
