# SROS2使用

RCL暴露接口来使用DDS提供的信息安全特性

## 参考

<https://design.ros2.org/articles/ros2_dds_security.html>

[https://ubuntu.com/blog/ros-2-foxy-fitzroy-and-its-enhanced-security-monitoring](https://ubuntu.com/blog/ros-2-foxy-fitzroy-and-its-enhanced-security-monitoring)

<https://github.com/ros2/sros2/blob/master/SROS2_Linux.md>

<http://wiki.ros.org/ROS2/WorkingGroups/Security>

<https://design.ros2.org/articles/ros2_security_enclaves.html>

## 设计

默认情况下安全功能是不启动的，启动的这一系列特性成为SROS2

- 对于每一个参与者的安全文件

- 支持安全授权

- 支持安全措施的一键开启和关闭

### 安全文件

RCL要求安全文件存储在指定的目录，每一个安全示例包含一个enclaves

```bash
<root>
├── enclaves
│ └── front
│         └── camera
│                 ├── cert.pem
│                 ├── key.pem
│                 ├── ...
└── public
    ├── ...
```

- identity_ca.cert.pem: The x.509 certificate of the CA trusted by the Authentication plugin (the “Identity” CA).

- cert.pem: The x.509 certificate of this enclave instance (signed by the Identity CA).

- key.pem: The private key of this enclave instance.

- permissions_ca.cert.pem: The x.509 certificate of the CA trusted by the Access control plugin (the “Permissions” CA).

- governance.p7s: The XML document that specifies to the Access control plugin how the domain should be secured (signed by the Permissions CA).

- permissions.p7s: The XML document that specifies the permissions of this particular enclave instance to the Access control plugin (also signed by the Permissions CA).

```bash
export ROS_SECURITY_KEYSTORE="/home/bob/.ros/sros2_keystore"
ros2 run <package> <executable> --ros-args --enclave="/front/camera"
export ROS_SECURITY_KEYSTORE="/home/bob/.ros/sros2_keystore"
export ROS_SECURITY_ENCLAVE_OVERRIDE="/front/camera"
ros2 run <package> <executable>
export ROS_SECURITY_KEYSTORE="/home/bob/.ros/sros2_keystore"
export ROS_SECURITY_ENCLAVE_OVERRIDE="/front/camera"
ros2 run <package> <executable> --ros-args --enclave="/spam"
```

### 保守模式和激进模式

当启动一个参与者时，有两种安全模式

- 保守模式：尝试寻找安全文件，如果找不到，则不开启任何安全特性，启动参与者

- 激进模式：尝试寻找安全文件，如果找不到，不启动参与者

```bash
export ROS_SECURITY_STRATEGY=Enforce
```

### 全局开关

```bash
export ROS_SECURITY_STRATEGY=true
```

## SROS2 CLI

`https://github.com/ros2/sros2`

CLI提供以下功能：

- 创建认证和授权CA

- 创建文件夹放置安全文件

- 对一个给定的enclave，产生密钥对，使用CA进行签名

- 创建管理文件加密DDS通信数据

- 对给定的enclave，支持上层控制策略转化为对应的DDS安全策略

- 支持对于运行的ROS系统自动发现所需要的授权

### Secure Enclaves

安全飞地是ROS2安全的实现形式。

数据只有通过Secure Enclaves才能够解密和授权，而Secure Enclaves是高度封闭的。

就相当于手机和SIM卡之间的关系，互相独立，SIM卡完全封闭，外界访问很有限

### ROS概念到DDS概念的映射

Namespaces不支持，

之前DDS安全必须指明节点的全名Fully Qualified Name (FQN)

现在Participants对应ROS中的概念是Contexts，所以FQN就不能对应到Participants了

之前一个Context是一个进程，一个进程对应一个Participant，后来一个进程中可以有多个节点，新的模式下，多个node共享一个context, context对应paticipant

而根据DDS安全规范，控制策略的细粒度为Participant，所以一个participant对应一个enclave，所以每一个context，以及下面的所有节点使用一个单独的enclave。

**意味着，只要在一个进程中，使用composedNodes的格式做进程内通信，那么一个节点的安全策略与另一个节点的安全策略一致.**

根据上述映射，Keystore的组织结构就和ROS的不一样了。

```bash
$ tree keystore/
keystore
├── enclaves
│ └── ...
│ └── ...
├── private
│ ├── ca.csr.pem
│ └── ca.key.pem
└── public
    ├── ca.cert.pem
    ├── identity_ca.cert.pem -> ca.cert.pem
    └── permissions_ca.cert.pem -> ca.cert.pem
```

public目录，包含公共访问的认证文件和授权文件，所有程序都可访问

private目录，私有的文件

encvlaves目录，包含安全文件，跟节点目录关系不大了，

`https://docs.ros.org/en/foxy/Releases/Release-Crystal-Clemmys.html`

context的概念在crystal版本就引入了

【最终结构根据FOXY测试结果】

### 中间件注意事项

中间件与中间件间的加密通信不能互通

### 中间件有额外的配置
