就是指ROS内部的接口定义语言msg, srv, action

## [1. Background](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#id1)[](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#background "Permalink to this headline")

ROS applications typically communicate through interfaces of one of three types: messages, services and actions. ROS 2 uses a simplified description language, the interface definition language (IDL), to describe these interfaces. This description makes it easy for ROS tools to automatically generate source code for the interface type in several target languages.

In this document we will describe the supported types.

- msg: `.msg` files are simple text files that describe the fields of a ROS message. They are used to generate source code for messages in different languages.

- srv: `.srv` files describe a service. They are composed of two parts: a request and a response. The request and response are message declarations.

- action: `.action` files describe actions. They are composed of three parts: a goal, a result, and feedback. Each part is a message declaration itself.

## [2. Message description specification](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#id2)[](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#message-description-specification "Permalink to this headline")

Messages are described and defined in `.msg` files in the `msg/` directory of a ROS package. `.msg` files are composed of two parts: fields and constants.

### [2.1 Fields](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#id3)[](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#fields "Permalink to this headline")

Each field consists of a type and a name, separated by a space, i.e:

```text
fieldtype1 fieldname1
fieldtype2 fieldname2
fieldtype3 fieldname3

```

For example:

```text
int32 my_int
string my_string

```

[2.1.1 Field types](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#id4)[](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#field-types "Permalink to this headline")

Field types can be:

- a built-in-type

- names of Message descriptions defined on their own, such as “geometry_msgs/PoseStamped”

*Built-in-types currently supported:*

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

*Every built-in-type can be used to define arrays:*

| Type name               | C++                | Python         | DDS type       |
| ----------------------- | ------------------ | -------------- | -------------- |
| static array            | std::array<T, N>   | builtins.list* | T[N]           |
| unbounded dynamic array | std::vector        | builtins.list  | sequence       |
| bounded dynamic array   | custom_class<T, N> | builtins.list* | sequence<T, N> |
| bounded string          | std::string        | builtins.str*  | string         |

All types that are more permissive than their ROS definition enforce the ROS constraints in range and length by software

*Example of message definition using arrays and bounded types:*

```text
int32[] unbounded_integer_array
int32[5] five_integers_array
int32[<=5] up_to_five_integers_array

string string_of_unbounded_size
string<=10 up_to_ten_characters_string

string[<=5] up_to_five_unbounded_strings
string<=10[] unbounded_array_of_string_up_to_ten_characters each
string<=10[<=5] up_to_five_strings_up_to_ten_characters_each

```

[2.1.2 Field names](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#id5)[](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#field-names "Permalink to this headline")

Field names must be lowercase alphanumeric characters with underscores for separating words. They must start with an alphabetic character, they must not end with an underscore and never have two consecutive underscores.

[2.1.3 Field default value](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#id6)[](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#field-default-value "Permalink to this headline")

Default values can be set to any field in the message type. Currently default values are not supported for string arrays and complex types (i.e. types not present in the built-in-types table above, that applies to all nested messages)

Defining a default value is done by adding a third element to the field definition line, i.e:

```text
fieldtype fieldname fielddefaultvalue

```

For example:

```text
uint8 x 42
int16 y -2000
string full_name "John Doe"
int32[] samples [-200, -100, 0, 100, 200]

```

Note:

- string values must be defined in single `'` or double quotes `"`

- currently string values are not escaped

### [2.2 Constants](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#id7)[](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#constants "Permalink to this headline")

Each constant definition is like a field description with a default value, except that this value can never be changed programatically. This value assignment is indicated by use of an equal ‘=’ sign, e.g.

```text
constanttype CONSTANTNAME=constantvalue

```

For example:

```text
int32 X=123
int32 Y=-123
string FOO="foo"
string EXAMPLE='bar'

```

Note

Constants names have to be UPPERCASE

## [3. Service description specification](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#id8)[](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#service-description-specification "Permalink to this headline")

Services are described and defined in `.srv` files in the `srv/` directory of a ROS package.A service description file consists of a request and a response msg type, separated by ‘—’. Any two `.msg` files concatenated with a ‘—’ are a legal service description.

Here is a very simple example of a service that takes in a string and returns a string:

```text
string str
---
string str

```

We can of course get much more complicated (if you want to refer to a message from the same package you must not mention the package name):

```text
#request constants
int8 FOO=1
int8 BAR=2
#request fields
int8 foobar
another_pkg/AnotherMessage msg
---
#response constants
uint32 SECRET=123456
#response fields
another_pkg/YetAnotherMessage val
CustomMessageDefinedInThisPackage value
uint32 an_integer

```

You cannot embed another service inside of a service.

## [4. New features in ROS 2 interfaces](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#id9)[](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#new-features-in-ros-2-interfaces "Permalink to this headline")

The ROS 2 IDL is closely related to the [ROS 1 IDL](https://wiki.ros.org/msg). Most existing ROS 1 `.msg` and `.srv` files should be usable as-is with ROS 2. Atop ROS 1’s existing feature set, the ROS 2 IDL introduces some new features, namely:

- **bounded arrays**: Whereas the ROS 1 IDL allows unbounded arrays (e.g., `int32[] foo`) and fixed-size arrays (e.g., `int32[5] bar`), the ROS 2 IDL further allows bounded arrays (e.g., `int32[<=5] bat`). There are use cases in which it’s important to be able to place an upper bound on the size of an array without committing to always using that much space (e.g., in a real-time system in which you need to preallocate all memory that will be used during execution).

- **bounded strings**: Whereas the ROS 1 IDL allows unbounded strings (e.g., `string foo`), the ROS 2 IDL further allows bounded strings (e.g., `string<=5 bar`).

- **default values**: Whereas the ROS 1 IDL allows constant fields (e.g., `int32 X=123`), the ROS 2 IDL further allows default values to be specified (e.g., `int32 X 123`). The default value is used when constructing a message/service object and can be subsequently overridden by assigning to the field. You can also specify default values for action parts.

# 参考

[https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#interfaceconcept](https://docs.ros.org/en/galactic/Concepts/About-ROS-Interfaces.html#interfaceconcept)
