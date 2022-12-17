# rmw互通现状

interoperatibility

inter vendor communication

**For many cases you will find that nodes using different RMW implementations are able to communicate**, however this is not true under all circumstances.

Here is a list of inter-vendor communication configurations that are not supported:

- Fastrtps vs. Connext

- OpenSplice vs. OpenSplice does not support `WString`

- Connext vs. CycloneDDs does not support `WString`

- Connext vs. Connext (connext_dynamic) does not support C services

| rmw                                                     | fastdds | cyclonedds | connext | iceoryx |
| ------------------------------------------------------- | ------- | ---------- | ------- | ------- |
| fastdds                                                 | -       | -          | -       | -       |
| cyclonedds                                              | Y       | -          | -       | -       |
| connext                                                 | N       | Y*         | -       | -       |
| iceoryx                                                 |         |            |         | -       |
| *: does not support pub/sub communication for `WString` |         |            |         |         |

# system tests

自动化测试各种rmw的互通性

# 参考

[Enable cross-vendor tests for unknown RMW implementations · Issue #429 · ros2/system_tests](https://github.com/ros2/system_tests/issues/429)

[Service interoperability with FastRTPS · Issue #184 · ros2/rmw_cyclonedds](https://github.com/ros2/rmw_cyclonedds/issues/184)
