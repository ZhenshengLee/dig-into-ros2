# 平台配置

## cpu频率设置

Frequency is setup to 1.50 GHz for all CPUs

```bash
# run it as root
sudo su

echo -n "setup constant CPU frequency to 1.50 GHz ... "
# disable ondemand governor
systemctl disable ondemand

# set performance governor for all cpus
echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor >/dev/null

# set constant frequency
echo 1500000 | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_min_freq >/dev/null
echo 1500000 | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_max_freq >/dev/null

# reset frequency counters
echo 1 | tee /sys/devices/system/cpu/cpu*/cpufreq/stats/reset >/dev/null

echo done

sleep 1
# get freq info
echo `cpufreq-info | grep stats | cut -d ' ' -f 23-25`
```

## 资源隔离

CPU 2,3 are isolated to run tests.

```text
sudo apt install -y sysstat u-boot-tools
```

```bash
# modify kernel cmdline
cd ~
dd if=/boot/firmware/boot.scr of=boot.script bs=72 skip=1

# edit boot.script and modify bootargs to
ubuntu@ubuntu:~$ cat boot.script | grep "setenv bootargs" | head -1
setenv bootargs " ${bootargs} rcu_nocbs=2,3 nohz_full=2,3 isolcpus=2,3 irqaffinity=0,1 audit=0 watchdog=0 skew_tick=1 quiet splash"

# generate boot.scr
mkimage -A arm64 -O linux -T script -C none -d boot.script boot.scr

# replace boot.scr
sudo cp boot.scr /boot/firmware/boot.scr

sudo reboot

# check cmdline
ubuntu@ubuntu:~$ cat /proc/cmdline
 coherent_pool=1M 8250.nr_uarts=1 snd_bcm2835.enable_compat_alsa=0 snd_bcm2835.enable_hdmi=1 bcm2708_fb.fbwidth=0 bcm2708_fb.fbheight=0 bcm2708_fb.fbswap=1 smsc95xx.macaddr=DC:A6:32:2E:5
4:97 vc_mem.mem_base=0x3ec00000 vc_mem.mem_size=0x40000000  net.ifnames=0 dwc_otg.lpm_enable=0 console=ttyS0,115200 console=tty1 root=LABEL=writable rootfstype=ext4 elevator=deadline roo
twait fixrtc rcu_nocbs=2,3 nohz_full=2,3 isolcpus=2,3 irqaffinity=0,1 audit=0 watchdog=0 skew_tick=1 quiet splash

# check interrupts
# Only the number of interrupts handled by CPU 0,1 increases.
watch -n1 cat /proc/interrupts

# check soft interrupts
watch -n1 cat /proc/softirqs

# check isolated CPUs
cat /sys/devices/system/cpu/isolated
2-3
cat /sys/devices/system/cpu/present
0-3

# run reference system on CPU2
taskset -c 2 install/autoware_reference_system/lib/autoware_reference_system/autoware_default_singlethreaded > /dev/null

# get pid
RF_PID=`pidof autoware_default_singlethreaded` && cat /proc/$RF_PID/status | grep ^Cpu

# check how many threads are running
ps -aL | grep $RF_PID
   3835    3835 ttyS0    00:03:46 autoware_defaul
   3835    3836 ttyS0    00:00:00 autoware_defaul
   3835    3837 ttyS0    00:00:00 autoware_defaul
   3835    3838 ttyS0    00:00:00 autoware_defaul
   3835    3839 ttyS0    00:00:00 gc
   3835    3840 ttyS0    00:00:00 dq.builtins
   3835    3841 ttyS0    00:00:00 dq.user
   3835    3842 ttyS0    00:00:00 tev
   3835    3843 ttyS0    00:00:00 recv
   3835    3844 ttyS0    00:00:00 recvMC
   3835    3845 ttyS0    00:00:00 recvUC
   3835    3846 ttyS0    00:00:00 autoware_defaul
```

# 参考

[ROS 2 Real-Time benchmarks](https://ros-realtime.github.io/ros2_realtime_benchmarks/)
