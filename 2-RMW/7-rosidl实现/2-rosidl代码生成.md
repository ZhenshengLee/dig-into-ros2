# 接口生成流程

Constraint Checking

Generated code for messages is only guaranteed to enforce constraints when a message is published. For example, there might not be an error when a field constrained to be a fixed size array is assigned an array with the wrong size. This could be inconsistent across client library implementations depending on language features and performance cost. In the future any client library may add additional constraint checking when a field is set or modified. Users of generated message code should assume constraints could be enforced at any time to be compatible with such a change.

### Type Mapping

The next table defines how IDL types are mapped to the following programming languages:

- C11 (or higher)

- C++11 (or higher)

- Python 3 (or higher)

| IDL type    | C type        | C++ type      | Python type         |
| ----------- | ------------- | ------------- | ------------------- |
| float       | float         | float         | float               |
| double      | double        | double        | float               |
| long double | long double   | long double2  | float               |
| char        | unsigned char | unsigned char | str with length 1   |
| wchar       | char16_t      | char16_t      | str with length 1   |
| boolean     | _Bool         | bool          | bool                |
| octet       | unsigned char | std::byte1    | bytes with length 1 |
| int8        | int8_t        | int8_t        | int                 |
| uint8       | uint8_t       | uint8_t       | int                 |
| int16       | int16_t       | int16_t       | int                 |
| uint16      | uint16_t      | uint16_t      | int                 |
| int32       | int32_t       | int32_t       | int                 |
| uint32      | uint32_t      | uint32_t      | int                 |
| int64       | int64_t       | int64_t       | int                 |
| uint64      | uint64_t      | uint64_t      | int                 |

1. If std::byte is not available then unsigned char is used instead.

1. Users should research the support for `long double` with their choice of middleware and platform. For example, it is only [64 bits when using Visual Studio](https://docs.microsoft.com/en-us/cpp/c-language/type-long-double?view=vs-2017).

The next table defines how IDL templated and constructed types are mapped to the programming languages (unless specified otherwise below). When specified `T` is either one of the types above or an IDL struct. `N` is the upper limit of a bounded type.

| IDL type       | C type                          | C++ type         | Python type |
| -------------- | ------------------------------- | ---------------- | ----------- |
| T[N]           | T[N]                            | std::array<T, N> | list        |
| sequence<T>    | struct {size_t, T * }           | std::vector<T>   | list        |
| sequence<T, N> | struct {size_t, T * }, size_t N | std::vector<T>   | list        |
| string         | char *                          | std::string      | str         |
| string<N>      | char *                          | std::string      | str         |
| wstring        | char16_t *                      | std::u16string   | str         |
| wstring<N>     | char16_t *                      | std::u16string   | str         |

These array and sequence types have special mappings. If a cell is blank then the default mapping is used.

| IDL type                  |                                                                                         | C type | C++ type | Python type                                   |                                                                                                                                                  |
| ------------------------- | --------------------------------------------------------------------------------------- | ------ | -------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| T[N]                      | for numeric types T:float, double,int8, uint8,int16, uint16,int32, uint32,int64, uint64 | -      | -        | numpy.ndarray(  shape=(N, ),  dtype=numpy.DT) | where DT is determined by T:float -> float32,double -> float64,intX -> intX,uintX -> uintX                                                       |
| sequence<T>sequence<T, N> | for numeric types T:float, double,int8, uint8,int16, uint16,int32, uint32,int64, uint64 | -      | -        | array.array(typecode=TC)                      | where TC is determined by T:float -> f,double -> d,int8 -> b, uint8 -> B,int16 -> h, uint16 -> H,int32 -> l, uint32 -> L,int64 -> q, uint64 -> Q |
| octet[N]                  |                                                                                         | -      | -        | bytes                                         |                                                                                                                                                  |
| sequence<octet>           |                                                                                         | -      | -        | bytes                                         |                                                                                                                                                  |
| sequence<octet, N>        |                                                                                         | -      | -        | bytes                                         |                                                                                                                                                  |

### Standardized Annotations

8.3.2.1 @key Annotation

While the key annotation doesn’t affect the generated ROS data types directly it is being passed through to the (DDS) vendor specific code generators.

8.3.3.1 @default Annotation

The default value is being used to initialize struct members.

8.3.5.1 @verbatim Annotation

When the value `comment` is passed as the `language` parameter the text parameter is used as the docblock for the annotated element across all supported programming languages.

# 参考

[Generated C++ interfaces](https://design.ros2.org/articles/generated_interfaces_cpp.html)

[https://design.ros2.org/articles/idl_interface_definition.html](https://design.ros2.org/articles/idl_interface_definition.html)
