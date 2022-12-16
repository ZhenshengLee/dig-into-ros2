# ccache使用

```text
which ccache
ccache --version
3.3.7
ccache -s
zs@zs-3630:~/zs_ws/ga_ros2$ ccache -s
cache directory                     /home/zs/.ccache
primary config                      /home/zs/.ccache/ccache.conf
secondary config      (readonly)    /etc/ccache.conf
stats updated                       Mon Jun 27 11:38:26 2022
cache hit (direct)                   260
cache hit (preprocessed)              30
cache miss                           246
cache hit rate                     54.10 %
called for link                      294
no input file                         42
cleanups performed                     0
files in cache                       690
cache size                          51.7 MB
max cache size                       5.0 GB


```

  默认情况下cache（缓存）会保存在~/.ccache目录下，如果要设置主目录位于NFS或其他非本地文件系统上，可获得更高的速度，如存储器支持，速度可提高10%到25%。在配置中添加如下配置：

```bash
# home/XXXXXX/.ccache   XXXXXX是你的用户名
export CCACHE_DIR=<path-to-your-cache-directory>

```

缓冲区大小

```text
ccache -M 50G
# 默认5G
```

# ROS2使用

[Ccache](https://ccache.dev/) can speed up recompilation. It is recommended to use it to save your time unless you have a specific reason not to do so.

1. Install `Ccache`:sudo apt update && sudo apt install ccache

1. Write the following in your `.bashrc`:export CC="/usr/lib/ccache/gcc"export CXX="/usr/lib/ccache/g++"

# 效果

大大增加clean之后的编译速度

![](https://tcs.teambition.net/storage/312j12ad6b14f9b375551fd4e15ac0f9d674?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTk1NSwiaWF0IjoxNjcxMTk1MTU1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmoxMmFkNmIxNGY5YjM3NTU1MWZkNGUxNWFjMGY5ZDY3NCJ9.NkNZ3VMhJVKpT3CK_YF7eoS5RV3giSm3zuYeY3T1Ybs&download=image.png "")

![](https://tcs.teambition.net/storage/312jfcbd1b94e1ef3b4f3a7264ad1a3cdd87?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTk1NSwiaWF0IjoxNjcxMTk1MTU1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmpmY2JkMWI5NGUxZWYzYjRmM2E3MjY0YWQxYTNjZGQ4NyJ9.J4q2LnEHRYh31A1XiZIhHSiiEugVlYtNY9UbrkVDUzI&download=image.png "")

# 使用建议

没有特殊原因，坚持使用ccache

# 后续

distcc分布式编译，将编译过程分布到多台机器中

# 参考

[高速的C/C++编译工具——ccache_king_weng的博客-CSDN博客_ccache](https://blog.csdn.net/King_weng/article/details/117415618)
