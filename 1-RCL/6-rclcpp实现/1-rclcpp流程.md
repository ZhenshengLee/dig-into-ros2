# 总体架构

rclcpp layer大概分两个组件，一个是executor，一个是node(component)，机器人概念全部封装在node中，包括pubsub, service, action, timer等。还有一个组件一般不关注，context，默认一个进程一个context

![](https://tcs.teambition.net/storage/312geb9a2e2136951a8a15202c9041db58e2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdlYjlhMmUyMTM2OTUxYThhMTUyMDJjOTA0MWRiNThlMiJ9.T4klM80AolfWu4BP_MFTyrNYscrKeQ6IZA-vY5Sm8co&download=image.png "")

# 进程初始化

In the call to `rclcpp::init()`, a process-specific `rclcpp::Context` object is fetched and CLI arguments are parsed. Much of the work is actually done by `rcl` through a call to `rcl_init()`. This call processes the `rcl_context_t` handle, which is wrapped by the `Context` object. Also, inside this call, `rcl` calls `rmw_init()` to process the `rmw` context (`rmw_context_t`) as well. This `rmw` handle is itself part of the `rcl_context_t` handle.

This has to be done once per process, and usually at the very beginning. The components that are then instanciated share this context.

![](https://tcs.teambition.net/storage/312ha0e675f2a7a69c4a580028bf09bff386?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhhMGU2NzVmMmE3YTY5YzRhNTgwMDI4YmYwOWJmZjM4NiJ9.DDIRR2Ex2NrHgitpHg_3A23sAgnBkGxl4Ouu_GruPhE&download=image.png "")

rclcpp初始化了context环境，加载了参数等数据结构

rmw初始化了dds实例，一个context初始化一个dds paticipant，创建若干个dds线程，发布订阅线程未创建

具体取决于rmw实现，比如cyclonedds只是初始化了一个context，加载了配置等数据结构

使用lazy-init的方法，在第一次node_init时才创建participant，当然这个participant是context内部唯一的

![](https://tcs.teambition.net/storage/312hd260be3e8257c9cbd7de10821ea3911c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhkMjYwYmUzZTgyNTdjOWNiZDdkZTEwODIxZWEzOTExYyJ9.gP6-QbgFWccZifCcJUtHOOSD8iBR_HGIBu8CTohXTI8&download=image.png "")

# node(component)

In ROS 2, a process can contain multiple nodes. These are sometimes referred to as "components."

These components are instanciated by the containing process. They are usually classes that extend `rclcpp::Node`, so that the node initialization work is done by the parent constructor.This parent constructor will allocate its own `rcl_node_t` handle and call `rcl_node_init()`, which will validate the node name/namespace. `rcl` will also call `rmw_create_node()` to get the node's `rmw` handle (`rmw_node_t`). This will be used later by publishers and subscriptions.

![](https://tcs.teambition.net/storage/312h4e47158033d63f1d08ace111ce48dc0b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg0ZTQ3MTU4MDMzZDYzZjFkMDhhY2UxMTFjZTQ4ZGMwYiJ9.bUwCs3HwoEpUy9_Zg9BPKHi5UhP1zlOkikgnwDo4_do&download=image.png "")

create_node，创建一个node，然后将其加入graph.cache

老旧的rmw实现，不会维护一个graph.cache，一般自行维护

![](https://tcs.teambition.net/storage/312ha0f107511be29ac024761022a5c49bb7?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhhMGYxMDc1MTFiZTI5YWMwMjQ3NjEwMjJhNWM0OWJiNyJ9.EIOplMoeVgix-t8LnE0HdGqTmYdJoo76kbsEUtA_VSU&download=image.png "")

![](https://tcs.teambition.net/storage/312h375932d14b0850451f3a8b10fe00747f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgzNzU5MzJkMTRiMDg1MDQ1MWYzYThiMTBmZTAwNzQ3ZiJ9.Y9EBx27h3n7XXgM_6kLR9k9cSTQN7iIVrl9aoZnzFrE&download=image.png "")

但是graph是ros_dds_common提供的方便工具，推荐使用

ddsNode对应哪些线程，取决于具体dds实现

# Publisher

The component calls `create_publisher()`, a `rclcpp::Node` method for convenience. That ends up creating an `rclcpp::Publisher` object which extends `rclcpp::PublisherBase`. The latter allocates an `rcl_publisher_t` handle, fetches the corresponding `rcl_node_t` handle, and calls `rcl_publisher_init()` in its constructor. `rcl` does topic name expansion/remapping/validation. It creates an `rmw_publisher_t` handle by calling `rmw_create_publisher()` of the given `rmw` implementation and associates with the node's `rmw_node_t` handle and the publisher's `rcl_publisher_t` handle.If intra-process publishing/subscription is enabled, it will be set up after creating the publisher object, through a call to `PublisherBase::setup_intra_process()`, which calls `rcl_publisher_init()`.

![](https://tcs.teambition.net/storage/312h63858969108fb38ceab07613e0a37592?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg2Mzg1ODk2OTEwOGZiMzhjZWFiMDc2MTNlMGEzNzU5MiJ9.UjLO7qA1tKpCquxF6bxfDdxKrtQFT4qTWPcOfWi76GY&download=image.png "")

这一步几乎可以肯定是创建新的线程的，而且会创建publisher和data writer

# Subscription

Subscription creation is done in a very similar manner.

The componenent calls `create_publisher()`, which ends up creating an `rclcpp::Subscription` object which extends `rclcpp::SubscriptionBase`. The latter allocates an `rcl_subscription_t` handle, fetches its `rcl_node_t` handle, and calls `rcl_subscription_init()` in its constructor. `rcl` does topic name expansion/remapping/validation. It creates an `rmw_subscription_t` handle by calling `rmw_create_subscription()` of the given `rmw` implementation and associates it with the node's `rmw_node_t` handle and the subscription's `rcl_subscription_t` handle.If intra-process publishing/subscription is enabled, it will be set up after creating the subscription object, through a call to `Subscription::setup_intra_process()`, which calls `rcl_subscription_init()`. This is very similar to a normal (inter-process) subscription, but it sets some flags for later.

![](https://tcs.teambition.net/storage/312h8122b0156bef2c9849d2c3a3e340a008?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg4MTIyYjAxNTZiZWYyYzk4NDlkMmMzYTNlMzQwYTAwOCJ9.ua3pMnxjQsIGOxoxRqMy3hqaKAMP2MNiVdN63rnNGqw&download=image.png "")

也创建线程

# Executor

An `rclcpp::executor::Executor` object is created for a given process. It can be a `SingleThreadedExecutor` or a `MultiThreadedExecutor`.Components are instanciated, usually as a `shared_ptr` through `std::make_shared<Component>()`, then added to the executor with `Executor::add_node()`.After all the components have been added, `Executor::spin()` is called. `SingleThreadedExecutor::spin()` simply loops forever until the process' context isn't valid anymore. It fetches the next `rclcpp::AnyExecutable` (e.g. subscription, timer, service, client), and calls `Executor::execute_any_executable()` with it. This then calls the relevant `execute*()` method (e.g. `execute_timer()`, `execute_subscription()`, `execute_intra_process_subscription()`, `execute_service()`, `execute_client()`).

![](https://tcs.teambition.net/storage/312h3642745bbd03e8549bf34ea794e7a6b7?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgzNjQyNzQ1YmJkMDNlODU0OWJmMzRlYTc5NGU3YTZiNyJ9.VAEjchK_uRrdequ0aXoVDRhQopWPtDVvyil-ULhGpJk&download=image.png "")

这就是真正的rclcpp线程，可以是单线程，可以是多线程

你会发现不与rmw打交道，因为是彻底的rclcpp线程！通过wait_set机制与rmw联系，这是proactor模式

# subscription callback

Subscriptions are handled in the `rclcpp` layer. Callbacks are wrapped by an `rclcpp::AnySubscriptionCallback` object, which is registered when creating the `rclcpp::Subscription` object.In `execute_*subscription()`, the `Executor` asks the `Subscription` to allocate a message though `Subscription::create_message()`. It then calls `rcl_take*()`, which calls `rmw_take_with_info()`. If that is successful, the `Executor` then passes that on to the subscription through `rclcpp::SubscriptionBase::handle_message()`. This checks if it's the right type of subscription (i.e. inter vs. intra process), then it calls `dispatch()` on the `rclcpp::AnySubscriptionCallback` object with the message (cast to the actual type). This calls the actual `std::function` with the right signature.Finally, it returns the message object through `Subscription::return_message()`.

![](https://tcs.teambition.net/storage/312h8d3f9e21ef4f69c63216613b460fd31b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg4ZDNmOWUyMWVmNGY2OWM2MzIxNjYxM2I0NjBmZDMxYiJ9.DhCYiLEpdGYp6lTa_54kek0th9_vGyBg9k3KKh_YY8I&download=image.png "")

请注意：executor是执行回调的时候才取数据，这依然是lazy的思想

之前wait的只是一个信号

# message publishing

To publish a message, an object is first allocated and then populated by the `Component` (or equivalent). Then, the message is sent to the `Publisher` through `publish()`. This then passes that on to `rcl`, which itself passes it to `rmw`.

TODO add inter- vs. intra-process execution flow TODO talk about IntraProcessManager stuff?

![](https://tcs.teambition.net/storage/312h93913a992d5adb16222cd9c997832a9b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg5MzkxM2E5OTJkNWFkYjE2MjIyY2Q5Yzk5NzgzMmE5YiJ9.lRf6sOl88msuwFCQfaPoSrQupB7r4FX_SGYcAyEDN_U&download=image.png "")

# Service和Client

和message大体一致

# Timer Creation

Timer creation is similar to subscription creation. The `Component` calls `create_service()` which ends up creating a `rclcpp::WallTimer`. In its constructor, it creates a `rclcpp::Clock` object, which (for a `WallTimer`) is simply a nanosecond clock. It then allocates a `rcl_timer_t` handle, then calls `rcl_timer_init()`. This processes the handle and validates the period.Note that `rcl_timer_init()` can take a callback as a parameter, but right now that feature is not used anywhere (`nullptr` is given), and callbacks are instead handled in the `rclcpp` layer.

![](https://tcs.teambition.net/storage/312hbe7e16e4409873392435320e37550300?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhiZTdlMTZlNDQwOTg3MzM5MjQzNTMyMGUzNzU1MDMwMCJ9.Z9poHwQaNpCeNEMp2ZaFVq7jEhMXodIrMFRM3iXWG5w&download=image.png "")

# Timer Callback

Timer callbacks are similar to susbcription callbacks. In `execute_timer()`, the `Executor` calls `execute_callback()` on the `WallTimer`. The timer then calls `rcl_timer_call()` with its `rcl_timer_t` handle and checks if the callback should be called.If it that is the case, then the timer will call the actual `std::function`. Depending on the `std::function` that was given when creating the timer, it will either call the callback without any parameters or it will pass a reference of itself.

![](https://tcs.teambition.net/storage/312h7a75765a6ae2b4ba032eb807860f1daa?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5MTQ4MCwiaWF0IjoxNjcxMTg2NjgwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg3YTc1NzY1YTZhZTJiNGJhMDMyZWI4MDc4NjBmMWRhYSJ9.ghG56C8UT8Er8JATJW0wMVmKu14XsNZaL11P74Q-RuU&download=image.png "")

# 参考

[ros2_tracing/design_ros_2.md at outdoor/galactic · ZhenshengLee/ros2_tracing](https://github.com/ZhenshengLee/ros2_tracing/blob/outdoor/galactic/doc/design_ros_2.md)
