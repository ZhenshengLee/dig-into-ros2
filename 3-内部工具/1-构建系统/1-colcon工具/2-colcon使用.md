# 使用colcon工具

## 参考

[https://index.ros.org/doc/ros2/Tutorials/Colcon-Tutorial/](https://index.ros.org/doc/ros2/Tutorials/Colcon-Tutorial/)

[build - Build Packages — colcon  documentation](https://colcon.readthedocs.io/en/released/reference/verb/build.html)

[mixin - Manage mixins — colcon  documentation](https://colcon.readthedocs.io/en/released/reference/verb/mixin.html)

## 安装colcon

```text
sudo apt install python3-colcon-common-extensions
```

## 基础知识

A ROS workspace is a directory with a particular structure. Commonly there is a src subdirectory. Inside that subdirectory is where the source code of ROS packages will be located. Typically the directory starts otherwise empty.

colcon does out of source builds. By default it will create the following directories as peers of the src directory:

The build directory will be where intermediate files are stored. For each package a subfolder will be created in which e.g. CMake is being invoked.

The install directory is where each package will be installed to. By default each package will be installed into a separate subdirectory.

The log directory contains various logging information about each colcon invocation.

注意：devel文件夹没有了

## 包覆盖overlay

通过source的方式完成

In the root of the workspace, run colcon build. Since build types such as ament_cmake do not support the concept of the devel space and require the package to be installed, colcon supports the option --symlink-install. This allows the installed files to be changed by changing the files in the source space (e.g. Python files or other not compiled resourced) for faster iteration.

```text
colcon build --symlink-install

```

符号安装可以在安装的时候创建符号连接而不是复制文件

cmake中也可以借鉴这种方式，不去拷贝而是创建目录或文件的符号链接

## 测试

```text
colcon test

```

## source

When colcon has completed building successfully, the output will be in the install directory. Before you can use any of the installed executables or libraries, you will need to add them to your path and library paths. colcon will have generated bash/bat files in the install directory to help setup the environment. These files will add all of the required elements to your path and library paths as well as provide any bash or shell commands exported by packages.

source的目的就是添加目录，让系统能够找到install文件夹中的文件

```text
. install/setup.bash
```

## colcon选项

COLCON_IGNORE

添加cmake参数： `--cmake-args -DBUILD_TESTING=0`

构建单独的包

```text
colcon test --packages-select YOUR_PKG_NAME --ctest-args -R YOUR_TEST_IN_PKG

```

--merge-install

Use the `--install-base` as the install prefix for all packages instead of a package specific subdirectory in the install base.

--log-base LOG_BASE

--build-base BUILD_BASE

--install-base INSTALL_BASE

## colcon扩展-mixin

自定义subverb 来自定义命令

当前项目通过sh脚本自定义命令，也挺好的
