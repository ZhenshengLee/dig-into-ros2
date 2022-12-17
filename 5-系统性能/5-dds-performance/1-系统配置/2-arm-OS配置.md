# CPU-performance

```bash
# jetson
sudo jetson_clocks

# tool
sudo apt-get install cpufrequtils
# to get the highest freq
sudo cpufreq-set -g performance

# common
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

# 网络协议栈

```bash
sudo sysctl -a | grep net.core.wmem_max
net.core.wmem_max = 1048576
sudo sysctl -a | grep net.core.rmem_max
net.core.rmem_max = 4194304

sudo sysctl -w net.ipv4.udp_mem="10240087380016777216"
sudo sysctl -w net.core.netdev_max_backlog="30000"
sudo sysctl -w net.core.rmem_max="67108864"
sudo sysctl -w net.core.wmem_max="67108864"
sudo sysctl -w net.core.rmem_default="67108864"
sudo sysctl -w net.core.wmem_default="67108864"

# 20971520=20MB
# 67108864=64MB


```

# 关闭swap

```bash
# 临时
sudo swapoff -a
sudo nano /etc/fstab
# 注释掉最后一行 /swap.img

```
