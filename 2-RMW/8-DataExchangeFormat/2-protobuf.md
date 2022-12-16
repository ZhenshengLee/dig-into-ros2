# 概述

谷歌出品，使用量大，配套完善

cyber使用，大量互联网基础设施使用

# 消息表示

![](https://tcs.teambition.net/storage/312h58770ec219fa29a2d9433206d3fe5391?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTIzNywiaWF0IjoxNjcxMTk0NDM3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmg1ODc3MGVjMjE5ZmEyOWEyZDk0MzMyMDZkM2ZlNTM5MSJ9.cV430EFZzSCsGg1OSyZ3oft3B36vb-u0PceeJZzaq0E&download=image.png "")

# 编解码

[__https://developers.google.com/protocol-buffers/docs/encoding__](https://developers.google.com/protocol-buffers/docs/encoding)

# cpp类型映射

[__https://github.com/lixiangyun/protobuf_doc_ZH_CN#%E6%A0%87%E9%87%8F%E6%95%B0%E5%80%BC%E7%B1%BB%E5%9E%8B__](https://github.com/lixiangyun/protobuf_doc_ZH_CN#%E6%A0%87%E9%87%8F%E6%95%B0%E5%80%BC%E7%B1%BB%E5%9E%8B)

protobuf int8

protobuf repeated bytes

Bytes基于C++ STL中的string实现

在message中对repeated 标识的原始数据类型的字段需要做大量ADD操作（例如超过3千）时，可以考虑预分配数据空间，避免重复大量地分配空间。

常量，不支持

typedef

# cpp使用

![](https://tcs.teambition.net/storage/312heda26edcc623a7add971bfe193c12611?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY3MTc5OTIzNywiaWF0IjoxNjcxMTk0NDM3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMmhlZGEyNmVkY2M2MjNhN2FkZDk3MWJmZTE5M2MxMjYxMSJ9.xw7tzEk83hvvXemF94KCwGTmRltHx5sFBWzV3N5j5Vs&download=image.png "")

# 参考

[https://developers.google.com/protocol-buffers/docs/overview](https://developers.google.com/protocol-buffers/docs/overview)

[https://github.com/lixiangyun/protobuf_doc_ZH_CN](https://github.com/lixiangyun/protobuf_doc_ZH_CN)

[https://developers.google.com/protocol-buffers/docs/reference/cpp-generated](https://developers.google.com/protocol-buffers/docs/reference/cpp-generated)

[Protobuf在Cmake中的正确使用](https://segmentfault.com/a/1190000039369187)

[https://developers.google.com/protocol-buffers/docs/proto3](https://developers.google.com/protocol-buffers/docs/proto3)

[【详细解析版】Protobuf3使用手册](https://zhuanlan.zhihu.com/p/348498896)

[https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.repeated_field#RepeatedField.Resize.details](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.repeated_field#RepeatedField.Resize.details)
