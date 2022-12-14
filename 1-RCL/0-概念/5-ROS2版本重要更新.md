# ros2-版本重要更新

# 参考

ROS 2 Rolling Ridley (codename ‘rolling’; June 2020) — ROS 2 Documentation: Foxy documentation<br /><https://docs.ros.org/en/foxy/Releases/Release-Rolling-Ridley.html>

# Crystal重要更新

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664174001371-e14c0e6e-d4ec-4c4d-96db-b906033f3764.png#averageHue=%23f9f8f8&clientId=ucd2f532e-6a89-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u0b9be9f4&margin=%5Bobject%20Object%5D&originHeight=457&originWidth=504&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=udc68f94d-0c41-4e47-9fbf-b5147eec5ba&title=)<br />rcl_context_t在该版本产生，目的是为node初始化提供更多选项，更细粒度的配置。<br />There are two new structs, the rcl_context_t and the rcl_init_options_t, which are used with rmw_init. The init options struct is used to pass options down to the middleware and is an input to rmw_init. The context is a handle which is an output of rmw_init function is used to identify which init-shutdown cycle each entity is associated with, where an “entity” is anything created like a node, guard condition, etc.<br />The init options are passed into rcl_init as an input and the context is passed in as an output. The context is used to associate all other rcl entities to a specific init-shutdown cycle, effectively making init and shutdown no longer global functions, or rather those functions no longer use an global state and instead encapsulate all state within the context type.<br />![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664174001397-91a3ae3b-a167-4931-8a25-d17a5c276602.png#averageHue=%23f9f7f6&clientId=ucd2f532e-6a89-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u6b1f29c9&margin=%5Bobject%20Object%5D&originHeight=865&originWidth=739&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uef6d0991-4b54-4cd6-9457-140ca2c1480&title=)

# Dashing重要更新

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664174001472-dbdf950c-4e21-4a6f-a978-d70560f6ac36.png#averageHue=%23f9f8f8&clientId=ucd2f532e-6a89-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u0989a573&margin=%5Bobject%20Object%5D&originHeight=573&originWidth=709&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc863da8e-0ea0-4663-8ea1-4f6b0bd87b2&title=)

- components的节点编程方式
- intra-process通信性能改善
- Parameters用来代替dynamic_reconfigure
- 可以使用idl来定义消息
- 增加actions和components
- 支持Deadline, Lifespan & Liveliness quality of service settings.

# Eloquent重要更新

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664174001343-8e70a364-eb5a-460a-b300-0d50a3da4a86.png#averageHue=%23f7f7f6&clientId=ucd2f532e-6a89-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u0ab3c149&margin=%5Bobject%20Object%5D&originHeight=439&originWidth=606&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u4f99fd33-2f22-4205-8c24-bf76080531b&title=)

- source setup.bash性能改进
- rqt功能更新
- 增加cyclonedds中间件
- ROS_LOCALHOST_ONLY限制网络通信
- 增加ros2_tracing
- rclcpp大改
- ros2msg and ros2srv deprecated
- Service clients have been added to ros2node info

# Foxy重要更新

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664174003067-bbf58852-4640-45bd-adcd-52a725165484.png#averageHue=%23f9f9f9&clientId=ucd2f532e-6a89-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u10e95fda&margin=%5Bobject%20Object%5D&originHeight=670&originWidth=642&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ue6c2d412-404f-463e-bd0b-3cddfc959d1&title=)

- 总体功能稳定，性能稳定
- **iceoryx可用zero_copy**
- **participant 不再与node一一对应**
- <https://design.ros2.org/articles/Node_to_Participant_mapping.html>
- security功能推出，enclave

# Galactic重要更新

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664174003086-6fe65c4c-7638-48d2-9d53-da1ee4226787.png#averageHue=%23faf9f9&clientId=ucd2f532e-6a89-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u4e1febd7&margin=%5Bobject%20Object%5D&originHeight=948&originWidth=693&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ua56eb390-b1f3-49d7-bd97-d0979619878&title=)

- rosbag2性能提升
- **rosbag2可以/clock**
- 这是一个模拟时间，可以加速减速播放
- **对每个节点配置日志目录**
- **在配置文件中配置qos**
- **cyclonedds 支持zero_copy**
- 支持单一网络流
- 运行时加载parameter文件
- 对节点配置日志等级
- **启动时配置qos**
- ros2 topic可以打印串行化数据
- 获取消息的yaml格式打印
- 检查QoS的不兼容性
- 参数文件中使用参数替换Use launch substitutions in parameter files

# Humble重要更新

![](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664174003129-3dc5acd0-d65d-4951-afd9-96df0f7eb59f.png#averageHue=%23fbfbfb&clientId=ucd2f532e-6a89-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u95e6a498&margin=%5Bobject%20Object%5D&originHeight=697&originWidth=546&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uef8deed1-005d-47b7-9f35-a1d7f0502ba&title=)

- contented filtered topics主题过滤话题
- ros2 launch --launch-prefix
- ros2 topic echo --filter 过滤输出
- ROS_DISABLE_LOANED_MESSAGES取消loaned_msg零拷贝
- TypeAdpater直接支持消息类型转换REP2007
- publisher可以知道发送成功
- pub->wait_for_all_acked(); // or pub->wait_for_all_acked(timeout)
- reliable qos有，best effort没有
- lifecycle publisher自动进入activate状态
- rosbag2 支持发送loaned_msg
