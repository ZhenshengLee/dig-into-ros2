RCL对DDS需求

![](https://tcs.teambition.net/storage/312g95b5e911d4e8be97742347433816730d?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjkyMywiaWF0IjoxNjcxMDE4MTIzLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmc5NWI1ZTkxMWQ0ZThiZTk3NzQyMzQ3NDMzODE2NzMwZCJ9.yJe87TjkVxlYyczoNicmJ7NmjoWeJzo8t2UE_rHUXjo&download=image.png "")

As the diagram shows, these [APIs](https://docs.ros.org/en/foxy/Glossary.html#term-API) are stacked such that the typical ROS user will use the [client library](https://docs.ros.org/en/foxy/Glossary.html#term-client_library) [API](https://docs.ros.org/en/foxy/Glossary.html#term-API), e.g. `rclcpp`, to implement their code (executable or library). The implementation of the [client libraries](https://docs.ros.org/en/foxy/Glossary.html#term-client_library), e.g. `rclcpp`, use the `rcl` interface which provides access to the ROS graph and graph events. The `rcl` implementation in turn uses the `rmw` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API) to access the ROS graph. The purpose of the `rcl` implementation is to provide a common implementation for more complex ROS concepts and utilities that may be used by various [client libraries](https://docs.ros.org/en/foxy/Glossary.html#term-client_library), while remaining agnostic to the underlying middleware being used. The purpose of the `rmw` interface is to capture the absolute minimum middleware functionality needed to support ROS’s client libraries. Finally, the implementation of the `rmw` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API) is provided by a middleware implementation specific [package](https://docs.ros.org/en/foxy/Glossary.html#term-package), e.g. `rmw_fastrtps_cpp`, the library of which is compiled against vendor specific DDS interfaces and types.

rcl是各个语言客户端的共同抽象

rcl维护各个语言客户端共同的模型

rcl对dds功能最小的索取

# 需求

The `rmw` [API](https://docs.ros.org/en/foxy/Glossary.html#term-API) is the interface between the ROS 2 software stack and the underlying middleware implementation. The underlying middleware used for ROS 2 is either a DDS or RTPS implementation, and is responsible for discovery, publish and subscribe mechanics, request-reply mechanics for services, and serialization of message types.

# 参考

[About internal ROS 2 interfaces — ROS 2 Documentation: Foxy  documentation](https://docs.ros.org/en/foxy/Concepts/About-Internal-Interfaces.html?highlight=middleware)
