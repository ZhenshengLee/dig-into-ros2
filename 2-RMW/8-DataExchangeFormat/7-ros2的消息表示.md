已经有了dds，应该按照dds idl来表示消息，但是也有一些厂家研制了ros2到protobuf等序列化方式的接口，这里补充学习一下eCAL的技术。

序列化是将数据结构转化为字节流，方便存储和传输。其主要目的是保存对象的状态，方便复现。

序列化是需要拷贝的

## 参考

[https://en.wikipedia.org/wiki/Serialization](https://en.wikipedia.org/wiki/Serialization)

[ROS 2 Message Research](https://design.ros2.org/articles/serialization.html)

[https://en.wikipedia.org/wiki/Comparison_of_data-serialization_formats](https://en.wikipedia.org/wiki/Comparison_of_data-serialization_formats)

## 概述

消息是ROS非常重要的组成部分，这篇文章聚焦于研究消息api，在性能和灵活性上取得平衡.

当前ros1的消息是数据对象，member-based。消息定义不是十分灵活，官方提供了c++， Python，lisp的串行工具。

### 改进准则

api是兼容未来需求的

串行是可选而不是强制的

messages are actually serialized (rather than passed by reference)

使用现有工具

### 改进空间

- 支持固定长度消息

  - 这样可以避免动态分配内存

- 可选字段，默认值

  - 在ros1中必须强制指定所有字段

- 增加数据类型，字典

## 设计决策

略

## 现有方案

Pipeline A

The message used by the userland code stores its data directly. For each communication channel the message data is then copied into the serialization specific message representation. The serialization library will perform the serialization into the wire format from there.

![](https://tcs.teambition.net/storage/31273928fa60d2e8392bb3c3b9cf5da74c36?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTMxNywiaWF0IjoxNjcxMTk0NTE3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjczOTI4ZmE2MGQyZTgzOTJiYjNjM2I5Y2Y1ZGE3NGMzNiJ9.0qMeNIxNWx62ZaS4SVbFVSM5PkzjwqQrwvsmvDrObys&download=blob.png "")

Pipeline B

The message fields can be serialized directly into the wire format using custom code. While this avoids the extra data copy it requires a significant effort for implementing the custom serialization routine.

![](https://tcs.teambition.net/storage/31278d3bdb7e42ab8e2cdf10f290140e9ac4?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTMxNywiaWF0IjoxNjcxMTk0NTE3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjc4ZDNiZGI3ZTQyYWI4ZTJjZGYxMGYyOTAxNDBlOWFjNCJ9.ufWXcNWMgeOjiU9bWH47TTLx1NhGOb3RJj7N6OGcsRM&download=blob.png "")

Pipeline C

The message delegates the data storage to an internally held storage backend, e.g. the serialization library specific message representation. Since the data is stored directly in the serialization library specific representation copying the data before serializing it is not necessary anymore.

This assumes that the API of the serialization library specific representation can be wrapped inside the ROS message API (see [Technical Issues -> Variances in field types](https://design.ros2.org/articles/serialization.html#variances_in_field_types)).

![](https://tcs.teambition.net/storage/31271ca87d42cd8907183ad346c018874445?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTMxNywiaWF0IjoxNjcxMTk0NTE3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjcxY2E4N2Q0MmNkODkwNzE4M2FkMzQ2YzAxODg3NDQ0NSJ9.VwA0p5bzTKOCcrn9UXa3a_AH4GvZu7MXHH1s_V68vlM&download=blob.png "")

## 附录：protobuf安装

一般apt安装即可，此处为加上-fPIC选项

```bash
./autogen.sh
./configure CFLAGS="-fPIC"  CXXFLAGS="-fPIC" --prefix=/opt/protobuf/protobuf
make -j10
sudo make install
```

[configure –prefix 和 make install DESTDIR= 还有make install prefix 什么区别 – kamuszhou is a geek](https://blog.ykyi.net/2014/01/configure-prefix-%E5%92%8C-make-install-destdir-%E8%BF%98%E6%9C%89make-install-prefix-%E4%BB%80%E4%B9%88%E5%8C%BA%E5%88%AB/)

make install prefix=/foo/bar/baz

和 make install DESTDIR的区别在于，前者不会创建prefix后面指定的所有目录层级，而DESTDIR会。

例如：

./configure –prefix=usr/local && make && sudo make install prefix=/usr/local/stow/foo 会把程序安装在/usr/local/stow/bin目录下。

而: make install DESTDIR=/usr/local/stow/foo，则把程序安装在/usr/local/stow/foo/usr/local/bin下面。

./configure –prefix=*

确定了程序会被包装到哪里，以及，程序将在什么位置搜寻运行需要用到的相关的文件。如果你在一台机器上编译，然后仍然在这台机器上运行，那使用configure –prefix就是你想要的了。

make install DESTDIR=*

把程序安装到一个临时用的目录，这个目录不是程序以后将要运行的目录。例如，这个功能被deb包的打包人员使用。打包人员使用此功能时，并不真正把程序安装到–prefix指定的目录下。因为他可能已经安装了不同的版本，他不想搞乱他的环境，或者他没有boot权限，没有办法把程序安装到/bin,/sbin之类目录。 –prefix=/usr，让程序运行时认为被安装在/usr，然后make install DESTDIR=debian/tmp则实际把程序安装到debian/tmp。
