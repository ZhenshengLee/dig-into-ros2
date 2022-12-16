并不是所有的特性都要用ament，尽量用cmake原生的指令保证可移植性。

当然ament是独立于ros2的单独工具，基于cmake增加了很多封装方便ros2包编译和处理包间依赖。

# 概念

ament是统一各个ros包，兼容各种类型构建系统的上层构建系统，任何一个package，只要遵循ament的规约，无论是cpp还是python，都可以很方便的融入ros构建系统。

由于ros工程大多数是cpp，ament借助cmake来支撑cpp工程的编译，由于cmake本身具有多种语言的兼容性和扩展性，所以ament通过cmake提供对python的支持

ament包括ament_cmake, ament_tools, 以及一些关于资源索引的设计，ament_tools作为上层构建系统的执行器，后面独立出去称为了colcon项目，所以最主要的就是ament_cmake以及ament_resource_index，所以现在的ament又叫做ament_cmake

## ament包

只要包含遵循ament规约的文件，就可以称为ament包，目前ros2的包管理是使用ament的。

ament包的种类包括：

CMake package

- Any [package](https://docs.ros.org/en/rolling/Glossary.html#term-package) containing a plain CMake project and a [package.xml](https://docs.ros.org/en/rolling/Concepts/About-Build-System.html#term-package.xml) manifest file.

ament CMake package

- A [CMake package](https://docs.ros.org/en/rolling/Concepts/About-Build-System.html#term-CMake-package) that also follows the `ament` packaging guidelines.

Python package

- Any [package](https://docs.ros.org/en/rolling/Glossary.html#term-package) containing a [setuptools](https://pypi.org/project/setuptools/) based Python project and a [package.xml](https://docs.ros.org/en/rolling/Concepts/About-Build-System.html#term-package.xml) manifest file.

ament Python package

- A [Python package](https://docs.ros.org/en/rolling/Concepts/About-Build-System.html#term-Python-package) that also follows the `ament` packaging guidelines.

# 基础

## ament包的定义-描述文件

package.xml这个文件包含所有依赖信息，colcon工具读取来寻找所有的包，安装所需要的依赖

cmakelists包含基于cmake的编译指令，其中使用大量的ament宏指令

## ament_package()

这句话很重要，安装package.xml，config和目标文件，保证该包能够被find_package找到，所以应该在cmakelists的最后被调用

ament_package()每个包中只能出现一次

ament_package() installs the package.xml, registers the package with the ament index, and installs config (and possibly target) files for CMake so that it can be found by other packages using find_package.

安装package.xml，注册索引信息，安装配置和目标文件到cmake，以便其他包可以找到该包内在目标

 Since ament_package() gathers a lot of information from the CMakeLists.txt it should be the last call in your CMakeLists.txt. Although it is possible to follow calls to ament_package() by calls to install functions copying files and directories, it is simpler to just keep ament_package() the last call.

这句话注册包信息，安装ament设置的可执行文件和库，~~所以写了这句话后不需要再写install了~~

ament_package can be given additional arguments:

`ament_package` can be given additional arguments:

- `CONFIG_EXTRAS`: a list of CMake files (`.cmake` or `.cmake.in` templates expanded by `configure_file()`) which should be available to clients of the package. For an example of when to use these arguments, see the discussion in [Adding resources](https://docs.ros.org/en/humble/How-To-Guides/Ament-CMake-Documentation.html#adding-resources). For more information on how to use template files, see [the official documentation](https://cmake.org/cmake/help/v3.5/command/configure_file.html).

  - 文件对于所有依赖该包的包，都是可见的

- `CONFIG_EXTRAS_POST`: same as `CONFIG_EXTRAS`, but the order in which the files are added differs. While `CONFIG_EXTRAS` files are included before the files generated for the `ament_export_*` calls the files from `CONFIG_EXTRAS_POST` are included afterwards.

Instead of adding to `ament_package`, you can also add to the variable `${PROJECT_NAME}_CONFIG_EXTRAS` and `${PROJECT_NAME}_CONFIG_EXTRAS_POST` with the same effect. The only difference is again the order in which the files are added with the following total order:

- files added by `CONFIG_EXTRAS`

- files added by appending to `${PROJECT_NAME}_CONFIG_EXTRAS`

- files added by appending to `${PROJECT_NAME}_CONFIG_EXTRAS_POST`

- files added by `CONFIG_EXTRAS_POST`

## 添加文件和头文件

```text
target_include_directories(my_target
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)

```

This adds all files in the folder ${CMAKE_CURRENT_SOURCE_DIR}/include to the public interface during build time and all files in the include folder (relative to ${CMAKE_INSTALL_DIR}) when being installed.

In principle, using generator expressions here is not necessary if both folders are called include and top-level with respect to ${CMAKE_CURRENT_SOURCE_DIR} and ${CMAKE_INSTALL_DIR}, but it is very common.

## 添加依赖

两种方式定义依赖

```text
find_package(Eigen3 REQUIRED)
ament_target_dependencies(my_target Eigen3)

```

It includes the necessary headers and libraries and their dependencies to be correctly found by the project. It will also ensure that the include directories of all dependencies are ordered correctly when using overlay workspaces.

```text
find_package(Eigen3 REQUIRED)
target_link_libraries(my_target Eigen3::Eigen)

```

有何不同，ament_target_dependencies it might not correctly order the dependencies when using overlay workspaces.

The recommended way in modern CMake is to only use targets, exporting and linking against them. CMake targets are namespaced, similar to C++. For instance, Eigen3 defines the target Eigen3::Eigen.

At least until Crystal Clemmys target names are not supported in the ament_target_dependencies macro. Sometimes it will be necessary to call the target_link_libaries CMake function. In the example of Eigen3, the call should then look like

## 编译库

```text
ament_export_targets(export_my_library HAS_LIBRARY_TARGET)
ament_export_dependencies(some_dependency)

install(
  DIRECTORY include/
  DESTINATION include
)

install(
  TARGETS my_library
  EXPORT export_my_library
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

```

Here is what’s happening in the snippet above:

- The `ament_export_targets` macro exports the targets for CMake. This is necessary to allow your library’s clients to use the `target_link_libraries(client my_library::my_library)` syntax. `ament_export_targets` can take an arbitrary list of targets named as `EXPORT` in an install call and an additional option `HAS_LIBRARY_TARGET`, which adds potential libraries to environment variables.

- The `ament_export_dependencies` exports dependencies to downstream packages. This is necessary so that the user of the library does not have to call `find_package` for those dependencies, too.

- The first `install` commands installs the header files which should be available to clients.

Warning

Calling `ament_export_targets`, `ament_export_dependencies`, or other ament commands from a CMake subdirectory will not work as expected. This is because the CMake subdirectory has no way of setting necessary variables in the parent scope where `ament_package` is called.

- The last large install command installs the library. Archive and library files will be exported to the lib folder, runtime binaries will be installed to the bin folder and the path to installed headers is `include`.

Note

Windows DLLs are treated as runtime artifacts and installed into the `RUNTIME DESTINATION` folder. It is therefore advised to not leave out the `RUNTIME` install even when developing libraries on Unix based systems.

- Regarding the `include directory`, the install command only adds information to CMake, it does not actually install the includes folder. This is done by copying the headers via `install(DIRECTORY <dir> DESTINATION <dest>)` as described above.

- The `EXPORT` notation of the install call requires additional attention: It installs the CMake files for the `my_library` target. It is named exactly like the argument in `ament_export_targets` and could be named like the library. However, this will then prohibit using the `ament_target_dependencies` way of including your library. To allow for full flexibility, it is advised to prepend the export target with something like `<target>Targets`.

- All install paths are relative to `CMAKE_INSTALL_PREFIX`, which is already set correctly by colcon/ament

There are two additional functions which can be used but are superfluous for target based installs:

```text
ament_export_include_directories(include)
ament_export_libraries(my_library)

```

The first macro marks the directory of the exported include directories (this is achieved by `INCLUDES DESTINATION` in the target `install` call). The second macro marks the location of the installed library (this is done by the `HAS_LIBRARY_TARGET` argument in the call to `ament_export_targets`).

Some of the macros can take different types of arguments for non-target exports, but since the recommended way for modern Make is to use targets, we will not cover them here. Documentation of these options can be found in the source code itself.

#

## 编译和链接选项

```

target_compile_options(my_target PRIVATE -Wall)

# ros2推荐使用下面的

add_compile_options(-Wall)

```

# 使用外部工具

## 测试和静态检查

```

if(BUILD_TESTING)

  find_package(ament_gtest)

  ament_add_gtest(<tests>)

endif()

```

```

find_package(ament_lint_auto REQUIRED)

ament_lint_auto_find_test_dependencies()

```

This will run linters as defined in the package.xml. It is recommended to use the set of linters defined by the package ament_lint_common. The individual linters included there, as well as their functions, can be seen in the ament_lint_common docs.

```

# 功能扩展

## [Adding a function/macro to ament](https://docs.ros.org/en/humble/How-To-Guides/Ament-CMake-Documentation.html#id12)

Extending ament will often times mean that you want to have some functions available to other packages. The best way to provide the macro to client packages is to register it with ament.

This can be done by appending the `${PROJECT_NAME}_CONFIG_EXTRAS` variable, which is used by `ament_package()` via

```text
list(APPEND ${PROJECT_NAME}_CONFIG_EXTRAS
  path/to/file.cmake"
  other/pathto/file.cmake"
)

```

Alternatively, you can directly add the files to the `ament_package()` call:

```text
ament_package(CONFIG_EXTRAS
  path/to/file.cmake
  other/pathto/file.cmake
)
```

## [Adding to extension points](https://docs.ros.org/en/humble/How-To-Guides/Ament-CMake-Documentation.html#id13)

In addition to simple files with functions that can be used in other packages, you can also add extensions to ament. Those extensions are scripts which are executed with the function which defines the extension point. The most common use-case for ament extensions is probably registering rosidl message generators: When writing a generator, you normally want to generate all messages and services with your generator also without modifying the code for the message/service definition packages. This is possible by registering the generator as an extension to `rosidl_generate_interfaces`.

As an example, see

```text
ament_register_extension(
  "rosidl_generate_interfaces"
  "rosidl_generator_cpp"
  "rosidl_generator_cpp_generate_interfaces.cmake")

```

which registers the macro `rosidl_generator_cpp_generate_interfaces.cmake` for the package `rosidl_generator_cpp` to the extension point `rosidl_generate_interfaces`. When the extension point gets executed, this will trigger the execution of the script `rosidl_generator_cpp_generate_interfaces.cmake` here. In particular, this will call the generator whenever the function `rosidl_generate_interfaces` gets executed.The most important extension point for generators, aside from `rosidl_generate_interfaces`, is `ament_package`, which will simply execute scripts with the `ament_package()` call. This extension point is useful when registering resources (see below).`ament_register_extension` is a function which takes exactly three arguments:

- `extension_point`: The name of the extension point (most of the time this will be one of `ament_package` or `rosidl_generate_interfaces`)

- `package_name`: The name of the package containing the CMake file (i.e. the project name of the project where the file is written to)

- `cmake_filename`: The CMake file executed when the extension point is run

Note

It is possible to define custom extension points in a similar manner to `ament_package` and `rosidl_generate_interfaces`, but this should hardly be necessary.

## [Adding extension points](https://docs.ros.org/en/humble/How-To-Guides/Ament-CMake-Documentation.html#id14)[](https://docs.ros.org/en/humble/How-To-Guides/Ament-CMake-Documentation.html#adding-extension-points "Permalink to this heading")

Very rarely, it might be interesting to define a new extension point to ament.

Extension points can be registered within a macro so that all extensions will be executed when the corresponding macro is called. To do so:

- Define and document a name for your extension (e.g. `my_extension_point`), which is the name passed to the `ament_register_extension` macro when using the extension point.

- In the macro/function which should execute the extensions call:

```text
ament_execute_extensions(my_extension_point)

```

Ament extensions work by defining a variable containing the name of the extension point and filling it with the macros to be executed. Upon calling `ament_execute_extensions`, the scripts defined in the variable are then executed one after another.

## [Adding resources](https://docs.ros.org/en/humble/How-To-Guides/Ament-CMake-Documentation.html#id15)[](https://docs.ros.org/en/humble/How-To-Guides/Ament-CMake-Documentation.html#adding-resources "Permalink to this heading")

Especially when developing plugins or packages which allow plugins it is often essential to add resources to one ROS package from another (e.g. a plugin). Examples can be plugins for tools using the pluginlib.

This can be achieved using the ament index (also called “resource index”).

# 参考

[ament_cmake user documentation — ROS 2 Documentation: Foxy  documentation](https://docs.ros.org/en/foxy/Guides/Ament-CMake-Documentation.html)

[https://index.ros.org/doc/ros2/Tutorials/Ament-CMake-Documentation/](https://index.ros.org/doc/ros2/Tutorials/Ament-CMake-Documentation/)

package.xml中定义依赖和元文件资源，colcon系统根据内容确定正确的构建顺序，为bloom提供顺序

cmakelist.txt中包含编译可执行文件和库的指令
