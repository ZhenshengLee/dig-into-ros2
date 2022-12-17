# 原理

![](https://tcs.teambition.net/storage/312h4b419330557b9917360415a2e12d797a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxNiwiaWF0IjoxNjcxMjQ2NjE2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg0YjQxOTMzMDU1N2I5OTE3MzYwNDE1YTJlMTJkNzk3YSJ9.NX4DU5zhA7fy9iIvs6ZSUhLj8BvOKMmisuhxF5JXvSg&download=image.png "")

# 例子

![](https://tcs.teambition.net/storage/312h70675c798c7645a24785da7c0f0b3be6?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxNiwiaWF0IjoxNjcxMjQ2NjE2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg3MDY3NWM3OThjNzY0NWEyNDc4NWRhN2MwZjBiM2JlNiJ9.lw5u3iFnc9lpl0ZRcoq8-zWJl3GLnHGIuwxow_y71Lw&download=architecture.png "")

初次运行，打开trace，rqt_graph记录，检查基础功能，生成架构文件

topic1:11hz,  0.09s, 执行0.01秒

topic2:7hz,  0.14s，执行0.01s

topic3:  topic1, 执行0.01s

topic4: topic3, 执行0.01s

topic5: topic2, 执行0.01s

topic6: 3hz, 0.33s, 执行0.01s

回调执行时间全部是0.01s，10ms

# 测试目标

测试no_dependency_node的回调执行时间

# 启动例子

```bash
ros2 launch e2e_demo demo1.launch.py
```

# 传统方法

```python
# 处理工具
from tracetools_analysis.loading import load_file
from tracetools_analysis.processor.ros2 import Ros2Handler
from tracetools_analysis.utils.ros2 import Ros2DataModelUtil

# 加载
events = load_file(path)
handler = Ros2Handler.process(events)
#handler.data.print_data()

# 分析
data_util = Ros2DataModelUtil(handler.data)

callback_symbols = data_util.get_callback_symbols()

output_notebook()
psize = 450
colours = ['#29788E', '#DD4968', '#410967', '#410967', '#410967', '#410967', '#410967']

# 画图
```

![](https://tcs.teambition.net/storage/312h6f5fbf1530b753a788710216ed6dca6d?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxNiwiaWF0IjoxNjcxMjQ2NjE2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg2ZjVmYmYxNTMwYjc1M2E3ODg3MTAyMTZlZDZkY2E2ZCJ9.yAiTAexGsxaLWStJLFTX9zNe-yU5JSNHtFUZtZeAA-8&download=image.png "")

![](https://tcs.teambition.net/storage/312hdde084b5eef70cdfa201cb3638b01e7f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxNiwiaWF0IjoxNjcxMjQ2NjE2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhkZGUwODRiNWVlZjcwY2RmYTIwMWNiMzYzOGIwMWU3ZiJ9.AkehM1YbymSaLMrGj80WJ4A22tDWV154wmiLYEwbMXE&download=image.png "")

# e2e方法

## 构建arch

```json
{
  "target_path": {
    "start_node_name": "/sensor_dummy_node",
    "end_node_name": "/no_dependency_node"
  },
  "nodes": [
    {
      "name": "/sensor_dummy_node",
      "callback_dependency": {
        "": ""
      },
      "publish": {
        "/topic1": "SensorDummy::SensorDummy()::{lambda()#1}",
        "/topic2": "SensorDummy::SensorDummy()::{lambda()#2}"
      },
      "callbacks": [
        {
          "type": "timer_callback",
          "period": 90909090,
          "symbol": "SensorDummy::SensorDummy()::{lambda()#1}"
        },
        {
          "type": "timer_callback",
          "period": 142857142,
          "symbol": "SensorDummy::SensorDummy()::{lambda()#2}"
        }
      ]
    },
    {
      "name": "/no_dependency_node",
      "publish": {
        "/topic3": "NoDependencyNode::NoDependencyNode()::{lambda(std::unique_ptr<sensor_msgs::msg::Image>)#1}"
      },
      "callbacks": [
        {
          "type": "subscribe_callback",
          "topic_name": "/topic1",
          "symbol": "NoDependencyNode::NoDependencyNode()::{lambda(std::unique_ptr<sensor_msgs::msg::Image>)#1}"
        }
      ]
    }
  ],
  "path_name_alias": {
    "NoDependencyNode::NoDependencyNode()::{lambda(std::unique_ptr<sensor_msgs::msg::Image>)#1}": "dependency_node_callback"
  }
}

```

## 运行程序

先确认lttng trace是否正常

```bash
babeltrace ./e2e_demo1/ | grep -e rclcpp_publish -e rclcpp_subscribe
babeltrace ./e2e_demo1/ | grep -e rclcpp_timer_added

```

## 分析脚本

略

## 结果

![](https://tcs.teambition.net/storage/312hff431412db94cc5a243fed96bfdefeba?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxNiwiaWF0IjoxNjcxMjQ2NjE2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhmZjQzMTQxMmRiOTRjYzVhMjQzZmVkOTZiZmRlZmViYSJ9.m30IM59sOAi30e_WRmAlZqtUZZ0TmqzqN2hFBdrDPg8&download=image.png "")

![](https://tcs.teambition.net/storage/312h5003d56ae9c0292d3b6013a438a1be3e?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxNiwiaWF0IjoxNjcxMjQ2NjE2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg1MDAzZDU2YWU5YzAyOTJkM2I2MDEzYTQzOGExYmUzZSJ9.qNnvfFKqvUES97Y-gNMoaPrdVZ5Z54RENzTNg9cH8ho&download=image.png "")

# 程序和trace分开启动

这样可以自主决定何时终止trace

启动trace

```bash
# kernel事件置空
ros2 trace -k -s standalong_callback_duration_demo

```

启动程序，按需启动trace

```bash
ros2 launch e2e_demo standalong_callback_duration_demo.launch.py

```

不停止程序，停止trace即可

有用户需求，可以不用在启动程序的时候就启动trace，程序运行过程中也可记录

不支持启动程序后再trace，因为缺少rcl_init初始化信息，无法分析数据

![](https://tcs.teambition.net/storage/312h951b50994fb92526be8006a6f9ccef9f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxNiwiaWF0IjoxNjcxMjQ2NjE2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg5NTFiNTA5OTRmYjkyNTI2YmU4MDA2YTZmOWNjZWY5ZiJ9.DgvZFmH8SyFSxstcJzN2Rx_HcCdo7759Cf1JETNVMvw&download=image.png "")

所以只能先启动trace，然后再启动程序，这种有缺点，但是也没办法，缺少启动信息就没法分析了。

缺点：系统启动时刻的一次性时间无法记录，如timer_added

![](https://tcs.teambition.net/storage/312hfed508430403ebf1563ba7b8ca256c30?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxNiwiaWF0IjoxNjcxMjQ2NjE2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhmZWQ1MDg0MzA0MDNlYmYxNTYzYmE3YjhjYTI1NmMzMCJ9.qsT19zY0i8Mw2NrjVSc3fyHU_yeBMoDnvpdYHobh9WA&download=image.png "")

![](https://tcs.teambition.net/storage/312he0fc6d9322ea98ce54b3c9beff5b67f1?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxNiwiaWF0IjoxNjcxMjQ2NjE2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhlMGZjNmQ5MzIyZWE5OGNlNTRiM2M5YmVmZjViNjdmMSJ9.S1z8zhOhPJZsXMMWq3V-bdaCCAAuLOtvepuI4Z9nuWw&download=image.png "")

# 前面时间截断

有需求将前面3秒的数据截断，

如定位程序的场景，前面在手动初始定位，回调时间可能执行很短，后面才开始配准

基于python处理timeseries即可

![](https://tcs.teambition.net/storage/312h471cae72fa03e60b5d1c43b60c861c10?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTg1MTQxNiwiaWF0IjoxNjcxMjQ2NjE2LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg0NzFjYWU3MmZhMDNlNjBiNWQxYzQzYjYwYzg2MWMxMCJ9.N9L1KUIbfEJYLCuWeW0m1q_pGb5oLj7Gy4nL97M-dW0&download=image.png "")
