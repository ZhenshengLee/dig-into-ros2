# OSRF描述

![](https://tcs.teambition.net/storage/312j4ac5942182a3e4fe86a575b9f2371f58?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjMwMCwiaWF0IjoxNjcxMDE3NTAwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmo0YWM1OTQyMTgyYTNlNGZlODZhNTc1YjlmMjM3MWY1OCJ9.yL6X6hujQMFNs0QMv93p-uXH3vHcj_ZD23W6I3sD5GE&download=blob.gif "")

As we discussed in prior blog posts, Open Robotics has been working with NVIDIA is to improve ROS 2 performance on NVIDIA hardware. Much of this work has focused on**providing hooks into ROS 2 to minimize memory consumption and better pipeline serial processing operations.**We are happy to report that this work is now available in ROS 2 Humble and should improve ROS 2 performance for perception and planning related tasks on NVIDIA hardware.

**TYPE ADAPTATION**

ROS topics can be adapted to a format better suited for acceleration in hardware using type adaptation ([REP-2007 2](https://ros.org/reps/rep-2007.html)). A node using an adapted type, can publish, and/or receive the adapted type. **Nodes using an adapted type, need to provide functions to convert from the standard type, to the adapted type, and visa-versa. This enables a graph of nodes to use an adapted type which can improve CPU and hardware acceleration concurrency, offload the CPU from compute tasks, and eliminate memory copies between the CPU and hardware accelerator.**

这个对于非Nvidia设备也是一样的效果

**TYPE NEGOTIATION**

![](https://tcs.teambition.net/storage/312jaa25e647e438276b3a1cf9e1c51dacbf?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjMwMCwiaWF0IjoxNjcxMDE3NTAwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmphYTI1ZTY0N2U0MzgyNzZiM2ExY2Y5ZTFjNTFkYWNiZiJ9.vIPPSQd1sybzaO9N_zKdSIawAhXz9BmxBAqWkI1rJVE&download=blob.png "")

With a graph of ROS nodes using an adapted type, we can further benefit from optimizing the type used between nodes in the graph. Nodes supporting type negotiation ([REP-2009 2](https://ros.org/reps/rep-2009.html)) can **share a list of types they support as a publisher, and as a subscriber with a weight indicating their preferences.** ROS will review the publishers and subscribers participating in type negotiation, and optimize for preferences while maintaining compatibility with nodes that do not support type negotiation. **Preferences are a way to reflect performance, or cost of the type, and should be tuned by the developer of the node, but can be overridden by the application developer.**

![](https://tcs.teambition.net/storage/312j7265d831d4a8c5058f90e0aeb509fc86?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTYyMjMwMCwiaWF0IjoxNjcxMDE3NTAwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmo3MjY1ZDgzMWQ0YThjNTA1OGY5MGUwYWViNTA5ZmM4NiJ9.LPeV9O9TIP7XjrdC1x4fCXJOs96e5hiElQuONLovPXA&download=blob.png "")

The end result of adding type negotiation and type adaptation to ROS 2 is that end users will see dramatic performance improvements on supported hardware, close to 10X! For example, when performing pixel processing operations on a Jetson AGX Orin the total throughput went from **0.55 gigapixels/sec** in Foxy to **4 gigapixels/sec** in Humble for our benchmark tests! This is a huge win for ROS 2 users who work on perception heavy applications!

# Nvidia描述

Working in collaboration since October 2021, NVIDIA and Open Robotics are introducing two important changes, now available in the Humble ROS 2 release for improved performance on compute platforms that offer hardware accelerators.

The new ROS 2 Humble hardware-acceleration features are called type adaptation and type negotiation. **NVIDIA will release a software package-implementing type adaptation and type negotiation in the next NVIDIA Isaac ROS release** (late June 2022).

These simple but powerful additions to the framework will significantly increase performance for developers seeking to incorporate AI/machine learning and computer vision functionality into their ROS-based applications.

“As ROS developers add more autonomy to their robot applications, the on-robot computers are becoming much more powerful. We have been working to evolve the ROS framework to make sure that it can take advantage of high-performance hardware resources in these edge computers,” said Brian Gerkey, CEO of Open Robotics.

“Working closely with the NVIDIA robotics team, we are excited to share new features (type adaptation and negotiation) in the Humble release that will benefit the entire ROS community’s efforts to embrace hardware acceleration.”

## Eliminating overhead of hardware acceleration

### Type adaptation

It is common for hardware accelerators to require a different data format to deliver optimal performance. Type adaptation ([REP-2007](https://ros.org/reps/rep-2007.html)) can now be used for ROS nodes to work in the format better suited for the hardware. Processing pipelines can eliminate memory copies between the CPU and the memory accelerator using the adapted type. **Unnecessary memory copies consume CPU compute, waste power, and slow down performance, especially as the size of the images increases.**

### Type negotiation

Another new innovation is type negotiation ([REP-2009](https://ros.org/reps/rep-2009.html)). Different ROS nodes in a processing pipeline can advertise their supported types, so that formats yielding ideal performance are chosen. The ROS framework performs this negotiation process and maintains compatibility with legacy nodes that don’t support negotiation.

Accelerating processing pipelines using type adaptation and negotiation **makes hardware accelerator zero-copy possible**. This reduces software/CPU overhead and unlocks the potential of the underlying hardware. As roboticists migrate to more powerful compute platforms like [NVIDIA Jetson Orin](https://developer.nvidia.com/embedded/jetson-orin), they can expect to realize more of the performance gains enabled by the hardware.

These changes are done completely inside of ROS 2, which ensures compatibility with existing tools, workflows, and codebases.

![Two examples of hardware accelerated compute graphs.](https://developer-blogs.nvidia.com/wp-content/uploads/2022/05/image5-1.gif "")

*Figure 1. Comparing hardware accelerated pipelines with and without type adaptation and negotiation*

Type adaptation and negotiation have shown promising results. A benchmark consisting of a graph of ROS nodes, with minimal compute in each node, was run on ROS 2 Foxy and ROS 2 Humble so that we could observe the underlying framework performance. We ran this benchmark on Jetson AGX Xavier and the new [Jetson AGX Orin](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/). We observed a 3x improvement on Xavier and an impressive 7x improvement on Orin.

![Bar Chart of Framework Performance](https://developer-blogs.nvidia.com/wp-content/uploads/2022/05/image1-7.png "")

*Figure 2. Type adaptation framework benchmark performance comparing ROS 2 Foxy and ROS 2 Humble on Jetson AGX Xavier and Jetson AGX Orin*

# 参考

[https://developer.nvidia.com/blog/improve-perception-performance-for-ros-2-applications-with-nvidia-isaac-transport-for-ros/](https://developer.nvidia.com/blog/improve-perception-performance-for-ros-2-applications-with-nvidia-isaac-transport-for-ros/)

[https://www.openrobotics.org/blog/2022/5/24/ros-2-humble-hawksbill-release](https://www.openrobotics.org/blog/2022/5/24/ros-2-humble-hawksbill-release)
