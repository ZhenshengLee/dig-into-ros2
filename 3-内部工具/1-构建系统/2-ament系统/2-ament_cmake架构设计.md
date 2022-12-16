# 参考

[__https://docs.ros.org/en/rolling/Concepts/About-Build-System.html#term-ament-package__](https://docs.ros.org/en/rolling/Concepts/About-Build-System.html#term-ament-package)

# ament_cmake

这儿的ament_cmake指的是ament基于cmake进行构建，在xml描述文件中有<build_type>ament_cmake</build_type>

里面的包是极度模块化的，但是有一个起点包叫做ament_cmake，用户包只要依赖这个包就可以获得所有其他包的功能

## 门面包

- `ament_cmake`

  - aggregates all other [packages](https://docs.ros.org/en/rolling/Glossary.html#term-package) in this repository, users need only to depend on this

  - 集合所有功能

## 语法糖包

- `ament_cmake_auto`

  - provides convenience CMake functions which automatically handle a lot of the tedious parts of writing a [package](https://docs.ros.org/en/rolling/Glossary.html#term-package)’s `CMakeLists.txt` file

  - 语法糖，很多自动化的功能

## 核心包

- `ament_cmake_core`

  - provides all built-in core concepts for `ament`, e.g. __environment hooks, resource indexing, symbolic linking install and others__

  - 核心概念！包括环境变量钩子，资源索引，符号连接安装

## cmake集成包

往往基于cmake来做隔离

- `ament_cmake_test`

  - aggregates different kinds of tests, e.g. gtest and nosetests, under a single target using [CTest](https://cmake.org/Wiki/CMake/Testing_With_CTest)

  - 通过ctest来集成各种单元测试工具

- `ament_cmake_python`

  - provides CMake functions for [packages](https://docs.ros.org/en/rolling/Glossary.html#term-package) that contain Python code

  - see the [ament_cmake_python user documentation](https://docs.ros.org/en/rolling/How-To-Guides/Ament-CMake-Python-Documentation.html)

## 底层包

为具体的实现

- `ament_cmake_gmock`

  - adds convenience functions for making gmock based unit tests

  - 对gmock的封装

- `ament_cmake_gtest`

  - adds convenience functions for making gtest based automated tests

  - 对gtest的封装

- `ament_cmake_nose`

  - adds convenience functions for making nosetests based python automated tests

  - 对python测试的封装

## 底层宏包

具体的cmake宏设计被分解成一个一个包！

# ament_package

The parser for the manifest files in the ament buildsystem.

是一个命令包，用来执行的？

# ament_lint

设计思路与ament_cmake一致，而且大部分也是基于cmake来做

定义一种机制来执行各种linter工具，其机制基于开源项目cmake_lint，其不属于cmake官方[__https://github.com/richq/cmake-lint__](https://github.com/richq/cmake-lint)

## 门面包

- `ament_lint`

  - aggregates all other [packages](https://docs.ros.org/en/rolling/Glossary.html#term-package) in this repository, users need only to depend on this

  - 集合所有功能

## 语法糖包

- `ament_lint_auto`

  - provides convenience CMake functions which automatically handle a lot of the tedious parts of writing a [package](https://docs.ros.org/en/rolling/Glossary.html#term-package)’s `CMakeLists.txt` file

  - 语法糖，很多自动化的功能

## 核心包

- ament_lint_cmake

  - 集成cmake和cmake_lint

- `ament_cmake_lint_cmake`

  - 使用cmake-lint的机制

## cmake集成包

往往基于cmake来做隔离

- `ament_cmake_cppcheck`

  - cppcheck，指定cppcheck使用的机制

## 底层包

为具体的实现

- `ament_cppcheck`

  - 对cppcheck的封装, 基于python使用cppcheck工具进行lint

# ament_cmake和ament_cmake_ros的区别
