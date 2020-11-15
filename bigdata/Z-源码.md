[parent](./README.md)

[TOC]

## 公共

RPC 通信

追踪器

配置

参数解析

指标度量



## 源码分析

源码下载

IDEA 中关联源码



### NameNode 启动过程

主要负责文件元信息与数据块映射的管理



### DN 启动流程



### NN 



### NN 高兵发访问





Edits_log 必须保证每条都有一个 transactionId





HDFS 优化解决方案：

串行化： 使用分段锁

写磁盘： 使用双缓冲



解决刷新磁盘动作 => 后台动作