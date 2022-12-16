# SIGINT信号基础知识

用户输入命令，在Shell下启动一个前台进程。<br />用户按下Ctrl-C，这个键盘输入产生一个硬件中断。<br />如果CPU当前正在执行这个进程的代码，则该进程的用户空间代码暂停执行，CPU从用户态切换到内核态处理硬件中断。<br />终端驱动程序将Ctrl-C解释成一个SIGINT信号，记在该进程的PCB中（也可以说发送了一个SIGINT信号给该进程）。<br />当某个时刻要从内核返回到该进程的用户空间代码继续执行之前，首先处理PCB中记录的信号，发现有一个SIGINT信号待处理，而这个信号的默认处理动作是终止进程，所以直接终止进程而不再返回它的用户空间代码执行。<br />注意，Ctrl-C产生的信号只能发给前台进程。一个命令后面加个&可以放到后台运行，这样Shell不必等待进程结束就可以接受新的命令，启动新的进程。Shell可以同时运行一个前台进程和任意多个后台进程，只有前台进程才能接到像Ctrl-C这种控制键产生的信号。前台进程在运行过程中用户随时可能按下Ctrl-C而产生一个信号，也就是说该进程的用户空间代码执行到任何地方都有可能收到SIGINT信号而终止，所以信号相对于进程的控制流程来说是异步（Asynchronous）的。

## 信号产生

- 通过终端按键产生信号
- 调用系统函数向进程发信号

首先在后台执行死循环程序,然后用kill命令给它发SIGSEGV信号。

## 信号处理

可选的处理动作有以下三种:

- 忽略此信号。
- 执行该信号的默认处理动作。
- 提供一个信号处理函数,要求内核在处理该信号时切换到用户态执行这个处理函数,这种方式称为捕捉(Catch)一个信号。

SIGINT的默认处理动作是终止进程,SIGQUIT的默认处理动作是终止进程并且Core Dump.

## 一般应用SIGINT处理流程

```cpp
class MyClass
{
 MyClass(){
        // constructor
    }
 ~MyClass(){
        // destructor
    }
}
int threadFun()
{
    while(...)
    {
        ...
        break;
    }
}
int main()
{
    MyClass myObject();
    auto myThread = std::thread(threadFun());
    myThread.join();
    return 0;
}
```

# ROS2框架SIGINT信号处理源码

## rclcpp

galactic\src\ros2\rclcpp\rclcpp\src\rclcpp\utilities.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663810351925-d07428af-91e4-4dff-affd-f20e3bf0917a.png#averageHue=%23252b34&clientId=uc818d1a4-4c63-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=223&id=uEQ95&margin=%5Bobject%20Object%5D&name=image.png&originHeight=223&originWidth=603&originalType=binary&ratio=1&rotation=0&showTitle=false&size=25870&status=done&style=none&taskId=u5182b7d7-776d-47a3-a2bb-c6d0ed66fb2&title=&width=603)<br />galactic\src\ros2\rclcpp\rclcpp\include\rclcpp\init_options.hpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663810227889-731df578-d175-4a06-84c6-ad4902bfd255.png#averageHue=%23242830&clientId=uc818d1a4-4c63-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=118&id=CKIRK&margin=%5Bobject%20Object%5D&name=image.png&originHeight=118&originWidth=801&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10958&status=done&style=none&taskId=u43298b69-45a8-40d7-bdac-b4a57a585e7&title=&width=801)<br />galactic\src\ros2\rclcpp\rclcpp\src\rclcpp\signal_handler.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663638932370-a7c233ee-8b2b-4327-9164-8f75b852fdb6.png#averageHue=%23242931&clientId=u834c1bfa-6a5d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=604&id=RspTa&margin=%5Bobject%20Object%5D&name=image.png&originHeight=604&originWidth=766&originalType=binary&ratio=1&rotation=0&showTitle=false&size=78910&status=done&style=none&taskId=u87cb9527-e891-493e-b35d-18d40ff8861&title=&width=766)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663638990213-9f8d1005-3eae-4db8-9824-7ee752044a1b.png#averageHue=%23242b34&clientId=u834c1bfa-6a5d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=460&id=V1ZWU&margin=%5Bobject%20Object%5D&name=image.png&originHeight=460&originWidth=791&originalType=binary&ratio=1&rotation=0&showTitle=false&size=69692&status=done&style=none&taskId=u22fb9033-ed92-4796-b39e-a0040e32a07&title=&width=791)<br />shutdown_on_sight默认是true<br />context_ptr是整个进程上下文，是rclcpp::init()初始化的<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663640110614-7d159c98-0302-4904-9bb1-c1bc71613ccd.png#averageHue=%23252b34&clientId=u834c1bfa-6a5d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=263&id=pM4L0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=263&originWidth=642&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35225&status=done&style=none&taskId=u6fc4252e-330d-4845-9a26-60d32ea09eb&title=&width=642)<br />galactic\src\ros2\rclcpp\rclcpp\src\rclcpp\utilities.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663810694729-c3f112e3-abe8-4878-b1a3-bc88cc6db406.png#averageHue=%23262c35&clientId=uc818d1a4-4c63-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=267&id=Ok7xt&margin=%5Bobject%20Object%5D&name=image.png&originHeight=267&originWidth=557&originalType=binary&ratio=1&rotation=0&showTitle=false&size=30804&status=done&style=none&taskId=ud077b3ab-eff6-4971-b603-0cd7aadf1ce&title=&width=557)<br />galactic\src\ros2\rclcpp\rclcpp\src\rclcpp\context.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663810928054-8f43c052-91ef-4c83-9a7f-f55ffbc5c6bd.png#averageHue=%23252a33&clientId=uc818d1a4-4c63-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=647&id=NJlpD&margin=%5Bobject%20Object%5D&name=image.png&originHeight=647&originWidth=644&originalType=binary&ratio=1&rotation=0&showTitle=false&size=76740&status=done&style=none&taskId=u342d9467-3afa-43b8-8754-47c9d4e5e0c&title=&width=644)

## rcl

galactic\src\ros2\rcl\rcl\src\rcl\init.c<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663811305194-d8dc942f-f7d0-4a20-83a5-712c8ca2fc0e.png#averageHue=%23242a33&clientId=uc818d1a4-4c63-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=464&id=udf358682&margin=%5Bobject%20Object%5D&name=image.png&originHeight=464&originWidth=726&originalType=binary&ratio=1&rotation=0&showTitle=false&size=69370&status=done&style=none&taskId=u6586a378-1cf5-4a90-8b51-13482f43145&title=&width=726)

## rmw

rmw_dds_common是rmw层的依赖，不是rclcpp层的<br />galactic\src\ros2\rmw_fastrtps\rmw_fastrtps_cpp\src\rmw_init.cpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1663811728740-f3d7dd44-8126-463c-b306-7410d832c4af.png#averageHue=%23262c35&clientId=uc818d1a4-4c63-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=298&id=u6ce15608&margin=%5Bobject%20Object%5D&name=image.png&originHeight=298&originWidth=572&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38084&status=done&style=none&taskId=u77ff42a9-b974-440a-8a0e-edc52f2318b&title=&width=572)

# ROS2应用SIGINT处理流程

## 启动流程

```cpp
int main(int argc, char * argv[])
{
  // rcl全局资源：创建context
  // dds全局资源：participant
  // os资源：
  rclcpp::init(argc, argv);

  using TimeConfig = nodes::timing::Default;
  // uncomment for benchmarking
  // using TimeConfig = nodes::timing::BenchmarkCPUUsage;
  // set_benchmark_mode(true);

  // rcl资源：node资源，机器人计算图graph资源（pub, sub, service, action, topic, timer）
  // dds资源：publisher, subscriber, data_writer和data_reader
  // os资源：用户自定义thread
  // node的定义和初始化由应用开发者开发并负责
  auto nodes = create_autoware_nodes<RclcppSystem, TimeConfig>();

  // os资源：ros2分布式通信的执行线程
  rclcpp::executors::SingleThreadedExecutor executor;
  for (auto & node : nodes) {
    executor.add_node(node);
  }

  // 执行线程开始工作，在主线程中spin，主线程阻塞
  executor.spin();
  // 收到SIGINT信号后executor正常退出

  // vector->clear，调用每个元素的析构函数
  // vector->clear并不能回收内存
  nodes.clear();
  rclcpp::shutdown();

  return 0;
}
```

## ROS2应用SIGINT处理流程

```cpp
  // 执行线程开始工作，主线程阻塞
  executor.spin();
  // 收到SIGINT信号后executor正常退出

  // 此处用户自行开发
  nodes.clear();

  // 销毁全局资源
  rclcpp::shutdown();

  return 0;
```

- rclcpp::init()
  - rcl全局资源：创建context
  - dds全局资源：participant
  - **context中已经安装了signal_handler**
- make_shared<Node>
  - rcl资源：node资源，机器人计算图graph资源（pub, sub, service, action, topic, timer）
  - dds资源：publisher, subscriber, data_writer和data_reader
  - **os资源：用户自定义线程**
- executor初始化
  - 初始化资源，与context绑定
- executor::spin
  - 进入执行线程，遵循proactor前摄器模式
  - 根据executor类型可以有不同的并发模式，可以对回调进行调度
- 用户第一次Ctrl-C
  - context中的sigint_handler处理，context->shutdown
  - executor::spin()结束
  - /用户node对象执行clear()（取决于用户是否调用）
  - rclcpp::shutdown()调用context->shutdown
  - main函数作用域结束
  - node调用析构函数，用户自定义线程结束（不符合预期）
- 用户第二次Ctrl-C（用户往往会按多次Ctrl-C）
  - 没有sigint_handler处理器
  - 用户进程直接被OS杀掉结束，segfault，造成资源泄露

所谓资源泄露包括但不限于：

- 进程及用户自定义的资源，
- rcl层的资源，包括context, publisher, subscriber等
- 通过rmw创建的DDS资源

## ROS2应用要不要自定义SIGINT处理器？

**只要遵循正确的编码规范**，ROS2应用不需要自定义SIGINT处理器。<br />根据软件分层架构的原则，应用开发者不应透过平台直接处理操作系统底层资源，而是应该通过平台的适配器adapter来处理。<br />在下面的例子中通过rclcpp::ok()来判断rcl_context是否失效，如果rcl_context失效，则业务线程全部退出，否则导致未定义后果<br />galactic\src\ros2\demos\intra_process_demo\include\image_pipeline\camera_node.hpp<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1664158045909-6ad2f6ab-2b8e-4152-a5b4-b3d15395d22c.png#averageHue=%23242a32&clientId=ua25f58b3-f825-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=634&id=u248a2085&margin=%5Bobject%20Object%5D&name=image.png&originHeight=634&originWidth=654&originalType=binary&ratio=1&rotation=0&showTitle=false&size=73701&status=done&style=none&taskId=u402f26bc-39e1-4486-af97-9248beb6494&title=&width=654)

# 故障定位

(1)ctrl+C，node是否还存在，很快不存在才是正常的<br />(2)ctrl+C，是否有调用到析构，执行了析构才是正常的<br />(3)spin之后进行deinit、shutdown，不可以进行逻辑业务

# cpp开发规范

析构函数执行的时机，从规范上来看是在离开作用域，且外部强引用个数为0的时候，但由于程序的复杂性，尤其是引用第三方库（opencv, tensorrt等）以及开发人员水平参差不齐的情况下，资源管理尤其复杂且不可控。<br />由于cpp标准的复杂性，析构函数的执行不被开发人员所控制，建议析构函数和构造函数中仅仅做最基本的初始化和析构，不要放业务相关的处理逻辑，如果需要做初始化和去初始化，最好放在自定义的成员函数init()和deinit()/clear()中。<br />（1）        自身模块退出的时候不能再通过ros2去发送msg<br />（2）        自身模块退出的时候需要所有线程需要都退出<br />（3）        自身模块退出的时候需要资源要释放(socket、IO、内存、句柄、。。。)<br />（4）        建议按照初始化的顺序进行反初始化<br />（5）        不要在callback里面做大量计算。 Callback只要做把数据放到队列<br />每个模块自检，改代码，本周重点搞定这个问题

# 参考

[https://www.timiguo.com/archives/209/](https://www.timiguo.com/archives/209/)<br />vector对对象的内存管理<br />[https://cloud.tencent.com/developer/article/1383922](https://cloud.tencent.com/developer/article/1383922)<br />clear（）函数的调用方式是，vector<datatype> temp（50）;//定义了50个datatype大小的空间。temp.clear();<br />作用：将会清空temp中的所有元素，包括temp开辟的空间（size），但是capacity会保留，即不可以以temp[1]这种形式赋初值，只能通过temp.push_back(value)的形式赋初值。

## 附录: 关闭时的确定性

[https://github.com/ros2/rclcpp/issues/1706](https://github.com/ros2/rclcpp/issues/1706)

## 附录：SIG快捷键

[https://superuser.com/questions/343031/sigterm-with-a-keyboard-shortcut](https://superuser.com/questions/343031/sigterm-with-a-keyboard-shortcut)<br />[https://superuser.com/questions/288772/shell-sigkill-keybinding/417991#417991](https://superuser.com/questions/288772/shell-sigkill-keybinding/417991#417991)<br />[https://linuxhandbook.com/sigterm-vs-sigkill/](https://linuxhandbook.com/sigterm-vs-sigkill/)<br />Ctrl+C  SIGINT<br />Ctrl+\   SIGQUIT<br />Ctrl+Z   SIGSUSP
