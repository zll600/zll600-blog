---
title: learn-rosedb
date: 2023-01-23 15:18:50
tags:
---



[rosedb](https://github.com/flower-corp/rosedb)



这里之分析 string 类型的相关操作



### 启动流程

这里核心函数就是 `RoseDB::Open` 这个函数，在调用这个函数的时候，会阻塞调用 `initDiscard` 初始化 discard 文件，调用`loadFromFiles` 从磁盘文件中载入数据，接着调用 `loadIndexFromLogFile 从磁盘文件中重建索引，



`loadFromFiles` 会从设置每种类型的数据对应的的 Log 文件 `fidMap`，这些文件都是 `read only` 的，也会记录在所有文件对应的只读文件集合中 `archivedLogFile`，另外需要注意的一点是约定每种数据类型的 LogFile 的 fid 都是递增的，所以在载入的时候，必须按照升序载入数据

`loadIndexFromFile` 时，会根据已经存在的 `read only` 文件集合，在内存中建立索引，这里也使用了并行来进行提速，

之后会起一个 goroutine 来做 GC,`handleLogFileGC` ，



### 写入流程

这里的 Set 操作同  Redis:

- 如果 key 不存在，会同时设置 key， value，
- 如果 key 已经存在，则只会更新 key 对应的 value

这里会直接对 key 做 assignment，不会 care key 是否已经存在了，

如果当前 append 的文件的大小已经超出了启动时设定的 `Options.LogFileSizeThreshold` ，则会将当前使用的 log 文件加入到  `archiveLogFiles` 中，然后打开一个新的的 log 文件，



首先会先 append log，然后更新 strindex：



这里对 str index 加的是读写锁，str index 就是一棵 radix-tree，



关于 radix-tree 可以参考这里: 

关于 mmap 的资料：

- https://www.zhihu.com/question/522132580



### 读取流程

