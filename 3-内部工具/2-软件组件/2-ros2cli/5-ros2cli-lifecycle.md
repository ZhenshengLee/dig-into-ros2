In the case you want to get the current state of the lc_talker node, you would call:

$ ros2 lifecycle get /lc_talker<br />unconfigured [1]

The next step would be to execute a state change:

$ ros2 lifecycle set /lc_talker configure<br />Transitioning successful

In order to see what states are currently available:

$ ros2 lifecycle list lc_talker<br />- configure [1]<br />  Start: unconfigured<br />  Goal: configuring<br />- shutdown [5]<br />  Start: unconfigured<br />  Goal: shuttingdown

In this case we see that currently, the available transitions are configure and shutdown. The complete state machine can be viewed with the following command, which can be helpful for debugging or visualization purposes:

$ ros2 lifecycle list lc_talker -a<br />- configure [1]<br />  Start: unconfigured<br />  Goal: configuring<br />- transition_success [10]<br />  Start: configuring<br />  Goal: inactive<br />- transition_failure [11]<br />  Start: configuring<br />  Goal: unconfigured<br />- transition_error [12]<br />  Start: configuring<br />  Goal: errorprocessing

[...]

- transition_error [62]<br />  Start: errorprocessing<br />  Goal: finalized

All of the above commands are nothing more than calling the lifecycle node\'s services. With that being said, we can also call these services directly with the ros2 command line interface:

$ ros2 service call /lc_talker/get_state lifecycle_msgs/GetState<br />requester: making request: lifecycle_msgs.srv.GetState_Request()

response:<br />lifecycle_msgs.srv.GetState_Response(current_state=lifecycle_msgs.msg.State(id=1, label='unconfigured'))

In order to trigger a transition, we call the change_state service

$ ros2 service call /lc_talker/change_state lifecycle_msgs/ChangeState "{transition: {id: 2}}"<br />requester: making request: lifecycle_msgs.srv.ChangeState_Request(transition=lifecycle_msgs.msg.Transition(id=2, label=''))

response:<br />lifecycle_msgs.srv.ChangeState_Response(success=True)

It is slightly less convenient, because you have to know the IDs which correspond to each transition. You can find them though in the lifecycle_msgs package.

$ ros2 interface show lifecycle_msgs/msg/Transition

# 参考

[https://index.ros.org/p/lifecycle/github-ros2-demos/](https://index.ros.org/p/lifecycle/github-ros2-demos/)
