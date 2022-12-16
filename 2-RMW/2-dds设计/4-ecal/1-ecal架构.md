# Overview

eCAL == enhanced Communication Abstraction Layer

middleware for high performance and scalable inter-process communication on single computational nodes or in heterogeneous networks

designed for minimal latency and high data throughput

lightweight API for message transport only

operates on a wide range of hardware platforms from high end server machines to ARM based embedded hardware

easy integration in different computing languages and frameworks

# Features

loose connection of all components via publish / subscribe pattern

all participants synchronize all information automatically, no central demon instance

different transport layers (inner-process, shared memory, udp unicast/multicast)

support a subset of quality of services (depends from transport layer mode)

native support of different serialization formats:

google::protobuf (monitor reflection supported)

capnproto (monitor reflection supported)

google::flatbuffers,

messagepack,

json ..

application eco system:

eCALMon: monitoring interface for real-time diagnostic and message debugging

eCALRec: recording distributed in an eCAL network or on a central host

eCALPLay: message replay with modern user interface or via command line

# Arch

![](https://tcs.teambition.net/storage/312j921a0b39f2f77869630d8b1fb8e42074?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODA1NiwiaWF0IjoxNjcxMTkzMjU2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmo5MjFhMGIzOWYyZjc3ODY5NjMwZDhiMWZiOGU0MjA3NCJ9.8C3RxoKOTCfrrgDWXyRlX5BbYI3C2KXNfEcsTeH1vik&download=image.png "")

# Transport layers

inner process

ultra fast, reliable, single threaded, single process

shared memory
ultra fast, none reliable, highest throughput for 1 to n scenarios, multi threaded, multi process

udp multicast (can use multiple multicast groups for data transport)

performance depends on ethernet stack, none reliable, single threaded, multi process / hosts

Iceoryx (<https://github.com/eclipse/iceoryx>)

Bosch zero copy shared memory transport layer (ipc only)

# Serialization support

![](https://tcs.teambition.net/storage/312j877baa183b3f3c3274739a615152d0f2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5ODA1NiwiaWF0IjoxNjcxMTkzMjU2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmo4NzdiYWExODNiM2YzYzMyNzQ3MzlhNjE1MTUyZDBmMiJ9.xHYTg6eiNn87lrGcKhRwRZwdneqv3ENdt4DRG54XtZk&download=image.png "")

# 线程和执行模型

eCAL is using different transport layers for intraprocess, interprocess or interhost communication. With a call of `eCAL::Initialize` different internal threads are started, with a call of `eCAL::Finalize` they are stopped:

每一个进程会初始化下列线程

- udp monitoring send thread

- udp registration send thread

- udp registration receive thread

- udp logging receive thread

- udp message (payload) receive thread

For the eCAL user interface entities `eCAL::CTimer`, `eCAL::CSubscriber`, `eCAL::CServiceServer`, `eCAL::CServiceClient` additional threads are utilized:

对于额外的应用实体，会有额外的线程

- shared memory synchronization event thread (1 thread per handled memory file)

- timer callback (1 thread per callback)

- tcp client/server implementation (2 threads per instance)
