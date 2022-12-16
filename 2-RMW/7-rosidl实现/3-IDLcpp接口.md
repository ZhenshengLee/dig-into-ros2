# 文件格式

cpp，hpp

![](https://tcs.teambition.net/storage/312g2b541845c219495f8346f7e8cd6530c5?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTA3NywiaWF0IjoxNjcxMTk0Mjc3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmcyYjU0MTg0NWMyMTk0OTVmODM0NmY3ZThjZDY1MzBjNSJ9.KXDmRzZxqyqihf2v63l4VrdqQmjbyLzgmlHH5HFD8Xc&download=image.png "")

# 包含内容

类，成员，常量定义，构造器，读取器和设置器，操作符， 指针

Types

Mapping of primitive types

| ROS type | C++ type    |
| -------- | ----------- |
| bool     | bool        |
| byte     | uint8_t     |
| char     | char        |
| float32  | float       |
| float64  | double      |
| int8     | int8_t      |
| uint8    | uint8_t     |
| int16    | int16       |
| uint16   | uint16      |
| int32    | int32       |
| uint32   | uint32      |
| int64    | int64       |
| uint64   | uint64_t    |
| string   | std::string |

Mapping of arrays and bounded strings

| ROS type                | C++ type           |
| ----------------------- | ------------------ |
| static array            | std::array<T, N>   |
| unbounded dynamic array | std::vector        |
| bounded dynamic array   | custom_class<T, N> |
| bounded string          | std::string        |

### Members

The struct has same-named public member variables for every field of the message. For each field a `typedef` is created which is named after the member with a leading underscore and a trailing `_type`.

### Constants

Numeric constants are defined as `enums` within the struct.All other constants are declared as `static const` members in the struct and their values are defined outside of the struct.

### Constructors

In the following discussion, “member” refers to the class member in the C++ class while “field” refers to the field definition in the IDL file.

The *default constructor* initializes all members with the default value specified in the IDL file, or otherwise with the common default for the field type [as defined in this article](http://design.ros2.org/articles/interface_definition.html#default-values) (note: `char` fields are considered numeric for C++). In some cases this may not be desirable, since these fields will often be immediately overwritten with user-provided values. Therefore, the constructor takes an optional directive of type `rosidl_generator_cpp::MessageInitialization` to control how initialization is done:

- `MessageInitialization::ALL` - Initialize each member with the field’s default value specified in the IDL file, or otherwise with the common default for the field type [as defined in this article](http://design.ros2.org/articles/interface_definition.html#default-values) (note: `char` fields are considered numeric for C++).

  - The safest option, and also the default (used if not passing any argument to the constructor).

- `MessageInitialization::SKIP` - Don’t initialize any members; it is the user’s responsibility to ensure that all members get initialized with some value, otherwise undefined behavior may result

  - Used for maximum performance if the user is setting all of the members themselves.

- `MessageInitialization::ZERO` - Zero initialize all members; all members will be [value-initialized](http://en.cppreference.com/w/cpp/language/value_initialization) ([dynamic size](https://design.ros2.org/articles/interface_definition.html#arrays-with-dynamic-size) or [upper boundary](https://design.ros2.org/articles/interface_definition.html#upper-boundaries) arrays will have zero elements), and default values from the message definition will be ignored

  - Used when the user doesn’t want the overhead of initializing potentially complex or large default values, but still wants to ensure that all variables are properly initialized.

- `MessageInitialization::DEFAULTS_ONLY` - Initialize only members that have field default values; all other members will be left uninitialized

  - Minimal initialization which ensures that existing code has correctly initialized members when a new field with a default value is added to the IDL later.

Optionally the constructor can be invoked with an allocator.

The struct has no constructor with positional arguments for the members. The short reason for this is that if code would rely on positional arguments to construct data objects changing a message definition would break existing code in subtle ways. Since this would discourage evolution of message definitions the data structures should be populated by setting the members separately, e.g. using the setter methods.

### Setters

For each field a *setter* method is generated to enable [method chaining](https://isocpp.org/wiki/faq/ctors#named-parameter-idiom). They are named after the fields with a leading `set__`. The setter methods have a single argument to pass the value for the member variable. Each setter method returns the struct itself.

### Operators

The comparison operators `==` and `!=` perform the comparison on a per member basis.

### Pointer types

The struct contains `typedefs` for the four common pointer types `plain pointer`, `std::shared_ptr`, `std::unique_ptr`, `std::weak_ptr`. For each pointer type there a non-const and a const `typedef`:

- `RawPtr` and `ConstRawPtr`

- `SharedPtr` and `ConstSharedPtr`

- `UniquePtr` and `ConstUniquePtr`

- `WeakPtr` and `ConstWeakPtr`

For similarity to ROS 1 the `typedefs` `Ptr` and `ConstPtr` still exist but are deprecated. In contrast to ROS 1 they use `std::shared_ptr` instead of Boost.

# struct

![](https://tcs.teambition.net/storage/312gfac61367f5e7a7f52fac67eedff7feee?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTA3NywiaWF0IjoxNjcxMTk0Mjc3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdmYWM2MTM2N2Y1ZTdhN2Y1MmZhYzY3ZWVkZmY3ZmVlZSJ9.PwTWdVw_HHVRm8-MBUYVbvvzoOAHfv9ZQB8jedMN-y0&download=image.png "")

# builder

![](https://tcs.teambition.net/storage/312gbe5257844900472c88ce1d3b38a40b01?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTA3NywiaWF0IjoxNjcxMTk0Mjc3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdiZTUyNTc4NDQ5MDA0NzJjODhjZTFkM2IzOGE0MGIwMSJ9.x3dyBO1DcRmqpMMvsV5lv6eJPLCqasZzmgFUS4YuwKk&download=image.png "")

# Traits

![](https://tcs.teambition.net/storage/312gc3fee13ca8864ba90363b0b019b5381a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTA3NywiaWF0IjoxNjcxMTk0Mjc3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdjM2ZlZTEzY2E4ODY0YmE5MDM2M2IwYjAxOWI1MzgxYSJ9.Qiw_GmLvlO0QWwuCF33-dSR1eDqQojeeet5cUT2LtoQ&download=image.png "")

# 参考

[Generated C++ interfaces](https://design.ros2.org/articles/generated_interfaces_cpp.html)
