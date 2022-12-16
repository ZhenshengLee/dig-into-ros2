# 概述

dds通过domain_id来切分可通信的子网，在一个域内可以自由发现和发送消息<br />ROS2默认使用domain_id=0

# ID划分规则

dds使用domain_id来定义服务发现通信的port，<br />the UDP port is an unsigned 16-bit integer. Thus, the highest port number that can be allocated is 65535. Doing some math with the formula in the article above, this means that the highest domain ID that can possibly be assigned is 232, while the lowest that can be assigned is 0.<br />需要避免使用操作系统外设通信接口，<br />By default, the Linux kernel uses ports 32768-60999 for ephemeral ports. This means that domain IDs 0-101 and 215-232 can be safely used without colliding with ephemeral ports. The ephemeral port range is configurable in Linux by setting custom values in /proc/sys/net/ipv4/ip_local_port_range. If a custom ephemeral port range is used, the above numbers may have to be adjusted accordingly.

# ID使用规范

每个参与者使用两个port用于服务发现通信<br />For each ROS 2 process running on a computer, one DDS “participant” is created. **Since each DDS participant takes up two ports on the computer,** running more than 120 ROS 2 processes on one computer may spill over into other domain IDs or the ephemeral ports.<br />To see why, consider the domain IDs 1 and 2.

- **Domain ID 1 uses port 7650 and 7651 for multicast.**
- **Domain ID 2 uses port 7900 and 7901 for multicast.**
- When creating the 1st process (zeroth participant) in domain ID 1, the ports 7660 and 7661 are used for unicast.
- When creating the 120th process (119th participant) in domain ID 1, the ports 7898 and 7899 are used for unicast.
- When creating the 121st process (120th participant) in domain ID 1, the ports 7900 and 7901 are used for unicast and overlap with domain ID 2.

If it is known that the computer will only ever be on a single domain ID at a time, and the domain ID is low enough, it is safe to create more ROS 2 processes than this.<br />When choosing a domain ID that is near the top of the range of platform-specific domain IDs, one additional constraint should be considered.<br />For instance, assume a Linux computer with a domain ID of 101:

- The zero’th ROS 2 process on the computer will connect to ports 32650, 32651, 32660, and 32661.
- The first ROS 2 process on the computer will connect to ports 32650, 32651, 32662, and 32663.
- The 53rd ROS 2 process on the computer will connect to ports 32650, 32651, 32766, and 32767.
- The 54th ROS 2 process on the computer will connect to ports 32650, 32651, 32768, and 32769, running into the ephemeral port range.

Thus the maximum number of processes that should be created when using domain ID 101 on Linux is 54. Similarly, the maximum number of processes that should be created when using domain ID 232 on Linux is 63, as the maximum port number is 65535.<br />The situation is similar on macOS and Windows, though the numbers are different. On macOS and Windows, when choosing a domain ID of 166 (the top of the range), **the maximum number of ROS 2 processes that can be created on a computer before running into the ephemeral port range is 120.**

# 结论

最好限制context数量少于120，即participant的数量

# 参考

[https://docs.ros.org/en/foxy/Concepts/About-Domain-ID.html](https://docs.ros.org/en/foxy/Concepts/About-Domain-ID.html)
