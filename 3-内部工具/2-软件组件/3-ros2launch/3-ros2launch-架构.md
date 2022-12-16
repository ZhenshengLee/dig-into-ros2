# 系统描述

system description是对actions和reactions的声明式描述，表达用户启动系统的需求<br />system description的目的是尽可能捕获用户的启动诉求，这样launch description可以可视化，可以静态分析。<br />用一个WYSIWYG 界面是一个很重要的使用场景。<br />本节描述一个实现无关的系统描述，来描述calling convention, event handling，以及其他specific behavior<br />进一步描述这种描述如何应用到具体的描述语言上如python，xml

## 启动描述

The system is described in parts which we’ll refer to here as “Launch Descriptions”. Launch descriptions are made of up of an ordered list of actions and groups of actions. It may also contain substitutions throughout the description, which are used to add some flexibility and configuration to the descriptions in a structured way.

### Actions

Actions may be one of several things, and each action has a type (associated with the action’s function) and may also contain arbitrary configurations. **Actions represent an intention to do something, but a launch description is parsed first, then actions are taken in order of definition later.** This allows actions to be interpreted and then statically introspected without actually executing any of them unless desired.<br />action是一种描述，在需要的时候执行<br />声明action在前，在launch description中包含actions，才会真正执行<br />Basic actions include being able to:

- include another launch description
- modify the launch system configurations at the current scope
- execute a process
- register/unregister an event handler
- emit an event
- additional actions defined by extensions to the launch system

Actions may also yield more actions and groups rather than perform an actual task. For example, an action to “run a node” may end up resulting in “executing two process” or in “executing a process and registering an event handler”. These kind of actions could be thought of a launch description generators or macros, since they effectively generate the same contents as a launch description, but look like an action to the user.<br />action可能会引出更多的action，比如node执行可能会引出执行进程等等。<br />This allows for more complex actions which might include, but not be limited to:

- include a launch description from a file with a certain markup type
- set an environment variable
- run a single-node process
- run a multi-node process
- run a node container
- run a node proxy to load into a node container
- run a process on a remote computer
- declare launch description arguments
  - exposed as either:
    - command line arguments for top-level launch descriptions
    - or additional arguments to the “include another launch description” action
  - stored in “launch system configuration”
- various OS actions, e.g. touch a file, read a file, write to a file, etc…

### including

One of the simplest actions is to include another launch description. This launch description is processed in its entirety, including parsing of any launch descriptions it includes recursively. Therefore processing of launch descriptions is in order, and depth first.<br />Included launch descriptions inherit all configurations of the current launch description, and any changes to the launch system configurations made in the included launch description will affect actions after the include action.<br />However, it should also be possible to control which configurations are inherited by an included launch description and also to “scope” an included launch description so that it cannot affect the configuration above it.<br />In all cases, the desired behavior may be achieved though selective use of optionally scoped group actions.

### Launch System Configuration

The “modify the launch system configurations at the current scope” action mentioned above is able to mutate a local scope of configurations which can affect other actions which come after the modification. Actions may use this local state to uniformly apply certain settings to themselves.<br />For example, the environment variables which are set when running an operating system process would be taken from the launch system configuration, and therefore can be modified with an action. Then any “execute a process” actions which come after it will be affected by the configuration change.<br />Changes to the local state by included launch descriptions persist, as they should be thought of as truly included in the same file, as if you had copied the contents of the included launch description in place of the include action. To avoid this, a group without a namespace could be used to produce a push-pop effect on the launch configurations.

### Execute a Process

Another basic action would be to execute a subprocess, with arguments and emitted events, as described in the calling conventions section under “operating system process”.<br />This action will take a few required arguments, a few optional requirements, and also take settings from the launch system configurations if they’re not explicitly given. The signature of this action should be similar to the API of Python’s subprocess.run function[7](https://design.ros2.org/articles/roslaunch.html#fn:subprocess_run). Basically taking things like the executable file, arguments, working directory, environment, etc. as input and reporting the return code, stdout and stderr, and any errors as emitted events.<br />Also, every executed process will automatically setup a few event handlers, so that the user can emit events to ask the launch system to terminate the process (following the signal escalation described in previous sections), signal the process explicitly, or write to the stdin of the process. More sophisticated calling conventions which are based on the “operating system process” may include other default event handlers.

### Event Handlers

The launch description can also contain event handlers. An event handler is essentially a function which takes an event as input and returns a launch description to be included at the location of the event handler registration. The event handler will be executed asynchronously when the associated event is emitted.<br />There are two actions associated with event handlers, registering one and unregistering one. How event types and event handlers are represented and tracked depends on the implementation of the launch system. However, as an example, a “launch file” written in Python might represent event’s as classes which inherit from a base class. If instead the “launch file” is written in XML, event types might be expressed as a string, with launch system events using a “well-known name” and with user definable events being represented with unique strings as well. Similarly, the Python based “launch file” might use instances of objects to represent registered event handlers, therefore you might need that object to perform the unregister action. And in the XML based “launch file” the user might be required to give a unique name to all registered event handlers, so that they can unregistered with the same name later.<br />When an event handler finishes, it is able to return a launch description which is implicitly given to the include action at the location of the event handler’s registration. This allows an event handler to cause any action upon completion, e.g. include another launch description, unregister an event handler, emit another event, run a process, start the termination of a process by emitting an event, etc…<br />The lowest level of event handlers is the function which takes an event and returns a launch description. For example, a user defined event handler might look like this in Python:<br />最底层的event handler是输入一个事件，返回一个launch description

```python
# This is a made up example of an API, consider it pseudo code...

launch_description = LaunchDescription(...)
# ...

def my_process_exit_logger_callback(event: ProcessExitedEvent) -> LaunchDescription:
    print(f"process with pid '{event.pid}' exited with return code '{event.return_code}'")

launch_description.register_event_handler(
    ProcessExitedEvent, my_process_exit_logger_callback, name='my_process_exit_logger')
```

### Emitting Events

Another basic action that the launch system should support is the ability to emit events and if necessary declare new kinds of events before emitting them.<br />发射事件<br />This feature could be used by users to filter a launch system event and then dispatch it to other user defined event handlers, or to create new or modified events based on existing events.<br />How events are defined is up to the implementation, but it should be possible to model the events so they can be emitted and then handled by registered event handlers.

# 架构

launch is designed to provide core features like describing actions (e.g. executing a process or including another launch description), generating events, introspecting launch descriptions, and executing launch descriptions. At the same time, it provides extension points so that the set of things that these core features can operate on, or integrate with, can be expanded with additional packages.

## Launch Entities and Launch Descriptions

The main object in launch is the [:class:`launch.LaunchDescriptionEntity`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id1), from which other entities that are "launched" inherit. This class, or more specifically classes derived from this class, are responsible for capturing the system architect's (a.k.a. the user's) intent for how the system should be launched, as well as how launch itself should react to asynchronous events in the system during launch. A launch description entity has its [:meth:`launch.LaunchDescriptionEntity.visit`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id3) method called during "launching", and has any of the "describe" methods called during "introspection". It may also provide a [:class:`asyncio.Future`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id5) with the [:meth:`launch.LaunchDescriptionEntity.get_asyncio_future`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id7) method, if it has on-going asynchronous activity after returning from visit.<br />When visited, entities may yield additional entities to be visited, and this pattern is used from the "root" of the launch, **where a special entity called**[**:class:`launch.LaunchDescription`**](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id9)**is provided to start the launch process.**<br />galactic\src\ros2\launch\launch\launch\launch_description_entity.py<br />galactic\src\ros2\launch\launch\launch\launch_description.py

```python
class LaunchDescription(LaunchDescriptionEntity):
    """
    Description of a launch-able system.
```

The [:class:`launch.LaunchDescription`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id11) class encapsulates the intent of the user as a list of discrete [:class:`launch.Action`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id13)'s, which are also derived from [:class:`launch.LaunchDescriptionEntity`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id15). As "launch description entities" themselves, these "actions" can either be introspected for analysis without performing the side effects, or the actions can be executed, usually in response to an event in the launch system.<br />Additionally, launch descriptions, and the actions that they contain, can have references to [:class:`launch.Substitution`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id17)'s within them. These substitutions are things that can be evaluated during launch and can be used to do various things like: get a launch configuration, get an environment variable, or evaluate arbitrary Python expressions.<br />Launch descriptions, and the actions contained therein, can either be introspected directly or launched by a [:class:`launch.LaunchService`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id19). A launch service is a long running activity that handles the event loop and dispatches actions.

## Actions

The aforementioned actions allow the user to express various intentions, and the set of available actions to the user can also be extended by other packages, allowing for domain specific actions.<br />Actions can have direct side effects (e.g. run a process or set a configuration variable) and as well they can yield additional actions. The latter can be used to create "syntactic sugar" actions which simply yield more verbose actions.<br />Actions may also have arguments, which can affect the behavior of the actions. These arguments are where [:class:`launch.Substitution`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id21)'s can be used to provide more flexibility when describing reusable launch descriptions.

### Basic Actions

launch provides the foundational actions on which other more sophisticated actions may be built. This is a non-exhaustive list of actions that launch may provide:

- [:class:`launch.actions.IncludeLaunchDescription`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id23)
  - This action will include another launch description as if it had been copy-pasted to the location of the include action.
- [:class:`launch.actions.SetLaunchConfiguration`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id25)
  - This action will set a [:class:`launch.LaunchConfiguration`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id27) to a specified value, creating it if it doesn't already exist.
  - These launch configurations can be accessed by any action via a substitution, but are scoped by default.
- [:class:`launch.actions.DeclareLaunchArgument`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id29)
  - This action will declare a launch description argument, which can have a name, default value, and documentation.
  - The argument will be exposed via a command line option for a root launch description, or as action configurations to the include launch description action for the included launch description.
- [:class:`launch.actions.SetEnvironmentVariable`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id31)
  - This action will set an environment variable by name.
- [:class:`launch.actions.AppendEnvironmentVariable`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id33)
  - This action will set an environment variable by name if it does not exist, otherwise it appends to the existing value using a platform-specific separator.
  - There is also an option to prepend instead of appending and to provide a custom separator.
- [:class:`launch.actions.GroupAction`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id35)
  - This action will yield other actions, but can be associated with conditionals (allowing you to use the conditional on the group action rather than on each sub-action individually) and can optionally scope the launch configurations.
- [:class:`launch.actions.TimerAction`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id37)
  - This action will yield other actions after a period of time has passed without being canceled.
- [:class:`launch.actions.ExecuteProcess`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id39)
  - This action will execute a process given its path and arguments, and optionally other things like working directory or environment variables.
- [:class:`launch.actions.RegisterEventHandler`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id41)
  - This action will register an [:class:`launch.EventHandler`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id43) class, which takes a user defined lambda to handle some event.
  - It could be any event, a subset of events, or one specific event.
- [:class:`launch.actions.UnregisterEventHandler`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id45)
  - This action will remove a previously registered event.
- [:class:`launch.actions.EmitEvent`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id47)
  - This action will emit an [:class:`launch.Event`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id49) based class, causing all registered event handlers that match it to be called.
- [:class:`launch.actions.LogInfo`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id51):
  - This action will log a user defined message to the logger, other variants (e.g. LogWarn) could also exist.
- [:class:`launch.actions.RaiseError`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id53)
  - This action will stop execution of the launch system and provide a user defined error message.

More actions can always be defined via extension, and there may even be additional actions defined by launch itself, but they are more situational and would likely be built on top of the above actions anyways.

### Base Action

All actions need to inherit from the [:class:`launch.Action`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id55) base class, so that some common interface is available to the launch system when interacting with actions defined by external packages. Since the base action class is a first class element in a launch description it also inherits from [:class:`launch.LaunchDescriptionEntity`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id57), which is the polymorphic type used when iterating over the elements in a launch description.<br />Also, the base action has a few features common to all actions, such as some introspection utilities, and the ability to be associated with a single [:class:`launch.Condition`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id59), like the [:class:`launch.IfCondition`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id61) class or the [:class:`launch.UnlessCondition`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id63) class.<br />The action configurations are supplied when the user uses an action and can be used to pass "arguments" to the action in order to influence its behavior, e.g. this is how you would pass the path to the executable in the execute process action.<br />If an action is associated with a condition, that condition is evaluated to determine if the action is executed or not. Even if the associated action evaluates to false the action will be available for introspection.

## Substitutions

A substitution is something that cannot, or should not, be evaluated until it's time to execute the launch description that they are used in. There are many possible variations of a substitution, but here are some of the core ones implemented by launch (all of which inherit from [:class:`launch.Substitution`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id65)):

- [:class:`launch.substitutions.Text`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id67)
  - This substitution simply returns the given string when evaluated.
  - It is usually used to wrap literals in the launch description so they can be concatenated with other substitutions.
- [:class:`launch.substitutions.PythonExpression`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id69)
  - This substitution will evaluate a python expression and get the result as a string.
- [:class:`launch.substitutions.LaunchConfiguration`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id71)
  - This substitution gets a launch configuration value, as a string, by name.
- [:class:`launch.substitutions.LaunchDescriptionArgument`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id73)
  - This substitution gets the value of a launch description argument, as a string, by name.
- [:class:`launch.substitutions.LocalSubstitution`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id75)
  - This substitution gets a "local" variable out of the context. This is a mechanism that allows a "parent" action to pass information to sub actions.
  - As an example, consider this pseudo code example OnShutdown(actions=LogInfo(msg=["shutdown due to: ", LocalSubstitution(expression='event.reason')])), which assumes that OnShutdown will put the shutdown event in the locals before LogInfo is visited.
- [:class:`launch.substitutions.EnvironmentVariable`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id77)
  - This substitution gets an environment variable value, as a string, by name.
- [:class:`launch.substitutions.FindExecutable`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id79)
  - This substitution locates the full path to an executable on the PATH if it exists.

The base substitution class provides some common introspection interfaces (which the specific derived substitutions may influence).

## The Launch Service

The launch service is responsible for processing emitted events, dispatching them to event handlers, and executing actions as needed. The launch service offers three main services:

- include a launch description
  - can be called from any thread
- run event loop
- shutdown
  - cancels any running actions and event handlers
  - then breaks the event loop if running
  - can be called from any thread

A typical use case would be:

- create a launch description (programmatically or based on a markup file)
- create a launch service
- include the launch description in the launch service
- add a signal handler for SIGINT that calls shutdown on the launch service
- run the event loop on the launch service

Additionally you could host some SOA (like REST, SOAP, ROS Services, etc...) server in another thread, which would provide a variety of services, all of which would end up including a launch description in the launch service asynchronously or calling shutdown on the launch service asynchronously. Remember that a launch description can contain actions to register event handlers, emit events, run processes, etc. So being able to include arbitrary launch descriptions asynchronously is the only feature you require to do most things dynamically while the launch service is running.

## Event Handlers

Event handlers are represented with the [:class:`launch.EventHandler`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id81) base class. Event handlers define two main methods, the [:meth:`launch.EventHandler.matches`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id83) method and the [:meth:`launch.EventHandler.handle`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id85) method. The matches method gets the event as input and must return True if the event handler matches that event, or False otherwise. The handle method gets the event and launch context as input, and can optionally (in addition to any side effects) return a list of [:class:`launch.LaunchDescriptionEntity`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id87) objects to be visited by the launch service.<br />Event handlers do not inherit from [:class:`launch.LaunchDescriptionEntity`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id89), but can similarly be "visited" for each event processed by the launch service, seeing if matches returns True and if so following up with a call to handle, then visiting each of the actions returned by handle, depth-first.

## Extension Points

In order to allow customization of how launch is used in specific domains, extension of the core categories of features is provided. External Python packages, through extension points, may add:

- new actions
  - must directly or indirectly inherit from [:class:`launch.Action`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id91)
- new events
  - must directly or indirectly inherit from [:class:`launch.Event`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id93)
- new substitutions
  - must directly or indirectly inherit from [:class:`launch.Substitution`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id95)
- kinds of entities in the launch description
  - must directly or indirectly inherit from [:class:`launch.LaunchDescriptionEntity`](https://github.com/ros2/launch/blob/master/launch/doc/source/architecture.rst#id97)

In the future, more traditional extensions (like with setuptools' entry_point feature) may be available via the launch service, e.g. the ability to include some extra entities and event handlers before the launch description is included.

# 包结构

## launch_ros

包含ros2cli的launch部分，以及launch和node紧密耦合的部分，属于用户包

### ros2launch

这个是ros2 launch的cli，使用python做封装

### launch_ros

ROS specific extensions to the launch tool.

### launch_testing_ros

测试launch文件的测试套件

### test_launch_ros

测试launch_ros包的单元测试

## launch

包含launch文件解析处理的前端包，以及具体实施launch description的后端包，launch与process紧密耦合的部分

### launch

包含launch后端python脚本

### launch_testing

launch的单元测试

### launch_testing_ament_cmake

ament集成

### launch_xml

launch的xml实现

### test_launch_testing

测试launch脚本的测试套件
