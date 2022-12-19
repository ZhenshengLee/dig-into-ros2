# 概念

相机内参数是与相机自身特性相关的参数，比如相机的焦距、像素大小等；

相机的内参数是六个分别为：1/dx、1/dy、r、u0、v0、f。

opencv1里的说内参数是4个其为fx、fy、u0、v0。实际其fx=F*Sx，其中的F就是焦距上面的f,Sx是像素/没毫米即上面的dx，其是最后面图里的后两个矩阵进行先相乘，得出的，则把它看成整体，就相当于4个内参。其是把r等于零，实际上也是六个。

dx和dy表示：x方向和y方向的一个像素分别占多少长度单位，即一个像素代表的实际物理值的大小，其是实现图像物理坐标系与像素坐标系转换的关键。u0，v0表示图像的中心像素坐标和图像原点像素坐标之间相差的横向和纵向像素数。

相机外参数是在世界坐标系中的参数，比如相机的位置、旋转方向等。

# CameraInfo格式

![](https://tcs.teambition.net/storage/312e3a3eb3be64752748fa3b3cff970e2ca9?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA1Mjc5OCwiaWF0IjoxNjcxNDQ3OTk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmUzYTNlYjNiZTY0NzUyNzQ4ZmEzYjNjZmY5NzBlMmNhOSJ9.UCEJharHZBF3p_de3oxWKnhs2anVBzuib87I1OTzrR4&download=image.png "")

```bash
# This message defines meta information for a camera. It should be in a
# camera namespace on topic "camera_info" and accompanied by up to five
# image topics named:
#
#   image_raw - raw data from the camera driver, possibly Bayer encoded
#   image            - monochrome, distorted
#   image_color      - color, distorted
#   image_rect       - monochrome, rectified
#   image_rect_color - color, rectified
#
# The image_pipeline contains packages (image_proc, stereo_image_proc)
# for producing the four processed image topics from image_raw and
# camera_info. The meaning of the camera parameters are described in
# detail at http://www.ros.org/wiki/image_pipeline/CameraInfo.
#
# The image_geometry package provides a user-friendly interface to
# common operations using this meta information. If you want to, e.g.,
# project a 3d point into image coordinates, we strongly recommend
# using image_geometry.
#
# If the camera is uncalibrated, the matrices D, K, R, P should be left
# zeroed out. In particular, clients may assume that K[0] == 0.0
# indicates an uncalibrated camera.

#######################################################################
#                     Image acquisition info                          #
#######################################################################

# Time of image acquisition, camera coordinate frame ID
Header header    # Header timestamp should be acquisition time of image
                 # Header frame_id should be optical frame of camera
                 # origin of frame should be optical center of camera
                 # +x should point to the right in the image
                 # +y should point down in the image
                 # +z should point into the plane of the image


#######################################################################
#                      Calibration Parameters                         #
#######################################################################
# These are fixed during camera calibration. Their values will be the #
# same in all messages until the camera is recalibrated. Note that    #
# self-calibrating systems may "recalibrate" frequently.              #
#                                                                     #
# The internal parameters can be used to warp a raw (distorted) image #
# to:                                                                 #
#   1. An undistorted image (requires D and K)                        #
#   2. A rectified image (requires D, K, R)                           #
# The projection matrix P projects 3D points into the rectified image.#
#######################################################################

# The image dimensions with which the camera was calibrated. Normally
# this will be the full camera resolution in pixels.
uint32 height
uint32 width

# The distortion model used. Supported models are listed in
# sensor_msgs/distortion_models.h. For most cameras, "plumb_bob" - a
# simple model of radial and tangential distortion - is sufficient.
string distortion_model

# The distortion parameters, size depending on the distortion model.
# For "plumb_bob", the 5 parameters are: (k1, k2, t1, t2, k3).
float64[] D

# Intrinsic camera matrix for the raw (distorted) images.
#     [fx  0 cx]
# K = [ 0 fy cy]
#     [ 0  0  1]
# Projects 3D points in the camera coordinate frame to 2D pixel
# coordinates using the focal lengths (fx, fy) and principal point
# (cx, cy).
float64[9]  K # 3x3 row-major matrix

# Rectification matrix (stereo cameras only)
# A rotation matrix aligning the camera coordinate system to the ideal
# stereo image plane so that epipolar lines in both stereo images are
# parallel.
float64[9]  R # 3x3 row-major matrix

# Projection/camera matrix
#     [fx'  0  cx' Tx]
# P = [ 0  fy' cy' Ty]
#     [ 0   0   1   0]
# By convention, this matrix specifies the intrinsic (camera) matrix
#  of the processed (rectified) image. That is, the left 3x3 portion
#  is the normal camera intrinsic matrix for the rectified image.
# It projects 3D points in the camera coordinate frame to 2D pixel
#  coordinates using the focal lengths (fx', fy') and principal point
#  (cx', cy') - these may differ from the values in K.
# For monocular cameras, Tx = Ty = 0. Normally, monocular cameras will
#  also have R = the identity and P[1:3,1:3] = K.
# For a stereo pair, the fourth column [Tx Ty 0]' is related to the
#  position of the optical center of the second camera in the first
#  camera's frame. We assume Tz = 0 so both cameras are in the same
#  stereo image plane. The first camera always has Tx = Ty = 0. For
#  the right (second) camera of a horizontal stereo pair, Ty = 0 and
#  Tx = -fx' * B, where B is the baseline between the cameras.
# Given a 3D point [X Y Z]', the projection (x, y) of the point onto
#  the rectified image is given by:
#  [u v w]' = P * [X Y Z 1]'
#         x = u / w
#         y = v / w
#  This holds for both images of a stereo pair.
float64[12] P # 3x4 row-major matrix


#######################################################################
#                      Operational Parameters                         #
#######################################################################
# These define the image region actually captured by the camera       #
# driver. Although they affect the geometry of the output image, they #
# may be changed freely without recalibrating the camera.             #
#######################################################################

# Binning refers here to any camera setting which combines rectangular
#  neighborhoods of pixels into larger "super-pixels." It reduces the
#  resolution of the output image to
#  (width / binning_x) x (height / binning_y).
# The default values binning_x = binning_y = 0 is considered the same
#  as binning_x = binning_y = 1 (no subsampling).
uint32 binning_x
uint32 binning_y

# Region of interest (subwindow of full camera resolution), given in
#  full resolution (unbinned) image coordinates. A particular ROI
#  always denotes the same window of pixels on the camera sensor,
#  regardless of binning settings.
# The default setting of roi (all values 0) is considered the same as
#  full resolution (roi.width = width, roi.height = height).
RegionOfInterest roi

```

![](https://tcs.teambition.net/storage/312e9e38ad9c9a0a10492a2acec8feaf7df7?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA1Mjc5OCwiaWF0IjoxNjcxNDQ3OTk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmU5ZTM4YWQ5YzlhMGExMDQ5MmEyYWNlYzhmZWFmN2RmNyJ9.oHPT-015gJPK4yucevOfxc52OOt1aQNCOkXj6fC5sYQ&download=image.png "")

里面的每一个参数代表着什么意思呢，接下来我将对这些参数进行解释。

header：标准消息头

seq：序列ID，连续递增的ID号

stamp：两个时间戳

frame_id：与此数据相关联的帧ID

height：图像尺寸，height代表高度，（height*width）相机的分辨率，以像素为单位

width：图像尺寸，width代表宽度，（height*width）相机分辨率，以像素为单位

distortion_model：指定了相机畸变模型，对于大多数相机，"plumb_bob"简单的径向和切向畸变模型就足够了

D：畸变参数，取决于畸变模型，(k1, k2, t1, t2, k3)，（我的这个usb相机号称是无畸变相机，但通过标定结果可以看出来还是存在畸变的，是不是被商家坑了，哈哈哈）

K：相机内参矩阵，使用焦距(fx, fy)和主点坐标(cx, cy)，单位为像素，内参矩阵可以将相机坐标中的3D点投影到2D像素坐标

R：旋转矩阵，将相机坐标系统对准理想的立体图像平面，使两张立体图像中的极线平行

P：投影矩阵

左边3*3矩阵是相机的内参矩阵，将相机坐标中的3D点投影到2D像素坐标，可能与相机内参K不同。对于单目相机Tx = Ty = 0。对于双目相机，Tx和Ty有所不同。

binning_x：图像下采样参数，水平方向

binning_y：图像下采样参数，竖直方向

(width / binning_x) x (height / binning_y)

下采样：binning_x = binning_y ＞ 1。缩小图像，生成对应图像的缩略图，使得图像符合显示区域的大小。

roi：感兴趣区域定义，即完整图像上的一个矩形子窗口

# 应用

利用Image_proc

![](https://tcs.teambition.net/storage/312e58ad69c1e67e6714296d2931cee9c6e1?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjA1Mjc5OCwiaWF0IjoxNjcxNDQ3OTk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmU1OGFkNjljMWU2N2U2NzE0Mjk2ZDI5MzFjZWU5YzZlMSJ9.-Oi0u7symtXcErb92NOtVgk4oon9NRjR79SNEfeWmVo&download=image.png "")

# 参考

[image_pipeline/CameraInfo - ROS Wiki](https://wiki.ros.org/image_pipeline/CameraInfo)

[Python CameraInfo.R示例，sensor_msgsmsg.CameraInfo.R Python示例 - HotExamples](https://python.hotexamples.com/zh/examples/sensor_msgs.msg/CameraInfo/R/python-camerainfo-r-method-examples.html)
