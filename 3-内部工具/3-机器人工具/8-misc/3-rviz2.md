# 测试

```text
ros2 run v4l2_camera v4l2_camera_node
rviz2
ros2 run rqt_image_view rqt_image_view

```

# trouble-shoot

```text
rm -rf ~/.rviz
ros2 run rviz2 rviz2 -l
```

![](https://tcs.teambition.net/storage/312h18b6bf6d352014f7ef4e9a3b0f045a7b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg0Njk0MCwiaWF0IjoxNjcxMjQyMTQwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgxOGI2YmY2ZDM1MjAxNGY3ZWY0ZTlhM2IwZjA0NWE3YiJ9.DQuAX8PWok3CM-m9PzbGp0NrXTyXLkLE4NjSKO527JM&download=image.png "")

## 查询libgl正常工作

```bash
glxgears
glxinfo | grep 'OpenGL renderer'

# "Copy", "PBuffer", or "FBO"
export OGRE_RTT_MODE=Copy

```

## 关闭硬件加速

```text
export LIBGL_ALWAYS_SOFTWARE=1
ros2 run rviz2 rviz2 -l
```

# 参考

[`rviz2` cannot show image and output error msg. · Issue #50 · ros2/rmw_iceoryx](https://github.com/ros2/rmw_iceoryx/issues/50)

[rviz Segmentation fault (core dumped)](https://stackoverflow.com/questions/37627489/rviz-segmentation-fault-core-dumped)

[rviz/Troubleshooting - ROS Wiki](https://wiki.ros.org/rviz/Troubleshooting)
