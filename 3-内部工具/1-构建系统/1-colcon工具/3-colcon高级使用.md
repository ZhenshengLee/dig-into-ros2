This page shows some advanced and useful usage of `colcon`. If you need more detailed information, refer to the [colcon documentation](https://colcon.readthedocs.io/).

## Common mistakes[#](https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/advanced-usage-of-colcon/#common-mistakes "Permanent link")

### Do not run from other than the workspace root[#](https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/advanced-usage-of-colcon/#do-not-run-from-other-than-the-workspace-root "Permanent link")

It is important that you always run `colcon build` from the workspace root because `colcon` builds only under the current directory. If you have mistakenly built in a wrong directory, run `rm -rf build/ install/ log/` to clean the generated files.

### Do not unnecessarily overlay workspaces[#](https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/advanced-usage-of-colcon/#do-not-unnecessarily-overlay-workspaces "Permanent link")

`colcon` overlays workspaces if you have sourced the `setup.bash` of other workspaces before building a workspace. You should take care of this especially when you have multiple workspaces.Run `echo $COLCON_PREFIX_PATH` to check whether workspaces are overlaid. If you find some workspaces are unnecessarily overlaid, remove all built files, restart the terminal to clean environment variables, and re-build the workspace.For more details about `workspace overlaying`, refer to the [ROS 2 documentation](https://docs.ros.org/en/rolling/Tutorials/Workspace/Creating-A-Workspace.html#source-the-overlay).

## Cleaning up the build artifacts[#](https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/advanced-usage-of-colcon/#cleaning-up-the-build-artifacts "Permanent link")

`colcon` sometimes causes errors of because of the old cache. To remove the cache and rebuild the workspace, run the following command:

```text
rm -rf build/ install/

```

In case you know what packages to remove:

```text
rm -rf {build,install}/{package_a,package_b}

```

## Selecting packages to build[#](https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/advanced-usage-of-colcon/#selecting-packages-to-build "Permanent link")

To just build specified packages:

```text
colcon build --packages-select <package_name1> <package_name2> ...

```

To build specified packages and their dependencies recursively:

```text
colcon build --packages-up-to <package_name1> <package_name2> ...

```

You can also use these options for `colcon test`.

## Changing the optimization level[#](https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/advanced-usage-of-colcon/#changing-the-optimization-level "Permanent link")

Set `DCMAKE_BUILD_TYPE` to change the optimization level.

Warning

If you specify `DCMAKE_BUILD_TYPE=Debug` or no `DCMAKE_BUILD_TYPE` is given for building the entire Autoware, it may be too slow to use.

```text
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Debug

```

```text
colcon build --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo

```

```text
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release

```

## Changing the default configuration of colcon[#](https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/advanced-usage-of-colcon/#changing-the-default-configuration-of-colcon "Permanent link")

Create `$COLCON_HOME/defaults.yaml` to change the default configuration.

```text
mkdir -p ~/.colcon
cat << EOS > ~/.colcon/defaults.yaml
{
    "build": {
        "symlink-install": true
    }
}

```

For more details, see [here](https://colcon.readthedocs.io/en/released/user/configuration.html#defaults-yaml).

## Generating compile_commands.json[#](https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/advanced-usage-of-colcon/#generating-compile_commandsjson "Permanent link")

[compile_commands.json](https://colcon.readthedocs.io/en/released/user/how-to.html#cmake-packages-generating-compile-commands-json) is used by IDEs/tools to analyze the build dependencies and symbol relationships.

You can generate it with the flag `DCMAKE_EXPORT_COMPILE_COMMANDS=1`:

```text
colcon build --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=1

```

## Seeing compiler commands[#](https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/advanced-usage-of-colcon/#seeing-compiler-commands "Permanent link")

To see the compiler and linker invocations for a package, use `VERBOSE=1` and `--event-handlers console_cohesion+`:

```text
VERBOSE=1 colcon build --packages-up-to <package_name> --event-handlers console_cohesion+

```

For other options, see [here](https://colcon.readthedocs.io/en/released/reference/event-handler-arguments.html).

# 参考

[Advanced usage of colcon - Autoware Documentation](https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/advanced-usage-of-colcon/)
