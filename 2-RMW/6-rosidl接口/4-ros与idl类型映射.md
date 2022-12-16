# 概述

simplified description language，ros2使用idl，来描述数据交换接口，ros使用自动化工具来生成代码来描述这些协议类型，不同语言有不同的生成工具

# ROS与DDS idl类型映射

内置类型和自定义类型

| Type name | C++            | Python          | DDS type           |
| --------- | -------------- | --------------- | ------------------ |
| bool      | bool           | builtins.bool   | boolean            |
| byte      | uint8_t        | builtins.bytes* | octet              |
| char      | char           | builtins.str*   | char               |
| float32   | float          | builtins.float* | float              |
| float64   | double         | builtins.float* | double             |
| int8      | int8_t         | builtins.int*   | octet              |
| uint8     | uint8_t        | builtins.int*   | octet              |
| int16     | int16_t        | builtins.int*   | short              |
| uint16    | uint16_t       | builtins.int*   | unsigned short     |
| int32     | int32_t        | builtins.int*   | long               |
| uint32    | uint32_t       | builtins.int*   | unsigned long      |
| int64     | int64_t        | builtins.int*   | long long          |
| uint64    | uint64_t       | builtins.int*   | unsigned long long |
| string    | std::string    | builtins.str    | string             |
| wstring   | std::u16string | builtins.str    | wstring            |

每一个内置类型可以来定义数组, array

| Type name               | C++                | Python         | DDS type       |
| ----------------------- | ------------------ | -------------- | -------------- |
| static array            | std::array<T, N>   | builtins.list* | T[N]           |
| unbounded dynamic array | std::vector        | builtins.list  | sequence       |
| bounded dynamic array   | custom_class<T, N> | builtins.list* | sequence<T, N> |
| bounded string          | std::string        | builtins.str*  | string         |

常量值，可以在程序中使用该消息定义的宏

```text
constanttype CONSTANTNAME=constantvalue
```

```text
int32 X=123
int32 Y=-123
string FOO="foo"
string EXAMPLE='bar'
```

# 参考

[About ROS 2 interfaces — ROS 2 Documentation: Galactic  documentation](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#interfaceconcept)

[Expanding on ROS 2 interfaces — ROS 2 Documentation: Galactic  documentation](https://docs.ros.org/en/galactic/Tutorials/Single-Package-Define-And-Use-Interface.html)

[C++语言中std::array的神奇用法总结，你需要知道！-云社区-华为云](https://bbs.huaweicloud.com/blogs/216330)

[std::basic_string_view - cppreference.com](https://en.cppreference.com/w/cpp/string/basic_string_view)

[What's the difference between span and array_view in the gsl library?](https://stackoverflow.com/questions/34832090/whats-the-difference-between-span-and-array-view-in-the-gsl-library)

# 零拷贝数据结构

![](https://tcs.teambition.net/storage/31288112246505dac6aae9b66ffed7318ca0?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTAyMCwiaWF0IjoxNjcxMTk0MjIwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjg4MTEyMjQ2NTA1ZGFjNmFhZTliNjZmZmVkNzMxOGNhMCJ9.SIivaAM1ZFzEt7Z0OWIIDbTcrVwI746sft8ukSWZ5dc&download=image.png "")

怎样以最小的代价实现定长数据结构？

string到std::array的替换

std::array目前并不能很好支持与string和char[]的比较

所以还是使用memcpy和memcmp，数据结构中加上实际的size

实际工作代码在

后续优化，使用c++17的string view和 std::span，array_view，span不拥有所有权，
