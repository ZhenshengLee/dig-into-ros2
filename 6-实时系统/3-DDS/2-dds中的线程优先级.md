# dds线程模型

dds会创建很多线程

patficipant, publisher, subscriber都会创建对应的线程

查阅对应dds的threading model文档

# RTI

## 线程模型

![](https://tcs.teambition.net/storage/312gfb2335058dafbb4836f57e5fe5bebeef?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMjk5OCwiaWF0IjoxNjcxNDE4MTk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdmYjIzMzUwNThkYWZiYjQ4MzZmNTdlNWZlNWJlYmVlZiJ9.2QxXRmXA4mePjGA1R84GoMuOfoQH2BuvaJLpCgIYTHk&download=image.png "")

![](https://tcs.teambition.net/storage/312g6ede1f640ca7e08c46e5e99fd5e70a5d?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMjk5OCwiaWF0IjoxNjcxNDE4MTk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmc2ZWRlMWY2NDBjYTdlMDhjNDZlNWU5OWZkNWU3MGE1ZCJ9.5esEs2BIcWiZSfOccXiolEdFmlHN-BMUosJMMTpbPqQ&download=image.png "")

## 配置线程优先级

可以通过xml配置文件，以及cpp api进行线程优先级配置

![](https://tcs.teambition.net/storage/312gca467e2fa3c05a88403179e2fb110c9f?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMjk5OCwiaWF0IjoxNjcxNDE4MTk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdjYTQ2N2UyZmEzYzA1YTg4NDAzMTc5ZTJmYjExMGM5ZiJ9.ydn-hCqLljmnTAgXEtWglNJ0aphbiGmc9W70XTZ84Ac&download=image.png "")

# FastDDS

## 线程模型

![](https://tcs.teambition.net/storage/312g5c6556845bb2d84e695b4864f198a731?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMjk5OCwiaWF0IjoxNjcxNDE4MTk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmc1YzY1NTY4NDViYjJkODRlNjk1YjQ4NjRmMTk4YTczMSJ9.4IX91jvTbNmLUESpgjZl8faqoTePUUJ-SY8z9Rv3Xjs&download=image.png "")

## 优先级配置

![](https://tcs.teambition.net/storage/312gadc1620c161d757eb656aeb2182b8ac2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMjk5OCwiaWF0IjoxNjcxNDE4MTk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmdhZGMxNjIwYzE2MWQ3NTdlYjY1NmFlYjIxODJiOGFjMiJ9.SzVMg2zoDd5LtjN2emNKc2X7_X7dZSBHEVHVV0Svi1A&download=image.png "")

# eCAL

## 线程模型

![](https://tcs.teambition.net/storage/312g980d47774e62ac86c9510f4ac2bec455?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MjAyMjk5OCwiaWF0IjoxNjcxNDE4MTk4LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmc5ODBkNDc3NzRlNjJhYzg2Yzk1MTBmNGFjMmJlYzQ1NSJ9.5il47-h42LtLUMS-y5c04n2BGPCKYr3xOzU29HJ0Xqs&download=image.png "")

## 配置线程优先级

不支持

# 参考

[How to change thread priority? | Data Distribution Service (DDS) Community RTI Connext Users](https://community.rti.com/forum-topic/how-change-thread-priority)

[Connext DDS Threading Model](https://community.rti.com/static/documentation/connext-dds/5.3.0/doc/manuals/connext_dds/html_files/RTI_ConnextDDS_CoreLibraries_UsersManual/Content/UsersManual/threading.htm)

[Configuring Thread Settings with XML](https://community.rti.com/static/documentation/connext-dds/5.2.3/doc/manuals/connext_dds/html_files/RTI_ConnextDDS_CoreLibraries_UsersManual/Content/UsersManual/ConfigThreadSettings_XML.htm)

[RTI Connext DDS](https://community.rti.com/static/documentation/connext-dds/6.1.0/doc/manuals/connext_dds_professional/platform_notes/index.htm#platform_notes/Linux_Platforms.htm#5.7_Thread_Configuration%3FTocPath%3D5.%2520Linux%2520Platforms%7C5.7%2520Thread%2520Configuration%7C_____0)

[rtiperftest/command_line_parameters.rst at master · rticommunity/rtiperftest](https://github.com/rticommunity/rtiperftest/blob/master/srcDoc/command_line_parameters.rst)

[Managing thread priority [13594] · Discussion #2435 · eProsima/Fast-DDS](https://github.com/eProsima/Fast-DDS/discussions/2435)

[Threading model — eCAL Documentation  documentation](https://continental.github.io/ecal/advanced/threading_model.html)
