---
title: 分布式计算模型ActorModel
published: false
author: '[ZhenshengLee](https://github.com/ZhenshengLee)'
reviewer: 'none'
---

# 动机

Another way to look at this architecture is that of distributed computing, communicating over some form of message passing. At large, all these represent the same concept. This message passing, communication-based architecture can also benefit from the rise of fast and high-bandwidth communication such as Ethernet. ---------From AP EXP_PlatformDesign  3.3.2

Trellis is based on the actor model, in which actors can only effect each other's state indirectly through the passing of messages.<br />See: [https://en.wikipedia.org/wiki/Actor_model](https://en.wikipedia.org/wiki/Actor_model) ----------From  [https://github.com/agtonomy/trellis#actor-model](https://github.com/agtonomy/trellis#actor-model)

# 介绍

分布式计算技术之Actor计算模式<br />The Actor Model for Concurrent Computation<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016118227-e6adee73-2d4e-4bd3-8209-fa2ae63e5f11.png#averageHue=%23e3e3e3&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&errorMessage=failed%20to%20set%20src&from=paste&id=ue29e4133&margin=%5Bobject%20Object%5D&name=image.png&originHeight=343&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=99440&status=error&style=none&taskId=u6f8e8139-4f39-4241-976d-0d76f60295a&title=)

# 背景

计算机CPU的计算速度提高(频率的提高)是有限度的，我们能做的是放入多个计算核心。为了利用多核心的计算机，我们需要并发执行。但是多线程的方式会引入很多问题和增加调试难度。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016132231-d5cf0844-8c69-4cc3-b2dd-f3812d0d89b8.png#averageHue=%23e5e5e5&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uf2b8ca28&margin=%5Bobject%20Object%5D&name=image.png&originHeight=470&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=123956&status=done&style=none&taskId=u72ac07fb-6b7d-42b5-bfbc-aaf2bbdbe20&title=)<br />我们有个替换的方案，叫做Actor模型。

# 两种分布式计算模式: Actor 和流水线

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016169882-b1be2ab8-6142-4cec-81a4-0e02357e8f65.png#averageHue=%23e4e4e4&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ue08ff1b8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=615&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=220680&status=done&style=none&taskId=uc30b2afa-c24a-4b56-8c15-c2789456c10&title=)

分布式计算的本质就是在分布式环境下，多个进程协同完成一件复杂的事情，但每个进程各司其职，完成自己的工作后，再交给其他进程去完成其他工作。当然，对于没有依赖的工作，进程间是可以并行执行的。<br />分布式进程那么多，如果需要开发者自己去维护每个进程之间的数据、状态等信息，这个开发量可不是一般得大，而且特别容易出错。那么，有没有什么办法可以让开发者只关注自己的逻辑呢？<br />答案是肯定的，Actor 计算模式就能满足你的需求。也就是说，你可以把数据、状态等都扔给 Actor。

# 什么是 Actor？

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016208803-746ea81b-0160-4983-a5be-413aa4aa48c0.png#averageHue=%23f3f2ef&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uf8307930&margin=%5Bobject%20Object%5D&name=image.png&originHeight=624&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=432785&status=done&style=none&taskId=u1f100092-a285-4bb4-a317-10fb8f9c720&title=)<br />Actor 类似于一个“黑盒”对象，封装了自己的状态和行为，使得其他 Actor 无法直接观察到它的状态，调用它的行为。多个 Actor 之间通过消息进行通信，这种消息类似于电子邮箱中的邮件。Actor 接收到消息之后，才会根据消息去执行计算操作。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016220545-fbeaca40-42fd-4cdb-a3ba-9e135c4f66f2.png#averageHue=%23e2e2e2&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub88545f1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=710&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=361908&status=done&style=none&taskId=u3846f638-3c84-44cb-a014-34a1abc90a3&title=)<br />那么，Actor 模型又是什么呢？<br />Actor 模型，代表一种异步消息模式的分布式并行计算模型。在 Actor 模型里，每个 Actor 相当于系统中的一个组件，都是基本的计算单元。<br />这种模型有自己的一套规则，规定了 Actor 的内部计算逻辑，以及多个 Actor 之间的通信规则。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016235290-49f1eb43-29cd-4286-966f-c3b2909ec172.png#averageHue=%23f4f4f4&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u05da314e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=787&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=179881&status=done&style=none&taskId=u0b798d89-ffa8-4555-bbfe-5e818d4d9ab&title=)<br />Actor 模型的计算方式与传统面向对象编程模型(Object-Oriented Programming，OOP)类似，一个对象接收到一个方法的调用请求(类似于一个消息)，从而去执行该方法。<br />OOP 因为数据封装在一个对象中，不能被外部访问，当多个外部对象通过方法调用方式，即同步方式进行访问时，会存在死锁、竞争等问题，无法满足分布式系统的高并发性需求。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016247207-70aa1c6e-316a-4e59-bc1f-aaa65e43326c.png#averageHue=%23dbdbdb&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u53de48bb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=491&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=195703&status=done&style=none&taskId=ub5c4a54c-1ecb-49da-bb3a-3ee2929fac8&title=)<br />而 Actor 模型通过消息通信，采用的是异步方式，克服了 OOP 的局限性，适用于高并发的分布式系统。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016259195-2a293ee7-0720-442d-b8bf-10f4094f746a.png#averageHue=%23f2f2f2&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u9b3082ea&margin=%5Bobject%20Object%5D&name=image.png&originHeight=665&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=191477&status=done&style=none&taskId=uffc24b36-faed-489f-b002-96972a99f0c&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016266636-5cf8357b-e583-4a4d-a96b-6a6386ffa3f7.png#averageHue=%23eee8e7&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u4bb7f256&margin=%5Bobject%20Object%5D&name=image.png&originHeight=743&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=344677&status=done&style=none&taskId=ub0486ef5-a336-493b-99b3-449684ccecb&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016274733-173660c4-4237-41f0-86ed-ef66f582ff4d.png#averageHue=%23f1efef&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u2d522369&margin=%5Bobject%20Object%5D&name=image.png&originHeight=611&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=187355&status=done&style=none&taskId=u02e9088d-3ea6-4e6f-9eac-c36515d0584&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016281373-74dff698-f416-4e24-8610-351632952adb.png#averageHue=%23f6f7f4&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ufc5a184f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=728&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=307621&status=done&style=none&taskId=u8c98bd77-caef-418a-b5e4-f5d5a532b59&title=)

# Actor模型

Actor模型是处理并行计算的概念模型。它定义了系统部件行为和交互的一些规则。使用这个模型的最著名的编程语言是Erlang。<br />模型中一个Actor是一个基本的计算单元。它接受消息然后基于接到的消息做一些计算。和面向对象编程有些类似，一个对象被调用(接收到一个消息)，基于调用方法(接受到的一个消息)做处理。区别是actor之间是完全隔离的，不共用内存区域。actor的私有状态不会被另外一个actor直接改变。<br />actor作为群体存在，单一的actor不是actor模式。在actor模型中，actor是唯一组成部分，actor带有地址以便互相发送消息。<br />actor按次序处理消息，比如你发送三个消息给一个actor，它们不会被并发处理。如果你想让这三个消息得到并发处理，你需要创建3个actor，然后分别发送给它们。<br />接受到的异步消息存在于actor内部的一个队列中，我们可以把它形象化的叫做邮箱(mailbox)。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016302672-fbe06b17-0003-431b-ad2f-64ba55f5fa14.png#averageHue=%23f9f7dd&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u6c62edf2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=890&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=426129&status=done&style=none&taskId=u8447bdab-2c1e-4d63-8fd2-cb18d878885&title=)

# Actor模型定义

一种分布式并行计算模型。<br />该模型有自己的一套规则，规定了Actor的内部计算逻辑以及多个Actor之间的通信规则。Actor模型通过异步消息模式，实现分布式系统的高并发

# Actor计算模式

Actor模型的三要素：<br />状态：Actor组件本身的信息<br />行为：Actor的计算处理操作<br />消息：Actor的消息以邮件的形式在多个Actor之间通信传递，每个Actor都会有一个自己的邮箱.<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016320043-8e2a7803-265e-4364-9aad-5accfb15ae38.png#averageHue=%23dad9d9&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u3846305d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=652&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=373043&status=done&style=none&taskId=ua4b5dac0-c670-4481-bac5-c52a0aac80c&title=)

# Actor工作原理

## 工作流程

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016334130-02bafe70-c0a2-4037-bb0f-20bac61c7ff3.png#averageHue=%23fbfdfc&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u48fb1314&margin=%5Bobject%20Object%5D&name=image.png&originHeight=600&originWidth=893&originalType=url&ratio=1&rotation=0&showTitle=false&size=176121&status=done&style=none&taskId=u046be66f-db51-4eae-a4e2-7953f36ac2b&title=)

Actor2从MailBox队列中依次取出Actor1和Actor3发送的消息执行相应的操作

## 消息传递流程

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016357895-36983c18-3c39-4150-9352-7ce38028d344.png#averageHue=%238da8d7&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u2282f2a1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=563&originWidth=896&originalType=url&ratio=1&rotation=0&showTitle=false&size=173646&status=done&style=none&taskId=uaff84a7b-c106-4f2e-be5a-46736ab5541&title=)

# Actor的行为

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016385316-8cc8d7c5-b959-42e6-8cfc-1d6145965708.png#averageHue=%23d3d3d3&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uffc887eb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=730&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=359477&status=done&style=none&taskId=u674aa819-860f-47db-8254-7968d2bcdbb&title=)

当一个actor接受到消息后，它可以做如下事情：<br />创建更多的actor<br />发送消息到其他actor<br />指派对下一条消息做什么处理。<br />最后一条定义了actor如何操作内部状态。例如一个计算器作用的actor，初始状态是0，处理到加1消息时，它不改变内部状态，但分派下一条消息处理时，状态是1.

## 失败可容忍

Erlang语言中有个“由它失败”的思想。就是你不可能考虑到所有导致失败的问题，与其绞尽脑汁处理这些问题，不如让它自然失败，然后指派给失败处理者处理(例如恢复到稳定状态)，在actor模型中，这是可行的。<br />actor之间的隔离性导致actor失败不会影响其他actor，监控者可以对自然失败的actor做直接处理而不会带来连锁问题。这让“自愈系统”成为可能，就是说一个actor异常后，监控者可以恢复一致性，可能以初始状态重起actor。

## 分布性

actor发消息时不在乎目标actor是本地运行的还是运行在其他节点。试想，如果actor只是只有内部状态的邮箱，只对消息做出反应，那就没人在乎它在哪里运行，知道有个地址让消息可以到达即可。这让我们可以创建分布式系统，并且在节点失败时独立恢复而不影响整个系统。<br />进一步建议看一下Erlang和Elixir语言， JVM上的 Actor库Akka，基于Actor的框架Vert.x

# Actor关键特征

## 优势

Actor 的通信机制与日常的邮件通信非常类似。因此，我们可以进一步总结出 Actor 模型的一些特点：<br />1.实现了更高级的抽象<br />Actor 与 OOP 对象类似，封装了状态和行为。但是，Actor 之间是异步通信的，多个 Actor 可以独立运行且不会被干扰，解决了 OOP 存在的竞争问题。<br />2.非阻塞性<br />在 Actor 模型中，Actor 之间是异步通信的，所以当一个 Actor 发送信息给另外一个 Actor 之后，无需等待响应，发送完信息之后可以在本地继续运行其他任务。也就是说，Actor 模型通过引入消息传递机制，从而避免了阻塞。<br />3.无需使用锁<br />Actor 从 MailBox 中一次只能读取一个消息，也就是说，Actor 内部只能同时处理一个消息，是一个天然的互斥锁，所以无需额外对代码加锁。<br />4.并发度高<br />每个 Actor 只需处理本地 MailBox 的消息，因此多个 Actor 可以并行地工作，从而提高整个分布式系统的并行处理能力。<br />5.易扩展<br />每个 Actor 都可以创建多个 Actor，从而减轻单个 Actor 的工作负载。当本地 Actor 处理不过来的时候，可以在远程节点上启动 Actor 然后转发消息过去。

## 不足

虽然 Actor 模型有上述的诸多优点，但它并不适用于分布式领域中所有的应用平台或计算框架。因为，Actor 模型还存在如下一些不足之处：<br />Actor 提供了模块和封装，但缺少继承和分层，这使得即使多个 Actor 之间有公共逻辑或代码部分，都必须在每个 Actor 中重写这部分代码，也就是说重用性小，业务逻辑的改变会导致整体代码的重写。<br />Actor 可以动态创建多个 Actor，使得整个 Actor 模型的行为不断变化，因此在工程中不易实现 Actor 模型。此外，增加 Actor 的同时，也会增加系统开销。<br />Actor 模型不适用于对消息处理顺序有严格要求的系统。因为在 Actor 模型中，消息均为异步消息，无法确定每个消息的执行顺序。虽然可以通过阻塞 Actor 去解决顺序问题，但显然，会严重影响 Actor 模型的任务处理效率。<br />可重用性低，业务逻辑的改变会导致整体代码的重写<br />工程上难以实现<br />不适用于对消息处理顺序有严格要求的系统

# Actor 主要应用

1.Erlang/OTP: Erlang是面向并发的编程语言，OTP是Erlang技术栈中的标准库<br />2.Akka: 为java和scala构建高度并发、分布式和弹性的消息驱动应用程序的工具包<br />Quasar(Java): 开源的JVM库<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016460909-3128cb41-fce6-4e92-b437-ee70dc161efa.png#averageHue=%23eeeeee&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ua73fca07&margin=%5Bobject%20Object%5D&name=image.png&originHeight=503&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=205496&status=done&style=none&taskId=uabbc6698-bd84-4e3e-9913-37a6535bb24&title=)

## 框架与语言

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1671016469776-616099bd-13a1-4299-b20c-caae9235d0df.png#averageHue=%23e3e3e3&clientId=u5820fe75-35c3-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ucb1e60b9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=567&originWidth=1184&originalType=url&ratio=1&rotation=0&showTitle=false&size=168332&status=done&style=none&taskId=u9662be35-8c15-4c6e-96de-2533f63de8b&title=)

Actor 模型在 1973 年被提出，已广泛应用在多种框架和语言中。可以说，很多框架或语言支持 Actor 编程模型，是为了给开发者提供一个通用的编程框架，让用户可以聚焦到自己的业务逻辑上，而不用像面向对象等编程模型那样需要关心死锁、竞争等问题。<br />那么，到底有哪些框架或语言支持 Actor 编程模型呢？接下来，和你列举几个典型的框架或语言吧，以方便你参考。<br />Erlang/OTP。Erlang 是一种通用的、面向并发的编程语言，使用 Erlang 编写分布式应用比较简单，而 OTP 就是 Erlang 技术栈中的标准库。Actor 模型在 Erlang 语言中得到广泛支持和应用，其他语言的 Actor 逻辑实现在一定程度上都是参照了 Erlang 的模式。实现了 Actor 模型逻辑的 Erlang/OTP，可以用于构建一个开发和运行时环境，从而实现分布式、实时的、高可用性的系统。<br />Akka。Akka 是一个为 Java 和 Scala 构建高度并发、分布式和弹性的消息驱动应用程序的工具包。Akka 框架基于 Actor 模型，提供了一个用于构建可扩展的、弹性的、快速响应的应用程序的平台。通过使用 Actors 和 Streams 技术， Akka 为用户提供了多个服务器，使用户更有效地使用服务器资源并构建可扩展的系统。Quasar (Java) 。<br />Quasar 是一个开源的 JVM 库，极大地简化了高度并发软件的创建。Quasar 在线程实现时，参考了 Actor 模型，采用异步编程逻辑，从而为 JVM 提供了高性能、轻量级的线程，可以用在 Java 和 Kotlin 编程语言中.<br />高可用: 分布式集群容错

# 参考

[https://blog.51cto.com/u_15236724/5366007](https://blog.51cto.com/u_15236724/5366007)<br />[https://fuzhe1989.github.io/2018/05/17/message-passing-and-the-actor-model/](https://fuzhe1989.github.io/2018/05/17/message-passing-and-the-actor-model/)

