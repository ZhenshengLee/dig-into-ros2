# 概念

数据交换格式

包括数据表示，数据编码（串行和解串），语言类型映射，以及代码生成器（编译器）

## 消息表示

以描述文件的方式来描述数据结构，一方面比较清晰，一方面是一种统一的方式（独立于语言），可读性也更好

## 编解码

数据结构的内存表示不适合网络通信，效率低，不可读，开发复杂度高

对于单机进程来讲，string或者vector<char>的确可以满足需求，但是对于复杂数据结构，则复杂度很高

## 支持RPC

复杂结构以及RPC网络通信模式，对消息表示提出了更高的要求，需要一个系统化工具来支持

# 数据表示

## 可读or 不可读

不可读：ddsidl, protobuf,

可读：json, xml, flatbuffers

# 数据编码

## 编码方法

## 性能

与内存布局结构越一致，性能越好

# cpp类型映射

# 需求

完善的工具配套

理想的性能

复杂数据类型支持

协议动态升级兼容

# 自动生成工具

[__https://github.com/foxglove/message-schemas__](https://github.com/foxglove/message-schemas)

通过ts语言自动生成ros1, ros2, json, protobuf的消息格式

# 参考

[GitHub - foxglove/message-schemas: Standard ROS message types converted to Protobuf and JSON Schema](https://github.com/foxglove/message-schemas)
