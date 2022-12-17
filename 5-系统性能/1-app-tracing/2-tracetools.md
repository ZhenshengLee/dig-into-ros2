# 模块组成

- `tracetools_trace`

  - wraps the LTTng Python bindings to setup and start a tracing session

  - exposes simplified setup functions with default values

  - provides an example `trace` entrypoint for tracing

    - `$ ros2 run tracetools_trace trace`

- `ros2trace`

  - provides a `ros2cli` extension `$ ros2 trace`

    - uses `tracetools_trace` functions

- `tracetools_launch`

  - provides a `Trace` action for `launch`

    - uses `tracetools_trace` functions

- `tracetools_read`

  - **wraps the babeltrace Python bindings to read CTF traces**

- `tracetools_test`

  - provides a `TraceTestCase` class extending `unittest.TestCase`

    - uses the `Trace` action with `launch` to trace the test nodes

    - provides trace-specific utility functions (e.g. assert)

- `tracetools_analysis`

  - uses `tracetools_read` to read traces

  - provides utilities to:

    - **convert CTF traces to pickle files**

    - **wrap trace events in Python**`**dict**`

    - **handle and process trace events to gather data**

- `ros2trace_analysis`

  - provides a `ros2cli` extension with verbs

    - `$ ros2 trace-analysis`

  - uses/exposes `tracetools_analysis` functions

    - `$ ros2 trace-analysis convert`

    - `$ ros2 trace-analysis process`

tracetoosl_ana单独成库，变成更大的包

![](https://tcs.teambition.net/storage/312hf7f379f48321fedc3626360e5b344ff1?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5MCwiaWF0IjoxNjcxMjQ2NTkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhmN2YzNzlmNDgzMjFmZWRjMzYyNjM2MGU1YjM0NGZmMSJ9.L0kD9uX5hrqF4khy2sWl9i07L2AR4tCxCVn6oMM3jMM&download=image.png "")

# analysis设计

Generally, for a given trace data analysis objective, the following classes are extended: `EventHandler`, `DataModel`, and `DataModelUtil`.A user/developer can implement an `EventHandler`, which defines callbacks for specific events. Those callbacks get called by the `Processor`, and end up putting slightly-processed data into a `DataModel`, which is a data container that uses `pandas` `DataFrame`s.Meaningful data can be extracted from the `DataModel`. However, a `DataModelUtil` can provide common utility functions so that users don't have to re-write them. This meaningful output data can then be presented through a Jupyter notebook (e.g. plots) or a normal Python script (e.g. tables).

![](https://tcs.teambition.net/storage/312he0fe69de0dc3c1aeb02367d37efe0c48?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5MCwiaWF0IjoxNjcxMjQ2NTkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhlMGZlNjlkZTBkYzNjMWFlYjAyMzY3ZDM3ZWZlMGM0OCJ9.Ju89hfFde8APonWkA-YI9cfVn4m9mdR-rX-u3-VoJfI&download=image.png "")

processor读取和分析ust事件，通过eventhandler将数据写入datamodel，通过datamodel进行分析提取出信息，通过plot工具画图

# 传统功能使用

callback_duration

```python
from tracetools_analysis.loading import load_file
from tracetools_analysis.processor.ros2 import Ros2Handler
from tracetools_analysis.utils.ros2 import Ros2DataModelUtil

# Process
events = load_file(path)
handler = Ros2Handler.process(events)
#handler.data.print_data()

data_util = Ros2DataModelUtil(handler.data)
callback_symbols = data_util.get_callback_symbols()

# Plot durations separately
colour_i = 0
for obj, symbol in callback_symbols.items():
    owner_info, temp_info = data_util.get_callback_owner_info(obj)
    if owner_info is None:
        owner_info = '[unknown]'
。。。
```

# e2e的改进

![](https://tcs.teambition.net/storage/312hdbce7f9d07806fe6b429969fd2ac5337?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5MCwiaWF0IjoxNjcxMjQ2NTkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhkYmNlN2Y5ZDA3ODA2ZmU2YjQyOTk2OWZkMmFjNTMzNyJ9.Do4ilMFqNccC9jfZpJxDQgh18xpQJrVmNXNWoYsrRyw&download=image.png "")

在processor和data_model，utils中增加了函数定义

在util中修改了get_callback_owner_info定义

e2e主要增加了rosmodel和test_interface

```python
from tracetools_analysis.ros_model import ApplicationFactory
# 实际上执行下列
from tracetools_analysis.utils.ros2 import Ros2DataModelUtil
from tracetools_analysis.loading import load_file
from tracetools_analysis.processor.ros2 import Ros2Handler

trace_path = '~/.ros/tracing/e2e_demo/ust'
architecture_path = './architecture.json'
architecture_path_template = './architecture.json.template'

# 选取指定node和topic，事先定义arch
app = ApplicationFactory.create_from_json(architecture_path)

# 读取分析数据
app.import_trace(trace_path)

# 读取回调
callback = app.callbacks[0]
callback.get_stats()

# 获取指定数据画图
hist = callback.hist(binsize_ns=10000)
latency_ms, hist = hist.get_xy()
max_ms = callback.get_stats()['max']
plt.step(latency_ms, hist, label='max: {:.1f} ms'.format(max_ms), where='post')

plt.title(callback.name)
plt.xlabel('Latency [ms]')
plt.ylabel('Probability')
plt.legend()

# 图
system_time_ns, duration_ns = callback.timeseries.get_xy()
time_s = (system_time_ns - system_time_ns[0]) * 1.0e-9
duration_ms = duration_ns * 1.0e-6
plt.step(time_s, duration_ms, label='max: {:.1f} ms'.format(max_ms), where='post')

plt.title(callback.name)
plt.xlabel('Time [s]')
plt.ylabel('Duration [ms]')
plt.legend()

```

# 附录：检测点添加

rclcpp添加检测点必须重新编译ros2

## rclcpp_publish

收集哪些信息？publisher_handle，publisher的标识，msg

![](https://tcs.teambition.net/storage/312h0551db154eff3f55bcc747dd69c4db48?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5MCwiaWF0IjoxNjcxMjQ2NTkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgwNTUxZGIxNTRlZmYzZjU1YmNjNzQ3ZGQ2OWM0ZGI0OCJ9.oc5iCAfj42062Cm6gjeLOlKJAJ_ttovFv8NPqXOI-bk&download=image.png "")

定义该检测点

![](https://tcs.teambition.net/storage/312h31a1e5905c398b51dcf9614008c5932d?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5MCwiaWF0IjoxNjcxMjQ2NTkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgzMWExZTU5MDVjMzk4YjUxZGNmOTYxNDAwOGM1OTMyZCJ9.mCz156X9KdzhE0L0GLRy18rbBY8EfHY2I3BuPK8qGCk&download=image.png "")

tp_call.h

![](https://tcs.teambition.net/storage/312h4049cb276bdf20ea85e1d9c56e157d6c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5MCwiaWF0IjoxNjcxMjQ2NTkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg0MDQ5Y2IyNzZiZGYyMGVhODVlMWQ5YzU2ZTE1N2Q2YyJ9.ctZfzarwbrY0lGMQZR2XeItgCcie_31jMnD0bom_Ao8&download=image.png "")

tools.c

![](https://tcs.teambition.net/storage/312hd33ea742492f10d443fcb1bd937c993a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5MCwiaWF0IjoxNjcxMjQ2NTkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhkMzNlYTc0MjQ5MmYxMGQ0NDNmY2IxYmQ5MzdjOTkzYSJ9.1VqyV-lfUWj_4b_d6-jIw_l8pa8Y9m2IxPkspHtunKM&download=image.png "")

tools.h

![](https://tcs.teambition.net/storage/312hf2ca18642d6de655e4a97c4597507137?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5MCwiaWF0IjoxNjcxMjQ2NTkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhmMmNhMTg2NDJkNmRlNjU1ZTRhOTdjNDU5NzUwNzEzNyJ9.M8isUbuxX3zEmnXCSHJBOFG31cF3QIW_9SRc2Xt7G8I&download=image.png "")

## image_pipline检测点

![](https://tcs.teambition.net/storage/312h17a2f799aec1756b43a67c32c423ce3c?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5MCwiaWF0IjoxNjcxMjQ2NTkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmgxN2EyZjc5OWFlYzE3NTZiNDNhNjdjMzJjNDIzY2UzYyJ9.1u7WPpiYlXYTL0gEayZG-IWLTkbKvqHseb_GThQLEZ4&download=image.png "")

注意，此处采用了新的provider，在roslaunch中要打开新的trace

![](https://tcs.teambition.net/storage/312h406e6ba6fec05f1277a168f8eb0e970f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTM5MCwiaWF0IjoxNjcxMjQ2NTkwLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg0MDZlNmJhNmZlYzA1ZjEyNzdhMTY4ZjhlYjBlOTcwZiJ9.9ElgtsFn-BfE3YRw6WouFwyPch_PHtyId7_fr0-pOmM&download=image.png "")

# 参考

[ros2_tracing/design_ros_2.md at outdoor/galactic · ZhenshengLee/ros2_tracing](https://github.com/ZhenshengLee/ros2_tracing/blob/outdoor/galactic/doc/design_ros_2.md)

[GitHub - ros-perception/image_pipeline at ros2](https://github.com/ros-perception/image_pipeline/tree/ros2)
