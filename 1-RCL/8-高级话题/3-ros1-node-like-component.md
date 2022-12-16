在移植ros程序过程中发现问题

- 节点中存在nh和private_nh

  - 这个是命名空间的私有性需求

- nh和private_nh作为类的成员，作为参数下发给委托对象，

  - 这个是委托对象执行回调函数的需求

目前这两个问题在ros2中都没有很好的解决方式

第一个问题有官方解决办法，但是还没开发完毕，有bug。

第二个问题没有官方解决办法，不过可以绕开。

## private nodehandlers的需求

[When is private nodehandle useful - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/192191/when-is-private-nodehandle-useful/)

[private nodehandles in ros 2 - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/294483/private-nodehandles-in-ros-2/)

在ros2中已经不推荐这种编程方式，在ros2中`node`已经不是对象成员而是对象的父类。虽然有单独使用的用法见`examples/rclcpp/topics/minimal_publisher/not_composable.cpp`但是这种做法无法在一个`excutable`中存在两个`node`

```bash
/* We do not recommend this style anymore, because composition of multiple
 * nodes in the same executable is not possible. Please see one of the subclass
 * examples for the "new" recommended styles. This example is only included
 * for completeness because it is similar to "classic" standalone ROS nodes. */
```

不过也有demo给出了`node_like_components`的用法`demos/composition/include/composition/node_like_listener_component.hpp`

```bash
/ Create a Listener "component" that does not subclass the generic rclcpp::Node base class.
/**
 * Note that "components" don't have to derive from rclcpp::Node.
 * In the case that an object does not inherit from rclcpp::Node, then it must implement:
 * - Constructor that takes `const rclcpp::NodeOptions&`
 * - get_node_base_interface() which returns a NodeBaseInterface::SharedPtr
 *
 * This is an example of an object that implements the interface required to be a component.
 */
```

按照这种方法，一个类可以将node作为成员，该对象不继承自node类，也可以作为component看起来像node，这个貌似为node作为参数传递给委托对象创造了条件

```cpp
auto listener = std::make_shared<composition::NodeLikeListener>(options);
  exec.add_node(listener->get_node_base_interface());

```

## ROS2的尝试

[[Ros2] Creating Multiple nodes using class inheritance - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/313608/ros2-creating-multiple-nodes-using-class-inheritance/)

## 需求1-subnode概念

[rclcpp: rclcpp::Node Class Reference](https://docs.ros2.org/foxy/api/rclcpp/classrclcpp_1_1Node.html#a1df8e4507d17a503630317432449d69d)

subnode不支持创建一个完整的node，只能够创建一个新的命名空间，其实还是同一个Node

## subnode当前的bug

[fixup parameters and sub-nodes · Issue #731 · ros2/rclcpp](https://github.com/ros2/rclcpp/issues/731)

[[ROS2] Using create_sub_node leads to two nodes with same name - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/331451/ros2-using-create_sub_node-leads-to-two-nodes-with-same-name/)

会显示两个同名节点

网友给出了暂时解决办法，可以通过下面方法来模拟命名空间

Below, It is shown a possible workaround. You can just create a "full" new node extending the original node namespace (second parameter).

```cpp
  auto subnode = rclcpp::Node::make_shared("test", parent_node->get_fully_qualified_name()));
  rclcpp::spin(subnode)
```

- 缺点：不能共享node

  - 本质上也是两个节点，无法解决`private_nh`的问题

## 需求2-node作为参数传递

subnode并不符合我们private_nh的需求。

对于private_nh，有两种办法。

- 自然的想法是，可以在一个node中创建另一个node，然后在顶层新建node进行参数注入，使用`rclcpp::spin_some(node);`

  - 这个方法参考SMACC2的代码`smacc2/src/smacc/signal_detector.cpp`

  - 不能够完全使用component launch

    - component launch可以方便将node作为一个进程或者线程。

    - 但是可以使用`multiple executor`实现进程内通信

    - 具体编程需要新建一个`main.cpp`

    - 但是按照`__node_like_listener_component__`的方法就可以使用component了！

- 但是`velodyne-ros2/velodyne_driver/src/lib/input.cpp`中将客户对象集成node，自己的指针作为node传给委托对象

  - 这个方法是可以component的

  - 共享node，依然是一个节点

  - 很好的解决了这个问题

- 利用对象继承的办法传递node

  - 实际上整个对象就是一个对象，整个node节点贯穿子类和父类

    - 共享Node

    - 可以component

该方法有四种实现方法

- 第一种，客户对象和委托对象都用node传递的方式初始化

  - 优点

    - 方便，对象节点容易顶层获取

    - 共享node，本质上是一个节点

  - 缺点：

    - 无法直接component，可以component like

- 第二种：客户对象继承自node，委托对象也继承自node

  - 顶层通过客户对象获取委托对象的node，然后spin_some

  - 但是本质上是两个节点

- 第三种，客户对象继承自node，委托对象传递自己指针作为node

  - 优点：

    - 方便后续重构，收回委托对象的nodehandler

  - 缺点：客户对象的命名空间不好获取，委托对象node不好获取，要额外编码

- 第四种：客户对象继承自委托对象，将委托关系改变成继承关系

  - 本质上是一个节点

  - 只有一个nh，命名空间通过编程实现私有

  - 顶层不需要spin_some，只需要spin即可

  - 算比较完美的解决了这个问题

在子对象中使用基类的函数，参考

[C++派生类中调用基类成员_C语言中文网](http://c.biancheng.net/cpp/biancheng/view/2768.html)

## 结论

对于需求1和需求2，采用node作为参数传递的方法是可行的。

对于需求一，未来可以用subnode，subnode作为参数传递给委托对象，即可同时满足需求二。

当前subnode不可用，对于需求二，可以使用方法三和方法四，更加推荐第三种方法

继承方式在ros2中是很成熟的，而component like更像是一种workaround.

不过解决这个问题最好还是重构，重构的工作需要思考：

- 重新分配两个对象的职责，是否需要两个节点？

- 将委托对象的通信职责收回到客户对象，这样被委托对象是一个非ROS的纯cpp类

## 搜索关键字

ros2 node in node class

executor as a member of a node ros2

creating nodes not using inheritance
