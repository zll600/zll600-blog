---
title: learn-leveldb
date: 2023-01-02 12:14:04
tags:
---

[leveldb](https://github.com/google/leveldb)在实现上使用了 LSM-Tree 来存储数据

- [manual](https://github.com/google/leveldb/blob/main/doc/index.md)
- [impl](https://github.com/google/leveldb/blob/main/doc/impl.md)

LSM-Tree 的主要优势是将磁盘的随机写入转化为了顺序写入,从而提高了写入速度

这里就记录一下我个人对于源码的一些理解和学习

LevelDB 支持两种写入方式, Put 和 Delete

这里先分析一下 Put 的整个流程

在写入的时候,首先会将用户提供的 WriteBatch 封装为一个 DBImpl::Writer 来,同一个时刻,只能有一个 Writer 在写,其他 Writer 均在等待,同时会再调用 MakeRoomForWrite 来为当前的写入提供 MemTable, BuildBatchGroup 会把后续的多个写入都合并为一个 WriteBatch,来做批量写入,

这里会先写 log,然后会根据指定的设置来决定在修改 memtable 之前 flush log,根据 WriteOptions 中的注释,如果 options.sync 为 false 的话,如果 machine crash,那么有可能会丢失最近的 write,然后将更新 append 到 memtable 中,

```c++
  if (status.ok() && updates != nullptr) {  // nullptr batch is for compactions
    WriteBatch* write_batch = BuildBatchGroup(&last_writer);
    WriteBatchInternal::SetSequence(write_batch, last_sequence + 1);
    last_sequence += WriteBatchInternal::Count(write_batch);

    // Add to log and apply to memtable.  We can release the lock
    // during this phase since &w is currently responsible for logging
    // and protects against concurrent loggers and concurrent writes
    // into mem_.
    {
      mutex_.Unlock();
      status = log_->AddRecord(WriteBatchInternal::Contents(write_batch));
      bool sync_error = false;
      if (status.ok() && options.sync) {
        status = logfile_->Sync();
        if (!status.ok()) {
          sync_error = true;
        }
      }
      if (status.ok()) {
        status = WriteBatchInternal::InsertInto(write_batch, mem_);
      }
      mutex_.Lock();
      if (sync_error) {
        // The state of the log file is indeterminate: the log record we
        // just added may or may not show up when the DB is re-opened.
        // So we force the DB into a mode where all future writes fail.
        RecordBackgroundError(status);
      }
    }
    if (write_batch == tmp_batch_) tmp_batch_->Clear();
```

- memtable: 内部负责存储的是一个 skiplist, 在 memtable Add 时,会将 key 和 value format 为固定的 entry,然后调用 skiplist 的 Insert 方法,这里有一点需要注意的是,这个 skiplist 是不支持重复插入同一个 Key 的
  接下来还有写入前,当 L0 的 SSTable 达到上限之后,会 delay 所有的写线程 1ms,而不是 delay 某一个写线程,同时这里的 delay 可以让出 cpu 给 compaction 线程使用,这里也不会 delay 同一写线程多次, 如果当前 memtable 还有 space 那么就 break 循环,如果当前 memtable 已经满了,但是 older memtable 还在 compaction,那么就等待 wakeup,如果有太多的 L0 file 也等待唤醒,其他情况下就切换到新的 memtable,然后出发 older memtable 的 compaction

- MaybeScheduleCompaction 会调用 BackgroundCompaction,首先会调用 CompactMemTable 来对 immtable 来做 compaction,会将 immtable 写入 level 0 的文件中,在 BackgroundCompaction 之后,还会在调用一次 MaybeScheduleCompaction 来避免一个 level 产生了过多的文件.在 CompactMemTable 结束之后,会进行 GC 来删除已经过期的文件

- 对于读流程来讲,在从文件或者 MemTable 中读的时候并不会持有锁,首先会利用指定的 key 在当前 snapshot 构造一个 LookupKey 来进行查找,会在 MemTable 中先进行查找,如果没有找到且存在 immtable 就继续在 immtable 中进行查找,如果也没有找到就在当前 Version 中进行查找,

- 在 MemTable (包括 mem 和 imm)上的查找操作都是使用 Table::Iterator 来做的,

- 接着在当前 Version 中查找,首先在 level0 的文件中从新到旧的查找,这里用到一个技巧,利用 level0 文件内容有序的特性,这里可以用每个文件的 smallestkey 和 largestkey 来判断是否存在,

- 如果没有找到的话,就去更到的 Level 文件中查找,从 level1 开始,查找所有的 level 的所有文件,这里在搜索每个文件的时候使用了二分查找来加速

- 在 leveldb 中存在一个 Version 的类,这个类记录当前 version 下的数据库的一些信息,另外还有一个 Version 的集合类 VersionSet 包含了多个 version,这些 version 使用一个手写的 double-linkedlist 来存储, current\_ 表示当前使用的 Version,此外还有一个 VersionEdit 表示对一个 Version 做出变更的所有操作,对上一个 Version 应用 VersionEdit 就可以得到一个新的 Version, 另外这里还存在一个 VersionSet::Builder 的类,这个类用来 wrap VersionEdit 并处理将其应用到 Version 上,

- 在从持久化存储恢复的过程中,会首先对 VersionSet 做恢复,首先从 log 中恢复 VersionEdit, 然后使用 VerionSet::Builder 来 apply 这个 VersionEdit, log 中的一条 record 就对应一个 VersionEdit, 将 VersionEdit 的内容存储到 Builder 之后,会用当前的 VersionSet 构造一个新的 Version, 并将这个新的 Version 加入到 VersionSet 中,接着就就是读取 log,然后生成 MemTable,并 compaction 到更高的 level 的 SSTable.
