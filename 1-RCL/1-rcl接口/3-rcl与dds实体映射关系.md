# 概念

## Node

In ROS, a Node is an entity used to group other entities. For example: Publishers, Subscriptions, Servers, Clients. Nodes ease organization and code reuse, as they can be composed in different ways.

在ROS中, node是一个实体，包含其他实体，pub, sub, server, clients

### Domain Participant

A Participant is a type of DDS entity. Participant also group other entities, like Publishers, Subscribers, Data Writters, Data Readers, etc. Creating more Participants adds overhead to an application:

- Each Participant participates in discovery.**Creating more than one Participant usually increases CPU usage and network IO load**.

- Each Participant keeps track of other DDS entities. **Using more than one within a single process may result in data duplication**.

- **Each Participant may create multiple threads for event handling, discovery**, etc. The number of threads created per Participant depends on the DDS vendor (e.g.: [RTI Connext](https://community.rti.com/best-practices/create-few-domainParticipants-possible)).

For those reasons, a Participant is a heavyweight entity.

participant是一个很重的实体

Note: This might actually depend on the DDS vendor, some of them share these resources between Participants (e.g. `OpenSplice`). Many DDS vendors, however, do not perform this kind of optimization (e.g.: `RTI Connext` and `Fast-RTPS`), and actually recommend creating just one Participant per process.

[2-dds中的线程优先级](https://thoughts.teambition.com/share/6268e5650c85220041ebb168#title=2-dds%E4%B8%AD%E7%9A%84%E7%BA%BF%E7%A8%8B%E4%BC%98%E5%85%88%E7%BA%A7)

### Context

在Crystal创建该类型，最初用来给node配置选项，后来作为一个比node 更高层的实体概念，一般来讲，一个进程只有一个context，default global context

In ROS, a Context is the non-global state of an init-shutdown cycle. It also encapsulates shared state between Nodes and other entities. In most applications, there is only one ROS Context in a process.

# Foxy前的实现

There is a one-to-one mapping between Nodes and DDS Participants. This simplified the original implementation, as DDS Participants provide many features equivalent to the ones of ROS Nodes. The drawback of this approach is the overhead that comes with creating many Participants. Furthermore, the maximum number of Domain Participants is rather small. For example, in [RTI Connext](https://community.rti.com/kb/what-maximum-number-Participants-domain) it is limited to 120 Participants per Domain.

# Foxy新实现

## Mapping of DDS Participant to a ROS entity

There are two alternatives, besides the one-to-one Node to Participant mapping used pre-Foxy:

- Using one Participant per process.

- Using one Participant per Context.

The second approach is much more flexible, allowing more than one Participant in a single application for those that need it e.g. domain bridge applications. Thus, a one-to-one Participant to Context mapping was chosen.

When multiple Nodes are running in a single process, there are different options for grouping them by - ranging from a separate context for each Node, over grouping a few Nodes in the same context, to using a single context for all Nodes. For most applications, only one Context is created.

新的实现是一个context一个paticipant，participant会创建一系列的dds线程，在rcl_init执行的时候产生

### Discovery information

If a one-to-one Node to Participant mapping is not used, extra discovery information is needed to be able to match other entities to a Node e.g. Publishers, Subscriptions, etc. Several approaches can be used to share this information. The proposed approach uses a topic for it. Each Participant publishes a message with all the information needed to match an entity to a Node. The message structure is the following:

- ParticipantInfo

  - gid

  - NodeInfo

    - Node namespace

    - Node name

    - Reader gid

    - writed gid

When one entity is updated (e.g.: a Publisher is created or destroyed), a new message is sent.

Identification of Clients and Servers happens according to the ROS conventions for their topic names (see [Topic and Service name mapping to DDS](https://design.ros2.org/articles/140_topic_and_service_name_mapping.md)).

This topic is considered an implementation detail, and not all `rmw` implementations have to use it. Thus, all the necessary logic has to be in the rmw implementation itself or in an upstream package. Implementing this logic in `rcl` would make it part of the API, and not an implementation detail.

To avoid code repetition, a common implementation of this logic is provided by the [rmw_dds_common](https://github.com/ros2/rmw_dds_common/) package.

Details of the ROS discovery topic

- topic name: `ros_discovery_info`

- Writer qos:

  - durability: transient local

  - history: keep last

  - history depth: 1

  - reliability: reliable

- Reader qos:

  - durability: transient local

  - history: keep all

  - history depth: 1

  - reliability: reliable

# 参考

[Node to Participant mapping](https://design.ros2.org/articles/Node_to_Participant_mapping.html)

[Reconsidering 1-to-1 mapping of ROS nodes to DDS participants](https://discourse.ros.org/t/reconsidering-1-to-1-mapping-of-ros-nodes-to-dds-participants/10062/16)
