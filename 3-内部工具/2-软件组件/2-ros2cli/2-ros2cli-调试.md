I found this: [https://wiki.python.org/moin/Debuggin...](https://wiki.python.org/moin/DebuggingWithGdb)

I installed the Python debug package (Ubuntu 20.04):

```text
sudo apt-get install python3-dbg
```

If you try to blindly follow the examples, it doesn't work if you just do:

```text
$ gdb -ex r --args ros2 topic /echo
"/home/$USER/ros2_ws/install/ros2cli/bin/ros2": not in executable format: file format not recognized
```

So you have to run `python3` with the actual entrypoint Python file (which you can see in the error message) as the first argument + the rest. You can also use `which` to find the Python file corresponding to `ros2`:

```text
gdb -ex r --args python3 `which ros2` topic echo /topic
```

This seemed to run `gdb` correctly for me, but with the default RMW implementation there's no segfault, so I can't tell if it actually gives a stack trace, etc. It should, though, according to what I can see in the link above. Let us know if it works!

# 在vscode中调试

# 参考

[How to debug rmw segfaults launched by ros2cli like `ros2 topic echo`? - ROS Answers: Open Source Q&A Forum](https://answers.ros.org/question/384441/how-to-debug-rmw-segfaults-launched-by-ros2cli-like-ros2-topic-echo/?answer=384845#post-id-384845)
