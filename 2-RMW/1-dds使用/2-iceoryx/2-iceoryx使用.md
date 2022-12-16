1.0.1版本

# 构建选项

```text
option(BINDING_C "Builds the C language bindings" ON)
option(BUILD_ALL "Build with all extensions and all tests" OFF)
option(BUILD_DOC "Build and generate documentation" OFF)
option(BUILD_SHARED_LIBS "Build iceoryx as shared libraries" OFF)
option(BUILD_STRICT "Build is performed with '-Werror'" OFF)
option(BUILD_TEST "Build all tests" OFF)
option(CCACHE "Use ccache when it's available" ON)
option(CLANG_TIDY "Execute Clang-Tidy" OFF)
option(COVERAGE "Build iceoryx with gcov flags" OFF)
option(DDS_GATEWAY "Builds the iceoryx dds gateway - enables internode communication via dds" OFF)
option(DOWNLOAD_TOML_LIB "Download cpptoml via the CMake ExternalProject module" ON)
option(EXAMPLES "Build all iceoryx examples" OFF)
option(INTROSPECTION "Builds the introspection client which requires the ncurses library with an activated terminfo feature" OFF)
option(ONE_TO_MANY_ONLY "Restricts communication to 1:n pattern" OFF)
option(ROUDI_ENVIRONMENT "Build RouDi Environment for testing, is enabled when building tests" OFF)
option(SANITIZE "Build with sanitizers" OFF)
option(TEST_WITH_ADDITIONAL_USER "Build Test with additional user accounts for testing access control" OFF)
option(TOML_CONFIG "TOML support for RouDi with dynamic configuration" ON)

```

# 传输支持

查看系统可用的shm大小

```text
df -H /dev/shm
```

# TOML配置

## 加载顺序

/etc/iceoryx/roudi_config.toml

/usr/local/etc/roudi_config_example.toml

```text
[general]
version = 1

[[segment]]

[[segment.mempool]]
size = 16448
count = 32768

16448bytes=16384 + 64 bytes of header
16384bytes=16k

```

```bash
[general]
version = 1

[[segment]]

[[segment.mempool]]
size = 128
count = 10000

[[segment.mempool]]
size = 1024
count = 1000

[[segment.mempool]]
size = 2048
count = 1000

[[segment.mempool]]
size = 4096
count = 1000

[[segment.mempool]]
size = 16448
count = 1000

[[segment.mempool]]
size = 131136
count = 200

[[segment.mempool]]
size = 527424
count = 50

[[segment.mempool]]
size = 1048640
count = 30

[[segment.mempool]]
size = 2097216
count = 20

[[segment.mempool]]
size = 4194368
count = 10

[[segment.mempool]]
size = 8388672
count = 5
```

size = 32896

count = 500

performance 使用的配置文件

```bash
# Adapt this config to your needs and rename it to e.g. roudi_config.toml
[general]
version = 1

[[segment]]

[[segment.mempool]]
size = 128
count = 10000

[[segment.mempool]]
size = 1024
count = 5000

[[segment.mempool]]
size = 16384
count = 1000

[[segment.mempool]]
size = 131072
count = 200

[[segment.mempool]]
size = 524288
count = 50

[[segment.mempool]]
size = 1048576
count = 30

[[segment.mempool]]
size = 4194304
count = 20

[[segment.mempool]]
size = 16777216
count = 20


```

## 使用方式

./iox-roudi -c /absolute/path/to/config/file.toml

The chunk size for the mempool needs to follow these restrictions:

1. Chunksize needs to be greater than the alignment

1. Chunksize needs to be a multiple of the alignment

# 安全组件

iceoryx提供大量安全组件使用

![](https://tcs.teambition.net/storage/312h25f6df185c5881b4f1f6377f63a0254a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDg0NiwiaWF0IjoxNjcxMTkwMDQ2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgyNWY2ZGYxODVjNTg4MWI0ZjFmNjM3N2Y2M2EwMjU0YSJ9.wXMchb6qB3MZ7AkJxxHHwavsY0aWuCPh-DU20B6qCGk&download=image.png "")

# 参考

[Home - iceoryx.io](https://iceoryx.io/v1.0.1/)
