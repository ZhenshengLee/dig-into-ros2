ros2最初使用ament_tool，后来迁移到colcon，所完成的功能和ament_tools差不多，可以参考ament_tools的概念来理解colcon的设计

# 参考

[A universal build tool](http://design.ros2.org/articles/build_tool.html)

[The build system "ament_cmake" and the meta build tool "ament_tools"](https://design.ros2.org/articles/ament.html)

When the first draft of this article was written the conclusion was to not to spend any resources towards a universal build tool. Meanwhile the author of this article went ahead and developed [colcon](https://github.com/colcon/) as a personal project. Therefore its feature set is closely aligned with the following requirements.

# 需求

## Naming

The existing build tools in ROS are all named by the build system they are supporting. In general it should be possible for a build tool to support multiple different build systems. Therefore a name for a build tool being derived from a single build system might mislead the users that the tool only works for that specific build system. To avoid confusion of the user the build tool should have a different unrelated name to avoid implying an undesired correlation.

## Requirements

The unified build tool should provide a superset of the functionality provided by the existing tools. In the following a few use cases are described as well as desired software criteria.

Other use cases which are not explicitly covered but are already supported by the existing tools (e.g. cross-compilation, `DESTDIR` support, building CMake packages without a manifest) should continue to work with the unified build tool.

## Use Cases

The following uses cases should be satisfied by the unified build tool.

Build ROS 1 workspaces

The tool needs to be able to build ROS 1 workspaces which can already be built using `catkin_make_isolated` / `catkin_tools`. It is up to the implementation to decide if it only supports the standard CMake workflow or also the *custom devel space concept* of `catkin`.

In ROS 2 the concept of the *devel space* has intentionally been removed. In the future it might be feasible to provide the concept of *symlinked installs* in ROS 1 to provide a similar benefit without the downsides.

Build ROS 2 workspaces

The tool needs to be able to build ROS 2 workspaces which can already be built using `ament_tools`.

Build Gazebo including dependencies

After cloning the repositories containing Gazebo and all its dependencies (excluding system packages) the tool needs to be able to build the set of packages. Meta information not inferable from the sources can be provided externally without adding or modifying any files in the workspace. After the build a single file can be sourced / invoked to setup the environment to use Gazebo (e.g. `GAZEBO_MODEL_PATH`).

## Development Environment Setup

Invoking a build system for a package implies also setting up environment variables before the process, e.g. the `CMAKE_PREFIX_PATH`. It should be possible for developers to manually invoke the build system for one package. The environment variable might be partially different from the environment variables necessary to use a package after it has been built. To make that convenient the tool should provide an easy-to-use mechanism to setup the development environment necessary to manually invoke the build system.

## Beyond Building

Building packages is only one task the build tool can perform on a set of packages. Additional tasks like e.g. running tests should also be covered by the build tool. The build tool must provide these abstract tasks and then map them to the steps necessary for each supported build system.

## Software Criteria

The tool aims to support a variety of build systems, use cases, and platforms. The above mentioned ones are mainly driven by the needs in the ROS ecosystem but the tool should also be usable outside the ROS ecosystem (e.g. for Gazebo). Therefore it should be designed in a way which enables extending its functionality.

Assuming that the tool will be implemented in Python (since that is the case for all existing ROS build tools) the entry point mechanism provides a convenient way to make the software extensible. Extensions don’t even have to be integrated into the Python package containing the core logic of the build tool but can easily be provided by additional Python packages. This approach will not only foster a modular design and promote clear interfaces but enable external contributions without requiring them to be integrated in a single monolithic package.

Several well known software principles apply:

- Separation of concerns

- Single Responsibility principle

- Principle of Least Knowledge

- Don’t repeat yourself

- Keep it stupid simple

- “Not paying for what you don’t use”

## Extension Points

The following items are possible extension points to provide custom functionality:

- contribute `verbs` to the command line tool (e.g. `build`, `test`)

- contribute command line options for specific features (e.g. `build`, `test`)

- discovery of packages (e.g. recursively crawling a workspace)

- identification of packages and their meta information (e.g. from a `package.xml` file)

- process a package (e.g. build a CMake package, test a Python package)

- execution model (e.g. sequential processing, parallel processing)

- output handling (e.g. console output, logfiles, status messages, notifications)

- setup the environment (e.g. `sh`, `bash`, `bat`)

- completion (e.g. `bash`, `Powershell`)

# 可选方案

Possible Approaches

When the first draft of this article was written neither of the existing build tools supported the superset of features described in this article. There were multiple different paths possible to reach the goal of a universal build tool which fall into two categories:

- One approach is to incrementally evolve one of the existing tools to satisfy the described goals.

- Another approach would be to start “from scratch”.

Since then the new project `colcon` has been developed which covers most of the enumerated requirements and represents the second category.

### Evolve catkin_make, catkin_make_isolated, or ament_tools

Since neither of these three build tools has the feature richness of `catkin_tools` it is considered strictly less useful to starting building upon one of these build tools. Therefore neither of these are being considered as a foundation for a universal build tool.

### Evolve catkin_tools

Since `catkin_tools` is in many aspects the most complete ROS build tool it should be the one being evolved. While `ament_tools` has a few features `catkin_tools` currently lacks (e.g. plain CMake support without a manifest, Windows support) the feature richness of `catkin_tools` makes it a better starting point.

### Start “from scratch” / colcon

Since the first draft of this article the `colcon` project has been developed with the goals and requirements of a universal build tool in mind. In its current form it is already able to build ROS 1 workspaces, ROS 2 workspaces, as well as Gazebo including its ignition dependencies. It uses Python 3.5+ and targets all platforms supported by ROS: Linux, macOS, and Windows.Since it hasn’t been used by many people yet more advanced features like cross compilation, `DESTDIR`, etc. hasn’t been tested (and will therefore likely not work yet).

# 关键决策

Decision process

## 起步

For the decision process only the following two options are being considering based on the rationale described above:

- option **A)** Use `catkin_tools` as a starting point

- option **B)** Use `colcon` as a starting point

If this topic would have been addressed earlier some of the duplicate effort could have likely been avoided. When the work towards a universal build tool was suspended over a year ago it was a conscious decision based on available resources. Nevertheless moving forward with a decision now will at least avoid further uncertainty and effort duplication.

Both of the considered options have unique and valuable features and there are good arguments to build our future development on either of the two tools. Since both are written in Python either of the two tools could be “transformed” to cover the pros of the other one. So the two important criteria for the decision are:

- the effort it takes to do (in the short term as well as in the long term) and

- the difference of the resulting code base after the “transformation” is completed.

## Immediate goals

A ROS 2 developer currently builds a steadily growing workspace with ROS 2 packages. The same is happening in the monolithic Jenkins jobs on [ci.ros2.org](https://ci.ros2.org/) (with the advantage to test changes across repositories easily). Therefore features to easily filter the packages which need to be build are eagerly awaited to improve the development process.

For the last ROS 2 release *Ardent* the buildfarm [build.ros.org](http://build.ros2.org/) only provides jobs to generate Debian packages. Neither *devel* jobs or *pull request* jobs are available nor is it supported to build a local *prerelease*. For the coming ROS 2 release *Bouncy* these job types should be available to support maintainers.

In ROS 2 *Bouncy* the universal build tool will be the recommended option.

Necessary work

For either option **A)** or **B)** the follow items would need to be addressed:

- The jobs and scripts on *ci.ros2.org* need to be updated to invoke the universal build tool instead of `ament_tools`.

- The `ros_buildfarm` package needs to be updated to invoke the universal build tool instead of `catkin_make_isolated`. The ROS 2 buildfarm would use this modification for the upcoming ROS 2 *Bouncy* release. The ROS 1 buildfarm could use the same modification in the future.

For option **A)** the follow items would need to be addressed:

- Support for setup files generated by `ament_cmake`.

- Support additional packages types: plain Python packages, CMake packages without a manifest.

- Support for Windows using `.bat` files.

- Support for the package manifest format version 3.

For option **B)** the follow items would need to be addressed:

- Address user feedback when the tool is being used by a broader audience.

## Future

The long term goal is that the universal build tool will be used in ROS 1, in ROS 2 as well as other non-ROS projects. There is currently no time line when the tool will be used on the ROS 1 build or be recommended to ROS 1 users. This solely depends on the resources available for ROS 1.

Beside that for both options there is follow up work beyond the immediate goals. The following enumerates a few of them but is by no means exhaustive:

For option **A)** the follow items should be considered:

- Support for Python packages using a `setup.cfg` file.

- Support for `PowerShell` to work around length limitations for environment variable on Windows.

- Support for `Pytest` to run Python tests (instead of using `nose`).

- Support to pass package specific argument.

- Update code base to Python 3.5+.

- Refactor code base to reduce coupling (e.g. separate [API](https://github.com/catkin/catkin_tools/blob/2cae17f8f32b0193384d2c7734afee1c60c4add2/catkin_tools/execution/controllers.py#L183-L205) for output handling).

- Additional functionality to build Gazebo including its dependencies.

- Whether or not to continue supporting the *devel space* concept in ROS 1.

For option **B)** the follow items should be considered:

- Support `DESTDIR`.

- Support a feature similar to the `profile` verb of `catkin_tools`.

- Support for a shared GNU Make job sever.

- Support for `GNUInstallDirs`

  - Not sure about the status of this, it would be in `colcon`’s generated shell files if anywhere.

  - Should have a test for this case.

- Test for, and fix if necessary, correct topological order with dependencies across workspaces.

  - See: [https://github.com/ros/catkin/pull/590](https://github.com/ros/catkin/pull/590)

# 总结和结论

Summary and Decision

Based on the above information a decision has been made to pick `colcon` as the universal build tool.The decision was made after considering the input of ROS 2 team members and some ROS 1 users. The decision was not easy, as it was not unanimous, but the vast majority of input was either pro `colcon` or ambivalent.To elaborate on the rationale one significant advantage of `colcon` is that it is ready to be deployed for ROS 2 right now and it covers our current use cases. Another argument leaning towards `colcon` is the expected little effort to provide devel / PR / prerelease jobs on build.ros2.org across all targeted platforms for the upcoming *Bouncy* release. While some additional feature and usability options are still missing they can be added in the future whenever there is time and/or demand for them.The necessary up front development effort for `catkin_tools` to achieve the goals described for *Bouncy* would distract the ROS 2 team from spending their time on feature development and bug fixing of ROS 2 itself.While the short term advantages are certainly a main reason for the decision in favor of `colcon` they are not the only ones. The cleaner architecture, modularity and extensibility as well as Python 3.5 code base will be valuable long term benefits when developing this tool in the future. The separation of the build tool name from the supported build systems as well as the separation from being a “ROS-only” tool will hopefully also help users to understand the difference and attract new users and potential contributors.

# 与ROS1

- Since `colcon` can be used to build ROS 1, early adopters can try to use it to build ROS 1 from source. While there is documentation how to migrate from [catkin_make_isolated](http://colcon.readthedocs.io/en/released/migration/catkin_make_isolated.html) and [catkin_tools](http://colcon.readthedocs.io/en/released/migration/catkin_tools.html) `colcon` won’t be the recommended build tool in ROS 1 for the foreseeable future.

- If a test buildfarm using `colcon` proofs to deliver the exact same results as the ROS 1 buildfarm using `catkin_make_isolated` it might be changed to use `colcon` in the future to benefit from the features `colcon` provides (like non-interleaved output per package when building in parallel, per package log files, etc.).
