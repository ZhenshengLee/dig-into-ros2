# 硬件依赖

根据硬件架构和Linux基础软件架构的不同，将相机驱动分为多个类型

- camera1394

- GigE IP Camera

- EthernetCamera

- USB Camera [usb_cam](http://wiki.ros.org/usb_cam) Unmaintained. Uses libavcodec for mjpeg.

- gPhoto照相机

- gmsl camera [__https://github.com/BeckerFelix/ros_gmsl_driver__](https://github.com/BeckerFelix/ros_gmsl_driver)

- 其他私有协议相机由供应商提供驱动

# 硬件独立

要求上层的软件层支持底层硬件接口，即Linux提供对应驱动，软件提供对应接口

- [cv_camera](http://wiki.ros.org/cv_camera) Uses OpenCV. Supports camera_info_manager.

- [uvc_camera](http://wiki.ros.org/uvc_camera) Unmaintained. Supports camera_info_manager.

- gstreamer [gstreamer camera driver](http://wiki.ros.org/gscam)

- v4l2 [__https://gitlab.com/boldhearts/ros2_v4l2_camera__](https://gitlab.com/boldhearts/ros2_v4l2_camera)

-

# ROS软件架构

![](https://tcs.teambition.net/storage/312eab484dabfff62e7f9cf8d64735060b67?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA1Mjc5MSwiaWF0IjoxNjcxNDQ3OTkxLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmVhYjQ4NGRhYmZmZjYyZTdmOWNmOGQ2NDczNTA2MGI2NyJ9.8Rlx_Zexfhbd8M5NsKFBdDw_W83XboFHveqmRQ5Edak&download=image.png "")

# 外部依赖

camera_calibration_parsers, camera_info_manager

# 参考

[Sensors/Cameras - ROS Wiki](http://wiki.ros.org/Sensors/Cameras)
