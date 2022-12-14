# rclcpp职责

Client libraries are the APIs that allow users to implement their ROS code. Using client libraries, users gain access to ROS concepts such as nodes, topics, services, etc. Client libraries come in a variety of programming languages so that users may write ROS code in the language that is best-suited for their application. For example, you might prefer to write visualization tools in Python because it makes prototyping iterations faster, while for parts of your system that are concerned with efficiency, the nodes might be better implemented in C++.

Nodes written using different client libraries are able to share messages with each other because all client libraries implement code generators that provide users with the capability to interact with ROS interface files in the respective language.

In addition to the language-specific communication tools, client libraries expose to users the core functionality that makes ROS “ROS”. For example, here is a list of functionality that can typically be accessed through a client library:

- Names and namespaces

- Time (real or simulated)

- Parameters

- Console logging

- Threading model

- Intra-process communication

# rcl职责

Most of the functionality found in a client library is not specific to the programming language of the client library. For example, the behavior of parameters and the logic of namespaces should ideally be the same across all programming languages. Because of this, rather than implementing the common functionality from scratch, client libraries make use of a common core ROS Client Library (RCL) interface that implements logic and behavior of ROS concepts that is not language-specific. As a result, client libraries only need to wrap the common functionality in the RCL with foreign function interfaces. This keeps client libraries thinner and easier to develop. For this reason the common RCL functionality is exposed with C interfaces as the C language is typically the easiest language for client libraries to wrap.

In addition to making the client libraries light-weight, an advantage of having the common core is that the behavior between languages is more consistent. If any changes are made to the logic/behavior of the functionality in the core RCL – namespaces, for example – all client libraries that use the RCL will have these changes reflected. Furthermore, having the common core means that maintaining multiple client libraries becomes less work when it comes to bug fixes.

[The API documentation for the RCL can be found here.](https://docs.ros2.org/latest/api/rcl/)

# 与ROS1比较

In ROS 1, all client libraries are developed “from the ground up”. This allows for the ROS 1 Python client library to be implemented purely in Python, for example, which brings benefits of such as not needing to compile code. However, naming conventions and behaviors are not always consistent between client libraries, bug fixes have to be done in multiple places, and there is a lot of functionality that has only ever been implemented in one client library (e.g. UDPROS).

# rclc

那么为了可移植，有没有纯posix c的客户端呢？有的OFERA项目就提供了rclc

这个rclc可以说是c的客户端，但是由于rcl本来就是c的，所以rclc的定位就是提供了rcl的功能补充

这也意味着，如果rclc的功能未来集成到rcl中，rclcpp等其他客户端也可以使用rcl的功能，相当于功能转正了。

rclc最大的好处就是可以移植

rclc是micro-ROS的重要组成部分，micro-ROS是具有实时性的ROS2，而且支持RTOS

关于实时性APEX.OS也做过探索，不过其还是基于Linux内核

# rclrs

使用rs语言的客户端

从架构图可以看出，只需要添加rclrs即可实现ros2的语言绑定，而不需要增加中间件，rmw的语言绑定

<https://github.com/ros2-rust/ros2_rust>

# 参考

[About ROS 2 client libraries — ROS 2 Documentation: Galactic  documentation](https://docs.ros.org/en/galactic/Concepts/About-ROS-2-Client-Libraries.html)
