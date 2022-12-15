作者：梁志成

# **Relation to other standards**

ISO23150是可应用于自动驾驶功能的标准。它规定了智能传感器与融合单元之间的逻辑接口。该接口以模块化、语义表示进行描述，允许不同类型的传感器技术和融合概念。<br />OSI和AutoSAR ADI支持ISO23150标准，是ISO23150的具体实现。<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669785457757-63fa3352-fea6-460b-856c-1e5a23e75be7.png#averageHue=%23f9f8f7&clientId=uc3ffa2d4-bf49-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uf5e875d4&margin=%5Bobject%20Object%5D&originHeight=506&originWidth=922&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u80b4e7ea-6fc0-4892-ac7c-1ebe002b078&title=)

# **Scope of Sensor Interface Standardization**

AP传感器接口的标准化旨在创建一个广为接受的规范，该规范基于并符合国际标准化组织（ISO）发布的传感器接口规范。虽然ISO规范主要关注不同传感器接口的语义定义，但AUTOSAR AP的ADI规范涵盖了所有其他方面，使其完全符合AP。这包括实现的所有语法元素以及传感器配置等附加功能。<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669785457812-872a7108-3af9-4e2b-8cab-107d859fc180.png#averageHue=%23cfe2bd&clientId=uc3ffa2d4-bf49-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uff253cbf&margin=%5Bobject%20Object%5D&originHeight=433&originWidth=880&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ubfc9611b-7a05-4af1-a721-aa1410d3a86&title=)<br />本规范有意排除传感器接口的以下方面，以避免施加限制：机械和电气接口、裸数据层接口。<br />对于读取和写入裸数据流，SWS Communication Management中定义了AP应用程序的API。<br />传感器通过服务接口连接到AUTOSAR自适应计算单元。

# **Sensor Interface API Design**

## **Sensor Interface realization as a Service**

已决定将传感器接口实现为服务。主要原因是在一台部署了不同平台(CP和AP)的车辆上通用地提供传感器服务。<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669785457772-7f64d674-a96a-4ec0-ad7e-5423f3e869e5.png#averageHue=%23f9f4f1&clientId=uc3ffa2d4-bf49-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u670ec024&margin=%5Bobject%20Object%5D&originHeight=555&originWidth=916&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc65e842c-f0cb-4285-bea9-97d39c5f2b6&title=)

## **ISO mapping to Sensor Services**

ISO定义了智能传感器的语法和内容。这些是独立于传感器技术的跟踪、路标和地标的object level列表。此外，还有一个传感器技术相关的特征列表用于检测。传感器服务设计直接映射此结构。因此，每个智能传感器都提供跟踪、路标和地标服务，以及取决于传感器技术的检测服务。<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669785457642-30e9b92f-9c51-4afa-a766-1f4b39b6cc7d.png#averageHue=%23fbf1ee&clientId=uc3ffa2d4-bf49-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u6aba36d9&margin=%5Bobject%20Object%5D&originHeight=333&originWidth=895&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ua8fd361f-a836-4460-a4f0-ceb3fcfd9e5&title=)

## **Sensor Service Template**

智能传感器的单个服务模板设计面临着ISO列表包含大量可选元素的挑战。这意味着服务模板具有强制和可选元素。这里的关键假设是传感器供应商和用户在设计时已知并固定了可选元素。有一个服务能力向量，指示传感器提供的可选元素。此外，相同类型的传感器可以具有不同的性能，即，也可以使用具有相同能力设置的不同分辨率的数据类型。这意味着服务模板具有不同的配置文件，例如低配置文件和高配置文件。下图显示了此服务模板的概念视图，其中包含能力向量和服务配置文件。<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669785457667-d047a5f1-8757-42f8-8ff6-9b17fc025ec5.png#averageHue=%23b0c3ad&clientId=uc3ffa2d4-bf49-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u48994485&margin=%5Bobject%20Object%5D&originHeight=593&originWidth=1130&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc66294d0-87b2-49e9-bf37-bfc2f9986a4&title=)<br />传感器服务提供商和用户之间的服务兼容性在部署阶段由集成商检查，或在服务发现阶段由用户检查。下面的序列图概述了服务发现阶段的矢量能力检查。服务提供商提供服务后，用户将向其注册。然后，用户将向提供商请求能力向量，以检查所需的可选元素是否由传感器的服务提供。如果支持（提供）所需的可选元素，则客户端将订阅服务并根据能力向量的设置处理数据包。在初始化阶段检查后，服务提供商和用户代码不检查可选状态。<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669785458428-b4cd0ccc-2647-4f8c-b309-798449d05f6b.png#averageHue=%23b79770&clientId=uc3ffa2d4-bf49-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u986ecdc1&margin=%5Bobject%20Object%5D&originHeight=540&originWidth=1089&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc48ff426-e786-4652-91d3-bddaf363faf&title=)

## **Interface stability**

自动驾驶的驱动因素之一是以合理的价格提供合适的传感器。因此，这一领域的研究和开发规模巨大。新的传感器、新的精度可能性或更高的采样率使得很难定义一个持续很长时间的接口。

## **Interface configuration**

传感器接口连接的配置可以在三个不同的时间进行：<br />Design time：<br />架构师或开发人员指定整个系统并配置所有连接。然后生成接口绑定，编译整个软件并将其闪存到完整的环境中。这也可能需要传感器的软件更新。<br />Connection time<br />指定系统并配置连接后，可以为一个连接指定多个接口版本。换句话说，配置在“Design time”并不完整。这意味着服务可能必须实现多个版本的接口。如果系统正在通电，客户端将启动握手并确定最适合连接的接口版本。此过程完成后，系统启动最终检查以验证完整配置。然后，可以达到与Design time方法相同的安全级别。<br />Runtime<br />客户端可以在系统运行时连接到任何可用接口。这创造了巨大的灵活性，但也降低了系统的确定性行为。例如，如果没有额外的对策，就无法保证连接的带宽使用。这使得安全分析变得复杂，因为需要评估更多的配置变量。

结论是，Runtime方法不适用于自驾系统，因为安全分析将变得过于复杂，成本将显著增加。<br />Design time配置已经使用了几十年，并且在创建安全系统方面有很多经验。然而，由于接口在未来几年可能会经历大量变化，因此需要更多的灵活性。如果在总体设计中没有引入这一点，将创建变通方法来克服这一限制。<br />因此，我们的想法是使用Connection time配置来提供灵活性，这是简化开发和避免变通方案所必需的。
