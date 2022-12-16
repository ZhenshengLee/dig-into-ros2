0.8.2版本

# 编译选项

![](https://tcs.teambition.net/storage/312h9495b894a0c87afa4ef3bee36253654f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5NDgwOSwiaWF0IjoxNjcxMTkwMDA5LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg5NDk1Yjg5NGEwYzg3YWZhNGVmM2JlZTM2MjUzNjU0ZiJ9.rEZ5bRh451Q8KpdpfPiWqR8vNVpBlcEQQcDOU9cew2A&download=image.png "")

ENABLE_SHM默认开，此处的SHM只包括iceoryx的zc-shm

# 传输支持

## udp模式

默认传输模式

## shm

shm传输在0.9才支持，目前版本是0.8.x

## shm-zc

通过iceoryx使用zc，但是不会像iceoryx一样，还是会有拷贝过程

## intra

没用过

# XML配置

# 性能测试

```bash
cd /opt/ros/galactic-dev/bin/
# 1
ddsperf pub size 1024k
# 2
ddsperf sub
```

# 参考

[Welcome to Eclipse Cyclone DDS’s documentation! — Eclipse Cyclone DDS 0.8.2 documentation](https://cyclonedds.io/docs/cyclonedds/0.8.2/)

[https://github.com/eclipse-cyclonedds/cyclonedds/tree/releases/0.8.x/docs/](https://github.com/eclipse-cyclonedds/cyclonedds/tree/releases/0.8.x/docs/)
