# 1-概述

## 1.1-AP DataTypes

本文档不关注所有的应用程序DataTypes，而是关注需要进行language-binding的数据类型<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669875179879-302297bc-3126-4d3c-b60f-e934b69a10bf.png#averageHue=%23f7f3ee&clientId=u6064fe07-4c61-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=809&id=u28b9bac9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=809&originWidth=883&originalType=binary&ratio=1&rotation=0&showTitle=false&size=114014&status=done&style=none&taskId=ue043f0fe-0700-46d7-876f-975b84d148b&title=&width=883)

## 1.2-languageBinding

AP主要关注cppbinding，但也不局限

## 1.3-Methodology

本文档规定了APDataTypes如何转换序列化为language level的数据类型，能够被编译器所识别处理。<br />1the term "serialization" should not be mixed with (de-)serialization in the context of Communication<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669875333382-919c48e9-e007-4395-883f-5631f5977430.png#clientId=u6064fe07-4c61-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=177&id=ubd3e5941&margin=%5Bobject%20Object%5D&name=image.png&originHeight=177&originWidth=837&originalType=binary&ratio=1&rotation=0&showTitle=false&size=46482&status=done&style=none&taskId=ub972c962-ddfe-4a7c-b48c-3f1d889efbf&title=&width=837)

# 5-依赖

LBAP不是一个FC，所以对其他FC没有依赖

# 7-功能规约

LBAP不是一个FC，所以没有功能规约，但是其生成的各个数据类型会被ARA app和其他FC所使用到。<br />As explained in 1.1, AutosarDataTypes referenced by elements of any Adaptive Platform Interface, e.g.:<br />• ServiceInterface.event<br />• ServiceInterface.method<br />• ServiceInterface.field<br />• PersistencyKeyValueStorageInterface.dataElement

## 7.1-cpp

### 7.1.1-LBAP

![image.png](https://cdn.nlark.com/yuque/0/2022/png/23125517/1669875724220-27415d2b-ee7f-43b6-8189-be38290172b2.png#averageHue=%23f2e1c7&clientId=u6064fe07-4c61-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=572&id=uad413250&margin=%5Bobject%20Object%5D&name=image.png&originHeight=572&originWidth=844&originalType=binary&ratio=1&rotation=0&showTitle=false&size=130527&status=done&style=none&taskId=uafce6671-3f59-4a87-a326-a71ba4c8803&title=&width=844)

### 7.1.2-基本类型
