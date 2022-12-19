# 概述

该REP介绍CameraInfo的相关信息，基本功能，对binning，ROI的支持，对畸变模型的支持。

subsampling (binning) -采样

# 动机

CameraInfo在ROS视觉项目中广泛使用，基于成熟的相机数学模型。

但是缺乏对binning，ROI的支持，畸变模型的支持。

## binning

Binning is the process of summing small neighborhoods of pixels on chip into larger "bins." For example, 2x2 binning reduces the resolution of the camera by half both horizontally and vertically. The chief advantage of binning is to increase the signal to noise ratio (SNR). This is especially useful in low-light environments. A second benefit is that decimating the resolution may allow for increased frame rate.

领域处理，下采样。两个作用，一个是提升信噪比，一个是提升帧率

Giving up resolution for increased SNR and/or frame rate is a useful trade-off in many vision applications. The utility of binning, combined with its wide availability in camera hardware, makes this a particularly valuable feature to support in CameraInfo.

The correct geometric interpretation of a binned image requires scaling the focal lengths and principal point calculated during calibration at full resolution.

在下采样后的图像，其几何模型发生改变，所以标定信息中要包含binning信息

## [Alternate Distortion Models](https://ros.org/reps/rep-0104.html#id30)

Currently CameraInfo assumes the "Plumb Bob" model of distortion [[2]](https://ros.org/reps/rep-0104.html#plumbbob), a 5-parameter polynomial approximation of radial and tangential distortion. Recently OpenCV has added support for an 8-parameter rational polynomial distortion model. This new model offers improved stability and accuracy for wide angle and fisheye lenses, which can severely distort the image. Unfortunately the size of the distortion parameter array D in CameraInfo is fixed at 5, so there is currently no way to support more complex models.

提供更多的畸变模型，更加灵活的畸变模型数据结构

This REP makes D variable-length and introduces a string distortion_model to distinguish which distortion model was used in the calibration. These changes also accommodate adding other distortion models in the future, should we find it necessary.

## [Region of Interest](https://ros.org/reps/rep-0104.html#id31)

Region of interest is another common camera feature, allowing the user to instruct the imager to only capture a desired sub-rectangle of its full resolution. This may be used to reduce bandwidth, particularly with high-definition cameras that could overwhelm a network connection at full resolution. Like binning, it may increase the frame rate for some cameras. Finally, it may improve the imagery of the object we are interested in by applying smart camera features such as auto-exposure only to the relevant image patch.

ROI感兴趣区域也是一个实用的功能，允许用户获取图像数据的一个子集，这样可以降低带宽使用，特别是高分辨率摄像头，和binning一样也可以增加帧率。

Applications of ROI to vision include performing high-speed tracking of an object, or acquiring a high-resolution close-up of an object with known location.

ROI的应用包括对象跟踪等

CameraInfo already supports ROI in that it contains fields for the sub-rectangle captured by the imager. This ROI is given in raw image coordinates, as supplied to the imager, as that is all the camera driver is expected to know and care about. **Given the ROI and the camera calibration, we (or rather image_geometry) can rectify the image patch**. But what is the ROI of the rectified patch in rectified coordinates; that is, given the full rectified image, what sub-rectangle inside of it corresponds to the rectified patch?

Currently image_geometry (by extension, [image_proc](http://www.ros.org/wiki/image_proc) [[15]](https://ros.org/reps/rep-0104.html#id23)) assumes that the ROI in rectified coordinates is the same as the ROI in raw (unrectified) coordinates. This behavior is broken. For small ROIs and/or large amounts of distortion, the raw patch (once rectified) may not even coincide with the same ROI in rectified coordinates. In that case the rectified image patch is set to all black, as the data required to fill it was not even captured [[3]](https://ros.org/reps/rep-0104.html#roiticket). Therefore we must introduce a mapping between the ROI in raw coordinates and the corresponding ROI in rectified coordinates.

# 新消息定义

The proposed CameraInfo message is listed (stripped of comments) below. Additions and changes are noted on the right.

```text
Header header
  uint32 seq
  time stamp
  string frame_id

uint32 height
uint32 width

# roi used to be located here.
# I've moved it to after the calibration parameters.

string distortion_model            # New field

float64[] D                        # Made variable-length
float64[9] K
float64[9] R
float64[12] P

uint32 binning_x                   # New field
uint32 binning_y                   # New field

sensor_msgs/RegionOfInterest roi   # Moved field
  uint32 x_offset
  uint32 y_offset
  uint32 height
  uint32 width
  bool do_rectify                  # New field
```

# 解析

There are three parts to CameraInfo.

- ROS header. The time stamp is the time at which the image was captured. frame_id is the name of the **optical coordinate frame** with origin at the optical center of the camera.

  - 这个坐标系一定是数据（感光单元）的坐标系，而不是安装孔位的坐标系

- Calibration parameters. These are fixed during camera calibration. Their values will be the same in all messages until the camera is recalibrated. Note that self-calibrating systems may "recalibrate" frequently.

  - 标定数据，在传感器标定后就固定下来，也有自标定系统进行在线重标定

- Operational parameters. These define the image region actually captured by the camera driver. Although they affect the geometry of the output image, they may be changed freely without recalibrating the camera.

  - 操作类的参数，定义了相机驱动输出数据的实际区域信息，这些数据与标定数据无关

| Calibration Parameters | Operational Parameters |
| ---------------------- | ---------------------- |
| height                 | binning_x              |
| width                  | binning_y              |
| D                      | roi                    |
| K                      |                        |
| R                      |                        |
| P                      |                        |

## [Calibration Parameters](https://ros.org/reps/rep-0104.html#id35)

The height and width fields always contain the image dimensions with which the camera was calibrated; normally this will be the full resolution of the camera.

The arrays of calibration parameters D, K, R and P are interpreted as described in [[1]](https://ros.org/reps/rep-0104.html#camerainfo) [[4]](https://ros.org/reps/rep-0104.html#opencv). D contains the parameters of the model named by the distortion_model string.

Recognized distortion_model names are given in the new header sensor_msgs/distortion_models.h. For Diamondback these will be "plumb_bob" and "rational_polynomial", as described in [Alternate Distortion Models](https://ros.org/reps/rep-0104.html#alternate-distortion-models). Empty D and distortion_model indicate that the CameraInfo cannot be used to rectify points or images, either because the camera is not calibrated or because the rectified image was produced using an unsupported distortion model, e.g. the proprietary one used by Bumblebee cameras [[5]](https://ros.org/reps/rep-0104.html#bumblebee).

## [Operational Parameters](https://ros.org/reps/rep-0104.html#id36)

**Binning reduces the resolution of the output image to (width / binning_x) x (height / binning_y).**Consumers of CameraInfo (such as image_geometry) must scale the focal lengths and principal point of the camera model. Both supported distortion models operate on normalized image coordinates (independent of focal length and principal point), and the rotation matrix R on 3D world coordinates, so binning does not affect these parameters.

**For the sake of backwards compatibility, binning_x = binning_y = 0 (the default values) is considered the same as binning_x = binning_y = 1, or no binning.**

**The ROI is specified in the full (unbinned) image coordinates.** For example, the 100x150 sub-rectangle at offset (25, 35) in a 2x2 binned image is represented as a ROI with dimensions 200x300 and offset (50, 70). The ROI specifies a sub-rectangle of pixels on the imager, independent of binning. x_offset and y_offset are the offset from the top-left corner of the **full image** to the top-left corner of the region of interest.

ROI是在全分辨率图像坐标系上标识的，ROI参数是独立于binning参数的

As a convenience, setting roi.x_offset, roi.y_offset, roi.width and roi.height **all to 0 has a special meaning; it is the same as the full resolution**. This is especially useful to users of the polled camera interface [[6]](https://ros.org/reps/rep-0104.html#polledcamera), who can request a full resolution image despite not knowing ahead of time what that resolution is. It also means that authors of camera drivers that do not support ROI can safely ignore the CameraInfo/roi fields, which default to 0.

## [Raw and Rectified ROI](https://ros.org/reps/rep-0104.html#id37)

When working with distorted images, a desired raw ROI can be given directly to the camera driver. More commonly, **however, the consumer wants the camera to provide an ROI in the rectified image,** and does not particularly care how the rectified image patch is acquired. To bridge the gap between user (who works in rectified coordinates) and camera driver (which understands only raw coordinates), we define a mapping between **raw and rectified ROI**.

Given a rectified ROI, the corresponding raw ROI is the smallest sub-rectangle such that every pixel in the rectified ROI maps to a point inside the raw ROI. In other words, the raw ROI must contain all the information needed to fill the rectified ROI. Geometrically, if we distort the outline of the rectified ROI into raw coordinates, the raw ROI circumscribes the resulting curve.

Likewise, given a raw ROI, the corresponding rectified ROI is the largest sub-rectangle such that every pixel in the rectified ROI maps to a point inside the raw ROI. If we rectify the outline of the raw ROI, the rectified ROI inscribes the resulting curve.

When a full resolution image is captured, the behavior is different. During the camera calibration process, the user chooses a scaling which trades off between having all valid pixels in the rectified image (but discarding some pixels from the raw image) versus discarding no raw image pixels (but permitting invalid pixels in the rectified image). The assumption that all rectified pixels should be valid does not necessarily hold; that is up to the user to decide during calibration. The raw and rectified images have the same resolution, and hence the same "ROI" (the full image). roi.do_rectify is set to False to indicate that no ROI mapping should be done.

When the raw ROI is only part of the full resolution, roi.do_rectify is set to True. The raw ROI is mapped to the rectified ROI as described above.

It is permitted to set roi.do_rectify = False when the ROI is not actually the same as the full image resolution to suppress the ROI mapping. This feature can be useful in special situations, for example if the camera supports particular video modes with the resolution cropped to a smaller field of view. See use case [#3 Cropped Video Mode](https://ros.org/reps/rep-0104.html#cropped-video-mode) below.

To complete the set of possibilities, the user could request an ROI of the full image resolution but with roi.do_rectify = True. This ensures that the rectified image contains no invalid pixels, but it will also discard pixels from the raw image.

# 驱动开发指南

The main purpose of a camera driver is to expose useful functions of the camera hardware. If a camera does not support binning or ROI in hardware, the driver has no obligation to implement these features in software; indeed, that might mislead users as to the camera's actual capabilities. Such post-processing can always be performed by a separate node.

[Raw and Rectified ROI](https://ros.org/reps/rep-0104.html#raw-and-rectified-roi) described a range of ways in which the ROI fields may be set for different purposes. In practice most camera drivers only need one or two behaviors. When capturing at full resolution, CameraInfo/roi can be left at the default of all zeros to signify full resolution. Drivers that do not support ROI need not touch CameraInfo/roi at all. When the user has requested some ROI (perhaps through dynamic_reconfigure), the driver should set CameraInfo/roi with the ROI offsets and size used and do_rectify = True.

# 例子

Here we examine how a camera driver might fill the CameraInfo message in different modes of operation, roughly increasing in complexity. We use the WGE100 camera [[8]](https://ros.org/reps/rep-0104.html#wge100) as an example. This camera has 752x480 resolution, but is commonly used in 640x480 mode, which crops the left-most and right-most 56 columns of the imager. It supports both binning and ROI.

The camera is calibrated only once, in full 752x480 resolution. The same camera parameters (height, width, D, K, R, P) are reused in all cases. They are combined with the driver's current operational parameters (binning_x, binning_y, roi) to derive the geometry of the output images.

[#1 Full Resolution](https://ros.org/reps/rep-0104.html#id42)

The most basic case - we capture full 752x480 images. The relevant CameraInfo settings are:

```text
height: 480
width:  752
binning_x: 0
binning_y: 0
roi
    offset_x:   0
    offset_y:   0
    height:     0
    width:      0
    do_rectify: False

```

For this example we have left the binning and ROI fields at their defaults of 0. Drivers that do not support binning and/or ROI do not need to touch these fields at all. The above message has the same meaning as:

```text
height: 480
width:  752
binning_x: 1
binning_y: 1
roi
    offset_x:   0
    offset_y:   0
    height:     480
    width:      752
    do_rectify: False

```

## [#2 Region of Interest](https://ros.org/reps/rep-0104.html#id43)

Let's capture a 200x300 ROI with top left corner (50, 70):

```text
height: 480
width:  752
binning_x: 1
binning_y: 1
roi
    offset_x:   50
    offset_y:   70
    height:     300
    width:      200
    do_rectify: True

```

The driver simply needs to fill in the roi field. roi.**do_rectify is now set to True**, as the best-fitting rectified ROI may overlap poorly with the original raw ROI.

## [#3 Cropped Video Mode](https://ros.org/reps/rep-0104.html#id44)

Now we change the camera to 640x480 mode, cropping 56 columns on each side:

```text
height: 480
width:  752
binning_x: 1
binning_y: 1
roi
    offset_x:   56
    offset_y:   0
    height:     480
    width:      640
    do_rectify: False

```

The cropping effect of the lower-resolution mode is encoded in CameraInfo as a ROI. roi.**do_rectify = False because we wish to pretend that we are in fact running a 640x480 camera at full resolution. The rectified image will also be 640x480.**

## [#4 Cropped Video Mode with ROI](https://ros.org/reps/rep-0104.html#id45)

Again capturing a 200x300 ROI with top left corner (50, 70), this time with respect to the 640x480 image:

```text
height: 480
width:  752
binning_x: 1
binning_y: 1
roi
    offset_x:   106
    offset_y:   70
    height:     300
    width:      200
    do_rectify: True

```

Notice that the ROI in CameraInfo is actually specified in the full 752x480 resolution we calibrated with. roi.offset_x = 106 = 56 + 50.

## [#5 Binned, Cropped Video Mode](https://ros.org/reps/rep-0104.html#id46)

Staying in 640x480 cropped video mode, we enable 2x2 binning:

```text
height: 480
width:  752
binning_x: 2
binning_y: 2
roi
    offset_x:   56
    offset_y:   0
    height:     480
    width:      640
    do_rectify: False

```

The output resolution is reduced to 320x240. Since ROI is specified in unbinned coordinates, the roi fields are unchanged from #3.

## [#6 Binned, Cropped Video Mode with ROI](https://ros.org/reps/rep-0104.html#id47)

Finally, we capture the same ROI as in #4:

```text
height: 480
width:  752
binning_x: 2
binning_y: 2
roi
    offset_x:   106
    offset_y:   70
    height:     300
    width:      200
    do_rectify: True

```

Again, the roi fields are unchanged from #4.

# 参考

[REP 104 -- CameraInfo updates for Diamondback (ROS.org)](https://ros.org/reps/rep-0104.html)

[sensor_msgs/CameraInfo Message理解_wangmj_hdu的博客-CSDN博客](https://blog.csdn.net/wangmj_hdu/article/details/115002514)
