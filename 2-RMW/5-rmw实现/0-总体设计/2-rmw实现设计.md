# Design of the *middleware interface*

The API is designed as a pure function-based interface in order to be implemented in C. A pure C interface can be used in ROS Client Libraries for most other languages including Python, Java, and C++ preventing the need to reimplement the core logic.

### Publisher interface

Based on the general structure of ROS nodes, publishers and messages for the case of publishing messages the ROS client library need to invoke three functions on the middleware interface:

- `create_node()`

- `create_publisher()`

- `publish()`

Essential signature of `create_node`Subsequent invocations of `create_publisher` need to refer to the specific node they should be created in. Therefore the `create_node` function needs to return a *node handle* which can be used to identify the node.

```text
NodeHandle create_node();

```

Essential signature of `create_publisher`Besides the *node handle* the `create_publisher` function needs to know the *topic name* as well as the *topic type*. The type of the *topic type* argument is left unspecified for now.Subsequent invocations of `publish` need to refer to the specific publisher they should send messages on. Therefore the `create_publisher` function needs to return a *publisher handle* which can be used to identify the publisher.

```text
PublisherHandle create_publisher(NodeHandle node_handle, String topic_name, .. topic_type);

```

The information encapsulated by the *topic type* argument is highly dependent on the middleware implementation.

Topic type information for the DynamicData API

In the case of using the DynamicData API in the implementation there is no C / C++ type which could represent the type information. Instead the *topic type* must contain all information needed to describe the format of the message. This information includes:

- the name of the package in which the message is defined

- the name of the message

- the list of fields of the message where each includes:

  - the name for the message field

  - the type of the message field (which can be either a built-in type or another message type), optionally the type might be an unbounded, bounded or fixed size array

  - the default value

- the list of constants defined in the message (again consisting of name, type and value)

In the case of using DDS this information enables one to:

- programmatically create a *DDS TypeCode* which represents the message structure

- register the *DDS TypeCode* with the *DDS Participant*

- create a *DDS Publisher*, *DDS Topic* and *DDS DataWriter*

- convert data from a *ROS message* into a *DDS DynamicData* instance

- write the *DDS DynamicData* to the *DDS DataWriter*

Topic type information for statically generated code

In the case of using statically generated code derived from an IDL there is are C / C++ types which represent the type information. The generated code contains functions to:

- create a *DDS TypeCode* which represents the message structure

Since the specific types must be defined at compile time the other functionalities can not be implemented in a generic (not specific to the actual message) way. Therefore for each message the code to perform the following tasks must be generated separately:

- register the *DDS TypeCode* with the *DDS Participant*

- create a *DDS Publisher*, *DDS Topic* and *DDS DataWriter*

- convert data from a *ROS message* into a *DDS DynamicData* instance

- write the *DDS DynamicData* to the *DDS DataWriter*

The information encapsulated by the *topic type* must include the function pointers to invoke these functions.

`get_type_support_handle`

Since the information encapsulated by the *topic type* argument is so fundamentally different for each middleware implementation it is actually retrieved through an additional function of the middleware interface:

```text
MessageTypeSupportHandle get_type_support_handle();

```

Currently this function is a template function specialized on the specific ROS message type. For C compatibility an approach using a macro will be used instead which mangles the type name into the function name.

Essential signature of `publish`Beside the *publisher handle* the `publish` function needs to know the *ROS message* to send.

```text
publish(PublisherHandle publisher_handle, .. ros_message);

```

Since ROS messages do not have a common base class the signature of the function can not use a known type for the passed ROS message. Instead it is passed as a void pointer and the actual implementation reinterprets it according to the previously registered type.

Type of the returned handles

The returned handles need to encapsulate arbitrary content for each middleware implementation. Therefore these handles are just opaque objects from a user point of view. Only the middleware implementation which created it knows how to interpret it.

In order to ensure that these information are passed back to the same middleware implementation each handle encodes a unique identifier which the middleware implementation can check before interpreting the handles payload.

### Subscriber interface

The details of the interface necessary for the subscriber side are not (yet) described in this document.

### Optionally exposing native handles

The RMW interface only exposes middleware agnostic handles. But the middleware implementation can optionally provide additional API to provide native handles. E.g. for a given ROS publisher handle a specific implementation can provide an API to access publisher related handles specific to the implementation.

While using such a feature would make the user land code specific to the middleware implementation it provides a way to use features of a middleware implementation which are not exposed through the ROS interface. See [this](https://github.com/ros2/demos/tree/master/demo_nodes_cpp_native) demo for an example.

## Current implementation

The described concept has been implemented in the following packages:

- The package [rmw](https://github.com/ros2/rmw/tree/master/rmw) defines the middleware interface.

  - The functions are declared in [rms/rmw.h](https://github.com/ros2/rmw/blob/master/rmw/include/rmw/rmw.h).

  - The handles are defined in [rmw/types.h](https://github.com/ros2/rmw/blob/master/rmw/include/rmw/types.h).

- The package [rosidl_typesupport_introspection_cpp](https://github.com/ros2/rosidl/tree/master/rosidl_typesupport_introspection_cpp) generates code which encapsulated the information from each ROS msg file in a way which makes the data structures introspectable from C++ code.

- The package [rmw_fastrtps_cpp](https://github.com/ros2/rmw_fastrtps/tree/master/rmw_fastrtps_cpp) implements the middleware interface using [eProsima Fast-RTPS](http://www.eprosima.com/index.php/products-all/eprosima-fast-rtps) based on the introspection type support.

- The package [rosidl_generator_dds_idl](https://github.com/ros2/rosidl_dds/tree/master/rosidl_generator_dds_idl) generates DDS IDL files based on ROS msg files which are being used by all DDS-based RMW implementations which use static / compile type message types.

- The package [rmw_connext_cpp](https://github.com/ros2/rmw_connext/tree/master/rmw_connext_cpp) implements the middleware interface using [RTI Connext DDS](http://www.rti.com/products/dds/index.html) based on statically generated code.

  - The package [rosidl_typesupport_connext_cpp](https://github.com/ros2/rmw_connext/tree/master/rosidl_typesupport_connext_cpp) generates:

    - the DDS specific code based on IDL files for each message

    - additional code to enable invoking the register/create/convert/write functions for each message type

- The package [rmw_connext_dynamic_cpp](https://github.com/ros2/rmw_connext/tree/master/rmw_connext_dynamic_cpp) implements the middleware interface using *RTI Connext DDS* based on the `DynamicData` API of Connext and the introspection type support.

- The package [rmw_opensplice_cpp](https://github.com/ros2/rmw_opensplice/tree/master/rmw_opensplice_cpp) implements the middleware interface using [PrismTech OpenSplice DDS](http://www.prismtech.com/opensplice) based on statically generated code.

  - The package [rosidl_typesupport_opensplice_cpp](https://github.com/ros2/rmw_opensplice/tree/master/rosidl_typesupport_opensplice_cpp) generates:

    - the DDS specific code based on IDL files for each message

    - additional code to enable invoking the register/create/convert/write functions for each message type

- The package [rmw_implementation](https://github.com/ros2/rmw_implementation/tree/master/rmw_implementation) provides the mechanism to switch between compile time and runtime selection of the middleware implementation.

  - If only one implementation is available at compile time it links directly against it.

  - If multiple implementations are available at compile time it implements the middleware interface itself and acts according to the strategy pattern by loading the shared library of a specific middleware implementation identified by an environment variable at runtime and pass all calls along.

### One or multiple type support generators

The packages contributing to the message generation process of each message are called *type support generators*. Their package name starts with the prefix `rosidl_typesupport_`.

Each message package will contain the generated code from all type support generators which are available when the package is configured. This can be only one (when building against a single middleware implementation) or multiple type support generators.

### Mapping between DDS and ROS concepts

Every ROS node is one DDS participant. If multiple ROS nodes are being run in a single process they are still mapped to separate DDS participants. If the containing process exposes its own ROS interface (e.g. to load nodes into the process at runtime) it is acting as a ROS node itself and is therefore also mapped to a separate DDS participant.

The ROS publishers and subscribers are mapped to DDS publishers and subscribers. The DDS DataReader and DataWriter as well as DDS topics are not exposed through the ROS API.

The ROS API defines queue sizes and a few [Quality of Service parameters](http://design.ros2.org/articles/qos.html) which are being mapped to their DDS equivalent. The other DDS QoS parameters are not being exposed through the ROS API.

# 示例：publisher

```text
create_node()
create_publisher()
publish()
NodeHandle create_node();
PublisherHandle create_publisher(NodeHandle node_handle, String topic_name, .. topic_type);
publish(PublisherHandle publisher_handle, .. ros_message);

```

## 示例：调用栈

```text
libc.so.6!__GI_raise(int sig) (/build/glibc-YbNSs7/glibc-2.31/sysdeps/unix/sysv/linux/raise.c:50)
libc.so.6!__GI_abort() (/build/glibc-YbNSs7/glibc-2.31/stdlib/abort.c:79)
libstdc++.so.6![Unknown/Just-In-Time compiled code] (Unknown Source:0)
libstdc++.so.6!std::terminate() (Unknown Source:0)
libstdc++.so.6!__cxa_throw (Unknown Source:0)
librmw_ecal_proto_cpp.so!eCAL::rmw::ProtoTypeSupportFactory::Create(rosidl_message_type_support_t const*) const (Unknown Source:0)
librmw_ecal_shared_cpp.so!eCAL::rmw::rmw_create_publisher(char const*, eCAL::rmw::TypesupportFactory const&, rmw_node_t const*, rosidl_message_type_support_t const*, char const*, rmw_qos_profile_t const*, rmw_publisher_options_t const*) (Unknown Source:0)
librmw_ecal_proto_cpp.so!rmw_create_publisher (Unknown Source:0)
librcl.so!rcl_publisher_init (Unknown Source:0)
librcl.so!rcl_logging_rosout_init_publisher_for_node (Unknown Source:0)
librcl.so!rcl_node_init (Unknown Source:0)
librclcpp.so!rclcpp::node_interfaces::NodeBase::NodeBase(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::shared_ptr<rclcpp::Context>, rcl_node_options_t const&, bool, bool) (Unknown Source:0)
librclcpp.so!rclcpp::Node::Node(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, rclcpp::NodeOptions const&) (Unknown Source:0)
librclcpp.so!rclcpp::Node::Node(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, rclcpp::NodeOptions const&) (Unknown Source:0)
libtopics_library.so!demo_nodes_cpp::Talker::Talker(rclcpp::NodeOptions const&) (Unknown Source:0)
libtopics_library.so!rclcpp_components::NodeFactoryTemplate<demo_nodes_cpp::Talker>::create_node_instance(rclcpp::NodeOptions const&) (Unknown Source:0)
main (Unknown Source:0)

```

# 参考

[About ROS 2 middleware implementations — ROS 2 Documentation: Rolling  documentation](https://docs.ros.org/en/rolling/Concepts/About-Middleware-Implementations.html)

[ROS 2 middleware interface](https://design.ros2.org/articles/ros_middleware_interface.html)
