构建Linux with RT_PREEMPT

# 注意

官方源kernel不能更新，第三方驱动会失效

# 安装内核

## 下载

As of Jan 21, 2021, Nvidia now natively supports the PREEMPT_RT kernel patch on the Jetson boards. Steps to install:

```bash
sudo echo 'deb https://repo.download.nvidia.com/jetson/rt-kernel r32.5 main' \
        >> /etc/apt/sources.list.d/nvidia-l4t-apt-source.list
sudo apt update
sudo apt install nvidia-l4t-rt-kernel nvidia-l4t-rt-kernel-headers
```

系统从ssd启动，拷贝启动文件到media

```bash
sudo cp -f /boot/Image.real-time /media/nvidia/09dc9e98-169c-49bc-b17f-48a1caf49d72/boot/
sudo mv /media/nvidia/09dc9e98-169c-49bc-b17f-48a1caf49d72/boot/extlinux/extlinux.conf /media/nvidia/09dc9e98-169c-49bc-b17f-48a1caf49d72/boot/extlinux/extlinux.conf.bak
sudo cp -f /boot/extlinux/extlinux.conf /media/nvidia/09dc9e98-169c-49bc-b17f-48a1caf49d72/boot/extlinux/extlinux.conf

```

# 验证

```bash
uname -a
4.9.140-rt93 #2 SMP PREEMPT RT

```

# 自定义内核

If you want to customize the kernel source

Find the source as a bzip file here under “Linux Driver Package (BSP) Sources”.

See instructions here on how to build it.

由于硬件更改，内核源代码由供应商把控，不对客户开放，所以不可行。

# 参考

[Setting up Realtime Kernel on Jetson • PREEMPT_RT on Jetson](https://orenbell.com/?p=436)

[https://r7vme.medium.com/real-time-kernel-for-nvidia-agx-xavier-b660e107a211](https://r7vme.medium.com/real-time-kernel-for-nvidia-agx-xavier-b660e107a211)
