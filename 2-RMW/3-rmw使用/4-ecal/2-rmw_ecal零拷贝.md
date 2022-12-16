eCAL也可以使用iceoryx的零拷贝，但是默认不开启该编译选项

iceoryx零拷贝目前还有bug

# 概述

single-write / multiple read pattern

eCAL的shm为什么性能没有iceoryx好？因为它做了一次拷贝，所以payload>32KB之后性能开始产生差异

拷贝一次的好处就是解耦

eCAL也有0次拷贝，0copy，不过由于目前读取是要加锁的，所以在回调中必须尽快返回，不然会影响数据写入。

目前已经支持了

# eCAL-shm

有了，默认开启

# eCAL-zc

有了，默认开启，在ini配置中修改

有自己的zero copy，也有iceoryx的zero copy，均不需要配置

版本要求5.10以上，因为2月份才把bug解除

# rmw_ecal支持

查看api

![](https://tcs.teambition.net/storage/312gc65a8cebfe98fe6ef25dc5d9d7e249cd?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODY4NSwiaWF0IjoxNjcxMTkzODg1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdjNjVhOGNlYmZlOThmZTZlZjI1ZGM1ZDlkN2UyNDljZCJ9.CRAHblKFpR_PMs1NV7-Rcc6NPhBeVw42d1JHgMmfy5g&download=image.png "")

rmw只支持通用的api供rclcpp使用，ecal内部自行判断决定

# rclcpp支持

默认api支持零拷贝

# 支持情况

## ecal+iceoryx

无法启动gazebo，造成gazebo插件初始化失败，urdf不能显示

## ecal原生

目前tf可以显示，urdf不能正常显示，工具正常使用，pointcloud可以显示，rosbag2可以使用

# 性能

# 参考

[Enable zero copy shared memory transport mode. by schilasky · Pull Request #322 · continental/ecal](https://github.com/continental/ecal/pull/322)

[Side Question: Usage of eclipse-iceoryx / iceoryx ? · Issue #326 · continental/ecal](https://github.com/continental/ecal/issues/326)
