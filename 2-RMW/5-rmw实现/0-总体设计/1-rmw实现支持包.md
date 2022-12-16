ROS middleware implementations are sets of [packages](https://docs.ros.org/en/rolling/Glossary.html#term-package) that implement some of the internal ROS interfaces, e.g. the `rmw`, `rcl`, and `rosidl` [APIs](https://docs.ros.org/en/rolling/Glossary.html#term-API).

# The `rcl` Repository

The ROS Client Library interface (`rcl` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API)) can be used by [client libraries](https://docs.ros.org/en/foxy/Glossary.html#term-client_library) (e.g. `rclc`, `rclcpp`, `rclpy`, etc.) in order to avoid duplicating logic and features. By reusing the `rcl` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API), client libraries can be smaller and more consistent with each other. Some parts of the client library are intentionally left out of the `rcl` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API) because the language idiomatic method should be used to implement those parts of the system.

A good example of this is the execution model, which `rcl` does not address at all. Instead the client library should provide a language idiomatic solution like `pthreads` in C, `std::thread` in C++11, and `threading.Thread` in Python. Generally the `rcl` interface provides functions that are not specific to a language pattern and are not specific to a particular message type.The `rcl` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API) is located in the [ros2/rcl](https://github.com/ros2/rcl) repository on [GitHub](https://github.com/) and contains the interface as C headers.

The `rcl` C implementation is provided by the `rcl` [package](https://docs.ros.org/en/foxy/Glossary.html#term-package) in the same repository. This implementation avoids direct contact with the middleware by instead using the `rmw` and `rosidl` [APIs](https://docs.ros.org/en/foxy/Glossary.html#term-API).

# The `rmw` Repository

The ROS middleware interface (`rmw` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API)) is the minimal set of primitive middleware capabilities needed to build ROS on top. Providers of different middleware implementations must implement this interface in order to support the entire ROS stack on top.

Currently all of the middleware implementations are for different DDS vendors.The `rmw` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API) is located in the [ros2/rmw](https://github.com/ros2/rmw) repository. The `rmw` [package](https://docs.ros.org/en/foxy/Glossary.html#term-package) contains the C headers which define the interface, the implementation of which is provided by the various [packages](https://docs.ros.org/en/foxy/Glossary.html#term-package) of rmw implementations for different DDS vendors.For a definition of the `rmw` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API), see the [API](https://docs.ros.org/en/foxy/Glossary.html#term-API) docs:

# The `rcutils` Repository

ROS 2 C Utilities (`rcutils`) is a C [API](https://docs.ros.org/en/foxy/Glossary.html#term-API) composed of macros, functions, and data structures used throughout the ROS 2 codebase.

These are mainly used for error handling, commandline argument parsing, and logging which are not specific to the client or middleware layers and can be shared by both.The `rcutils` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API) and implementation are located in the [ros2/rcutils](https://github.com/ros2/rcutils) repository on [GitHub](https://github.com/) which contains the interface as C headers.

# rmw_dds_common

专为dds支撑的工具

- 包括一个graphCache，跟踪dds实体

  - 注意：node和dds participate已经不是一一对应的关系了

- 节点发现信息

  - `rmw_dds_common/msg/Gid`

  - `rmw_dds_common/msg/NodeEntitiesInfo`

  - `rmw_dds_common/msg/ParticipantEntitiesInfo`

- 一些抽象类

  - A generic `Context` type to withhold most state needed to implement [ROS nodes discovery](https://github.com/ros2/design/pull/250)

  - [Comparison utilities and some C++ operator overloads](https://github.com/ros2/rmw_dds_common/blob/master/rmw_dds_common/docs/rmw_dds_common/include/rmw_dds_common/gid_utils.hpp) for `rmw_gid_t` instances

  - [Conversion utilities](https://github.com/ros2/rmw_dds_common/blob/master/rmw_dds_common/docs/rmw_dds_common/include/rmw_dds_common/gid_utils.hpp) between `rmw_dds_common/msg/Gid` messages and `rmw_gid_t` instances

  - A function for checking the compatibility of two QoS profiles, `qos_profile_check_compatible`

iceoryx和ecal都不严格遵循dds规范，所以这个dds-common只能部分利用，在某些细节比如`domain_id`是不支持的

rqt工具直接调用rclpy的接口

是否可以直接使用底层接口而不使用dds_common呢？不推荐，因为以前就是这么做的，后来因为graph的维护导致了性能瓶颈，各个节点也就共享一个graph了，维护一个大的graph会方轻松一些。

所以最佳方式是使用dds_common作为graph的实现方式。

# RMW基本结构

Structure of ROS Middleware Implementations

A ROS middleware implementation is typically made up of a few [packages](https://docs.ros.org/en/rolling/Glossary.html#term-package) in a single repository:

- `<implementation_name>_cmake_module`: contains CMake Module for discovering and exposing required dependencies

- `rmw_<implementation_name>_<language>`: contains the implementation of the `rmw` [API](https://docs.ros.org/en/rolling/Glossary.html#term-API) in a particular language, typically C++

- `rosidl_typesupport_<implementation_name>_<language>`: contains tools to generate static type support code for `rosidl` files, tailored to the implementation in a particular language, typically C or C++

The `<implementation_name>_cmake_module` [package](https://docs.ros.org/en/rolling/Glossary.html#term-package) contains any CMake Modules and functions needed to find the supporting dependencies for the middleware implementation. For example, `rti_connext_dds_cmake_module` provides wrapper logic around the CMake Module shipped with RTI Connext DDS to make sure that all packages that depend on it will select the same installation of RTI Connext DDS. Similarly, `fastrtps_cmake_module` includes a CMake Module to find eProsima’s Fast DDS and `gurumdds_cmake_module` includes a CMake Module to find GurumNetworks GurumDDS. Not all implementations will have a package like this: for example, Eclipe’s Cyclone DDS already provides a CMake Module which is used directly by its RMW implementation without the need of additional wrappers.

- cmake_module是用来帮助cmake寻找三方依赖的

The `rmw_<implementation_name>_<language>` [package](https://docs.ros.org/en/rolling/Glossary.html#term-package) implements the `rmw` C [API](https://docs.ros.org/en/rolling/Glossary.html#term-API) in a particular language. The implementation itself can be C++, it just must expose the header’s symbols as `extern "C"` so that C applications can link against it.

The `rosidl_typesupport_<implementation_name>_<language>` [package](https://docs.ros.org/en/rolling/Glossary.html#term-package) provides a generator which generates DDS code in a particular language. This is done using the `.idl` files generated by the `rosidl_generator_dds_idl` [package](https://docs.ros.org/en/rolling/Glossary.html#term-package) and the DDS IDL code generator provided by the DDS vendor. It also generates code for converting ROS message structures to and from DDS message structures.

This generator is also responsible for creating a shared library for the message package it is being used in, which is specific to the messages in the message package and to the DDS vendor being used.

As mentioned above, the `rosidl_typesupport_introspection_<language>` may be used instead of a vendor specific type support package if an rmw implementation supports runtime interpretation of messages. This ability to programmatically send and receive types over topics without generating code beforehand is achieved by supporting the [DDS X-Types Dynamic Data standard](http://www.omg.org/spec/DDS-XTypes). As such, rmw implementations may provide support for the X-Types standard, and/or provide a package for type support generated at compile time specific to their DDS implementation.

# 参考

[ros2/rmw_dds_common](https://github.com/ros2/rmw_dds_common)
