# rosdep国内问题

```text
sudo pip3 install rosdepc

sudo rosdepc init
rosdepc update

```

# bloom方法

## 前提

所有依赖的包都要在本地构建好或者在rosdep可用

所有的依赖项在package.xml必须明确完整的定义

## 依赖

```bash
sudo apt install python3-bloom python3-rosdep fakeroot

$ sudo rosdep init
$ rosdep update

```

## 构建debian

```bash
$ cd /path/to/pkg_source  # this should be the directory that contains the package.xml
source /opt/ros/galactic-dev/setup.bash
$ bloom-generate rosdebian
# 默认版本
$ fakeroot debian/rules binary
# 生成deb
```

# cmake package方法

略

# 参考

[Releasing a ROS 2 package with bloom — ROS 2 Documentation: Foxy  documentation](https://docs.ros.org/en/foxy/How-To-Guides/Releasing-a-ROS-2-package-with-bloom.html)

[How to generate .deb of ROS package step by step outside ros buildfarm - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/241663/how-to-generate-deb-of-ros-package-step-by-step-outside-ros-buildfarm/)
