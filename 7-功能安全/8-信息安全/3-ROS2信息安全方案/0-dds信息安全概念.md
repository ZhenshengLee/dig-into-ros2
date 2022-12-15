# dds网络安全

## 参考

<https://www.omg.org/spec/DDS-SECURITY/1.1/PDF>

<https://design.ros2.org/articles/ros2_dds_security.html>

[https://design.ros2.org/articles/ros2_access_control_policies.html](https://design.ros2.org/articles/ros2_access_control_policies.html)

## 概述

DDS定义了网络安全规范，，通过定义一个SPI Service Plugin Interface，SPI的一系列实现来完成安全措施。包括：

- 认证Authentication：验证DDS参与者的身份

- Access control访问控制：对验证过的参与者，基于规则进行操作限制

- 加密Cryptographic：加密，签名，Hashing操作

- 日志Logging：对安全相关事件进行审查

- 数据标签DataTagging：数据标记确保完整性

目前ROS2的安全特性只涵盖**前三种**。因为后两种都需要遵循DDS安全规范，不过不是所有的DDS实现都完整支持这两个功能。

## 认证

认证是SPI设施的中心

在ROS2中，认证身份之后可以做任何事

ROS2使用内置的认证机制DDS:Auth:PKI-DH，使用经过验证的PKI

这种机制下，对每个参与者都有一个公钥和私钥，

Each x.509 certificate must be signed by (or have a signature chain to) a specific Certificate Authority (CA) that the plugin is configured to trust.

之所以用这种机制，有两个原因：

- 这是DDS规范中唯一细节描述的规范

- 对所有DDS实现是强制性的要求

## 访问控制

对于一个DDS参与者进行行为限制。

例如，允许某个参与者访问给定的主题。

使用了内置的访问控制机制，DDS:Access:Permission，which again uses PKI

要求参与者使用两个文件：

- Governance file：规定每个域的保护方式

- Permissions file：规定每个域的授权文件

这些文件都需要经过CA签名。

## 加密

encryption, decryption, signing, hashing,

认证和访问控制也使用到了加密基础设施。

加密部分将数据传输进行加密。

默认使用了内置的DDS:Crypto:AES-GCM-GMAC

使用了Advanced Encryption Standard (AES) in Galois Counter Mode (AES-GCM).加密方法

## 访问控制细节

`https://blog.csdn.net/LngZd/article/details/100781310`

`https://blog.csdn.net/ajian005/article/details/8490082`

### 一些概念

- Mandatory Access Control

强制访问控制，意味着只有在完全匹配规则的情况下才允许访问资源

MAC非常适合机密机构或者其他等级观念强烈的行业，过重强调保密性，管理不够灵活。在实现上，MAC和DAC通常为每个用户赋予对客体的访问权限规则集，考虑到管理的方便，在这一过程中还经常将具有相同职能的用户聚为组，然后再为每个组分配许可权。

deny by default

- discretionary access control (DAC)

自主访问控制模型（DAC，Discretionary Access Control）是根据自主访问控制策略建立的一种模型，允许合法用户以用户或用户组的身份访问策略规定的客体，同时阻止非授权用户访问客体。拥有客体权限的用户，可以将该客体的权限分配给其他用户。例如没有文件File1访问权限的用户可以从有访问权限的B用户那里得到访问权限。

- Principle of Least Privilege

最小特权原则：给主体分配权限时要遵循权限最小化原则，最小特权原则的优点是最大限度地限制了主体实施授权行为，可以避免来自突发事件、错误和未授权用主体的危险。

最小泄漏原则：它是指主体执行任务时,按照主体所需要知道的信息最小化的原则分配给主体权利。也就是要保护敏感信息不要被无关人员知道，别人知道得越少越好。

多级安全策略：多级安全策略是指主体和客体间的数据流向和权限控制按照安全级别的绝密（TS）、秘密（S）、机密（C）、限制（RS）和无级别（U）五级来划分。多级安全策略的优点是避免敏感信息的扩散，只有安全级别比他高的主体才能够访问。

访问控制XML设计

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema
    xmlns:xs="http://www.w3.org/2001/XMLSchema"
    xmlns:xml="http://www.w3.org/XML/1998/namespace"
    elementFormDefault="qualified" attributeFormDefault="unqualified">
    <xs:import namespace="http://www.w3.org/XML/1998/namespace"
               schemaLocation="http://www.w3.org/2001/03/xml.xsd" />
    <xs:element name="policy" type="Policy" />
    <xs:complexType name="Policy">
        <xs:sequence minOccurs="1" maxOccurs="1">
            <xs:element name="enclaves" type="Enclaves" />
        </xs:sequence>
        <xs:attribute name="version" type="xs:string" use="required" fixed="0.2.0"/>
    </xs:complexType>
    <xs:complexType name="Enclaves">
        <xs:sequence minOccurs="1" maxOccurs="unbounded">
            <xs:element name="enclave" type="Enclave" />
        </xs:sequence>
    </xs:complexType>
    <xs:complexType name="Enclave">
        <xs:sequence minOccurs="1" maxOccurs="unbounded">
            <xs:element name="profiles" type="Profiles" />
        </xs:sequence>
        <xs:attribute name="path" type="xs:string" use="required" />
    </xs:complexType>
    <xs:complexType name="Profiles">
        <xs:sequence minOccurs="1" maxOccurs="1">
            <xs:sequence minOccurs="1" maxOccurs="unbounded">
                <xs:element name="profile" type="Profile" />
            </xs:sequence>
            <xs:sequence minOccurs="0" maxOccurs="1">
                <xs:element name="metadata" type="xs:anyType" />
            </xs:sequence>
        </xs:sequence>
        <xs:attribute name="type" type="xs:string" use="optional" />
    </xs:complexType>
    <xs:complexType name="Profile">
        <xs:sequence minOccurs="0" maxOccurs="unbounded">
            <xs:choice minOccurs="1" maxOccurs="1">
                <xs:element name="topics" minOccurs="1" type="TopicExpressionList" />
                <xs:element name="services" minOccurs="1" type="ServicesExpressionList" />
                <xs:element name="actions" minOccurs="1" type="ActionsExpressionList" />
            </xs:choice>
        </xs:sequence>
        <xs:attribute name="ns" type="xs:string" use="required" />
        <xs:attribute name="node" type="xs:string" use="required" />
        <xs:attribute ref="xml:base" />
    </xs:complexType>
    <xs:complexType name="TopicExpressionList">
        <xs:sequence minOccurs="1" maxOccurs="unbounded">
            <xs:element name="topic" type="Expression" />
        </xs:sequence>
        <xs:attribute name="publish" type="RuleQualifier" use="optional" />
        <xs:attribute name="subscribe" type="RuleQualifier" use="optional" />
        <xs:attribute ref="xml:base" />
    </xs:complexType>
    <xs:complexType name="ServicesExpressionList">
        <xs:sequence minOccurs="1" maxOccurs="unbounded">
            <xs:element name="service" type="Expression" />
        </xs:sequence>
        <xs:attribute name="reply" type="RuleQualifier" use="optional" />
        <xs:attribute name="request" type="RuleQualifier" use="optional" />
        <xs:attribute ref="xml:base" />
    </xs:complexType>
    <xs:complexType name="ActionsExpressionList">
        <xs:sequence minOccurs="1" maxOccurs="unbounded">
            <xs:element name="action" type="Expression" />
        </xs:sequence>
        <xs:attribute name="call" type="RuleQualifier" use="optional" />
        <xs:attribute name="execute" type="RuleQualifier" use="optional" />
        <xs:attribute ref="xml:base" />
    </xs:complexType>
    <xs:simpleType name="Expression">
        <xs:restriction base="xs:string" />
    </xs:simpleType>
    <xs:simpleType name="RuleQualifier">
        <xs:restriction base="xs:string">
            <xs:enumeration value="ALLOW" />
            <xs:enumeration value="DENY" />
        </xs:restriction>
    </xs:simpleType>
</xs:schema>
```
