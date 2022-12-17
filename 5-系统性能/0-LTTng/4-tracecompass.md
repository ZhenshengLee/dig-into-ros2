# 功能

- Real-time deadline investigation

- Latency analysis

- Log correlation with operating system traces

- Network packet correlation across layers

- Identification of relevant information in large amounts of trace data

- Critical path analysis

- Causes of high processor usage and memory leaks

- Correlation of hardware and software components execution traces

- Time synchronization of traces from different nodes

- Symbol name resolution using debug information

- Additional analyses available with the Trace Compass Incubator

- and more!

支持格式众多

- [Common Trace Format](http://diamon.org/ctf) (CTF), including but not limited to:

  - Linux [LTTng](https://lttng.org/) kernel traces

  - Linux LTTng-UST userspace traces

  - Linux Perf traces [converted to CTF](https://lwn.net/Articles/634333/)

  - Bare metal traces

  - Integration with the [LTTng-Analyses](https://github.com/lttng/lttng-analyses) scripts

- Hardware traces (e.g. IEEE Nexus 5001 CTF conversion). See also [this link](http://www.multicore-association.org/workgroup/tiwg.php).

- [GDB traces](https://sourceware.org/gdb/onlinedocs/gdb/Tracepoints.html) for debugging

- The [Best Trace Format](https://wiki.eclipse.org/images/e/e6/TA_BTF_Specification_2.1.3_Eclipse_Auto_IWG.pdf) (BTF) for [OSEK](http://en.wikipedia.org/wiki/OSEK)

- The [libpcap](http://wiki.wireshark.org/Development/LibpcapFileFormat) (Packet CAPture) format, for network traces

- Custom text or XML parsers that can be added right from the graphical interface by the user

- Can be extended to support various log or trace files.

- Provided by the TraceCompass Incubator:

  - Linux [FTrace](http://archive.eclipse.org/tracecompass.incubator/doc/org.eclipse.tracecompass.incubator.ftrace.doc.user/User-Guide.html) raw textual format

  - Google's [trace event](https://docs.google.com/document/d/1CvAClvFfyA5R-PhYUmn5OOQtYMH4h6I0nSsKchNAySU/preview#!) json format

  - Additional [Linux Perf2ctf](http://archive.eclipse.org/tracecompass.incubator/doc/org.eclipse.tracecompass.incubator.perf.profiling.doc.user/User-Guide.html) traces features

  - [Android traces](http://archive.eclipse.org/tracecompass.incubator/doc/org.eclipse.tracecompass.incubator.atrace.doc.user/User-Guide.html)

  - [UFtrace format](http://archive.eclipse.org/tracecompass.incubator/doc/org.eclipse.tracecompass.incubator.uftrace.doc.user/User-Guide.html)

# 操作

## 新建项目

new

配置为trace project

引入一个trace文件

# trace控制

# 参考

[Trace Compass User Guide](https://archive.eclipse.org/tracecompass/doc/stable/org.eclipse.tracecompass.doc.user/User-Guide.html)

[GitHub - tuxology/tracevizlab: Trace visualization tutorials for Trace Compass](https://github.com/tuxology/tracevizlab)

[Trace Compass - Eclipsepedia](https://wiki.eclipse.org/Trace_Compass)

[Trace Compass User Guide - Installation](https://archive.eclipse.org/tracecompass/doc/stable/org.eclipse.tracecompass.doc.user/Installation.html#Trace_Compass_Plug-ins)

[Looking for feedback on ros2_tracing and use-cases/needs for performance analysis & visualization](https://discourse.ros.org/t/looking-for-feedback-on-ros2-tracing-and-use-cases-needs-for-performance-analysis-visualization/18720)

[How to use Tracecompass to analyse kernel traces from LTTng](https://www.codethink.co.uk/articles/2020/how-to-use-tracecompass-to-analyse-kernel-traces-from-lttng/)
