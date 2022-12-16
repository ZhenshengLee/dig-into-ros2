ament文件资源管理

## 参考

[https://github.com/ament/ament_cmake/blob/master/ament_cmake_core/doc/resource_index.md](https://github.com/ament/ament_cmake/blob/master/ament_cmake_core/doc/resource_index.md)

## 概念

package_resource_indexer用于软件包内容发布的分类和索引

这个项目提供两个能力，第一个是软件包注册资源类型，第二个运行时索引资源的能力。

资源表示任何软件包的资产，索引表示一些查询行为，如“这个包有没有安装？”，“哪一个包提供了指定的插件？”，“某包提供了哪些消息和服务？”。

这个项目目标不是分类和索引具体的资源，而是提供元信息，关于资源在某个软件包中，提供包的路径信息。

这些需求来自于ROS的发现系统

## 设计

注册包的元信息，使用文件系统。这样设计有两个好处，（不想翻译了）

### 文件系统布局

我们定义一个众所周知的地址，`<prefix>/share/ament_index/resource_index`，这个空间防止所有的包和所有的文件，我们称这个为resource index，在这个资源索引中是一个文件系统结构，有一些文件结构，每一种文件结构成为一种资源类型。每一个包提供一个资源类型标识符，是一个文件，成为“marker file”

```

<prefix>

    `-- share

        `-- ament_index

            `-- resource_index

                `-- packages

                    `-- foo # empty file

                    `-- bar # empty file

                    `-- baz # empty file

```

有了这种类型结构，我们就可以查询哪个包提供了rviz的 plugin

```

<prefix>

    `-- share

        `-- ament_index

            `-- resource_index

                |-- packages

                | `-- foo

                | `-- bar

                | `-- baz

                |-- plugins.rviz.display

                    `-- foo

```

## 此系统对比于ros的优势

总的来讲，在包数量很大的时候，以及查询特别复杂信息情况下，会有速度上的优势

```

Currently in ROS, this requires that all packages are discovered first and then each manifest file for each package must be parsed (package.xml or manifest.xml) and then for each package zero to many plugin.xml files must be parsed. For this example system, the package.xml file and plugin.xml files may still need to be parsed, but instead of discovering all packages, and parsing all package.xml files this system can already narrow down the packages which need to be considered. In the above example that only saves us from parsing two out of three packages, but in the presence of hundreds of packages, this could be parsing one or two package manifests versus parsing hundreds. It also prevents us from parsing additional, unrelated, plugin.xml files, so the win for this type of system with respect to plugin discovery is potentially huge. For other resources, the speed up at this point is probably minimal, but other examples might include "Which packages have defined message files?" or "Which packages have launch files?". These types of queries can potentially speed up command line tools considerably.

```

## resource index

每一个目录下，都会有一个文件，resource_index

`<prefix>/share/ament_index/resource_index`

## resource types

resource type作为文件夹来表示，由marker file来表示

## marker files

文件中的内容是没规范的，但是被系统用来识别资源类型

以plugin包为例子，需要一个plugin.xml

包的话，就需要一个package.xml

不仅可以看文件名字，文件中的内容也是可以约定好来帮助查询信息的
