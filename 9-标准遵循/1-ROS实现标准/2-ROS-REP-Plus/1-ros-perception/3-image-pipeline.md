# vision_opencv

## cv_bridge

opencv 格式转换成ROS消息格式

## image_geometry

使用cameraInfo的工具类，最主要的作用就是将三维点，通过内参投影到image中

图像几何变换, 相机模型相关操作

简化图像的几何操作, 使用camerainfo中的参数, 使用这个库可以免除图像校正的开发难度, 而且这些操作开发起来非常难.

```

The camera parameters in CameraInfo are for a full-resolution image; region-of-interest alone significantly complicates the creation of rectification maps and requires adjusting the projection matrix. Adding options such as subsampling (binning) to CameraInfo would further complicate the correct interpretation of the corresponding Images. Using image_geometry simplifies and future-proofs imaging code.

In order to maintain invariance, the CameraModel classes offer read-only access to specific parameters and matrices. Setting a CameraModel can only be performed with full information using the fromCameraInfo() functions.

```

使用opencv相机模型, 包括针孔相机模型pinhole,

### 应用: 将tf坐标系投影到图像中

cam_model_.fromCameraInfo(info_msg);

uv = cam_model_.project3dToPixel(pt_cv);

```

Here we fill our PinholeCameraModel using the information from the CameraInfo message. Note that cam_model_ is a member variable rather than a local one. In the common case that the camera parameters do not change, updating a previously filled camera model is very cheap.

```

```

tf::Point pt = transform.getOrigin();

      cv::Point3d pt_cv(pt.x(), pt.y(), pt.z());

      cv::Point2d uv;

      uv = cam_model_.project3dToPixel(pt_cv);

We extract the origin of the requested frame in the camera optical frame and store it in a cv::Point3d. Then we use our image_geometry::PinholeCameraModel to project that point into image coordinates.

```

```

We assume here that we are listening on an undistorted image topic. The raw images from the camera driver are typically distorted, preventing a clean projective mapping from 3D world coordinates to 2D image coordinates; characterizing this distortion is an important part of camera calibration. image_geometry includes methods for undistorting images, but in ROS this is typically performed by an image_proc or stereo_image_proc node sitting between the camera driver and your vision processing node. See the image_pipeline for details.

注意, 使用的图像必须是经过校正的, 一般来讲raw_image是有畸变的, 在相机标定过程中, 很重要的任务是进行畸变校正, image_geometry有畸变校正的方法, 但是ros中一般是在image_proc中进行处理

```

# image_pipline

image-pipline这个包集用来处理驱动产生的生数据, 处理完后转化成视觉算法的输入, 这些数据包括: 校正过的图像, 视觉图像, 视觉点云, 软件组件包括

标定: 将图像中的数据和三维世界关联起来

单目相机图像处理: 去除图像畸变

立体视觉图像处理: 对于多个相机, 产生深度图和点云

深度图处理: 产生点云的( 这个在三维处理文章中说明)

可视化: image_view

## camera-calibration

使用opencv中的标定工具, [https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html](https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html)

具体描述信息 [http://wiki.ros.org/image_pipeline/CameraInfo](http://wiki.ros.org/image_pipeline/CameraInfo)

如何操作软件

[http://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration](http://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration)

## image-proc

图像校正, 一般来讲raw数据不是视觉算法想要的, 节点支持的功能有, 产生非校正和校正的图像

![](https://tcs.teambition.net/storage/312e809187e22b21d4fad944e94bf49ead7b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA1MjgwNSwiaWF0IjoxNjcxNDQ4MDA1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmU4MDkxODdlMjJiMjFkNGZhZDk0NGU5NGJmNDllYWQ3YiJ9.5bnJy1a0yZCEi3eU3lyXoUEuuf_FCiaScD9cuiahadY&download=image.png "")

可以使用nodelet, 可以在一个proc节点内部做很多处理, 支持的功能有

image_proc node simply loads one debayer nodelet and two rectify nodelets.

```

image_proc/debayer-去拜耳化

image_proc/rectify-校正

image_proc/crop_decimate- 感兴趣区域抽取

使用的参数包括: x, y, x偏移, y偏移,

image_proc/resize 重新定义尺寸

```

## image_publisher

手动发布一个图片, 也支持nodelets

## image_view

```

rosrun image_view image_view image:=<image topic> [image transport type]

rosrun image_view image_view image:=/camera/image

rosrun image_view image_view image:=/camera/image _image_transport:=theora

```

### 保存图片

从ros图片数据流中保存数据

```

rosrun image_view image_saver image:=[your topic]

rosrun image_view image_saver image:=image_raw _save_all_image:=false _filename_format:=foo.jpg __name:=image_saver

rosservice call /image_saver/save

```

### 保存为视频

保存为视频文件

# image_common

- [image_transport](https://wiki.ros.org/image_transport): Provides support for transporting images in low-bandwidth compressed formats. Should be used by anyone publishing or subscribing to images.

- [camera_calibration_parsers](https://wiki.ros.org/camera_calibration_parsers): Contains routines for reading and writing camera calibration parameters. Mostly used by camera drivers.

- [camera_info_manager](https://wiki.ros.org/camera_info_manager): C++ interface for saving, restoring and setting camera calibration information. Mostly used by camera drivers.

- [polled_camera](https://wiki.ros.org/polled_camera): Defines a ROS interface for requesting images from a polling camera driver.

## camera_calibration_parsers

将cameraInfo转成yaml标定文件

文件解析器

## camera_info_manager

管理标定信息

![](https://tcs.teambition.net/storage/312efc637fa04a871633769ba13c4108474f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA1MjgwNSwiaWF0IjoxNjcxNDQ4MDA1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmVmYzYzN2ZhMDRhODcxNjMzNzY5YmExM2M0MTA4NDc0ZiJ9.FJ2bdQADj6XSJ58CIb3m3v5WocLWuDapodp6Il7zjjY&download=image.png "")

## image_transport

必须使用image_transport来订阅和发布图像数据, 它提供低带宽下的图像数据透明传输

使用特殊传输策略, 图像压缩. 当使用这个包时, 开发者依然只看得到sensor_msgs/Image消息

根据需求使用不同的传输策略

```

image_transport ("raw") - The default transport, sending sensor_msgs/Image through ROS.

compressed_image_transport ("compressed") - JPEG or PNG image compression.

theora_image_transport ("theora") - Streaming video using the Theora codec.

```

#### 使用

就像使用ros publisher 和 subscriber一样

```

// Use the image_transport classes instead.

#include <ros/ros.h>

#include <image_transport/image_transport.h>

void imageCallback(const sensor_msgs::ImageConstPtr& msg)

{

  // ...

}

ros::NodeHandle nh;

image_transport::ImageTransport it(nh);

image_transport::Subscriber sub = it.subscribe("in_image_base_topic", 1, imageCallback);

image_transport::Publisher pub = it.advertise("out_image_base_topic", 1);

```

```

rosrun image_transport list_transports

```
