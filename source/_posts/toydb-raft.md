---
title: toydb-raft
date: 2023-02-06 10:00:57
tags:
---

toydb： https://github.com/erikgrinaker/toydb

### heartbeat 

heartbeat 主要涉及的就是 follower 和 Leader 了，
首先分析 follower 的逻辑
1. 首先对 msg 进行校验，主要就是 normalization 和 validation
2. 如果 msg 中是更新的 term，那么就更新自己的 term，并成为 from 的 follower，同时如果当前 node 没有 leader，也会成为 from 的 follower
3. 更新 leader 之后，如果确认这个 msg 来源于 leader,那么就清除 election timeout ticks
4. handle hearbeat 类型的 msg，如果 follower 的 log 有符合条件的 `(index, term)`，那么就可以进行 commit.

其他逻辑的更新你没有写，不过实现上也不是很难，结合测试用例，还是可以比较方便的搞明白的


### 接下来就是 candidate 的逻辑了

- 在 become_leader 这里，candidate 会先向所有的 peer 广播一条 heartbeat 来表示自己是 leader，然后 append 一条空日志, 原因应该可以参考这里 https://wanghenshui.github.io/2019/08/08/raft.html

````text
在failover时，新的Leader由于没有持久化commitIndex，所以并不清楚当前日志的commitIndex在哪，也即不清楚log entry是committed还是uncommitted状态。通常在成为新Leader时提交一条空的log entry来提交之前所有的entry。
````

- 在 candidate 执行 step 的时候，如果遇到更高 term 的 msg，则会降级为 follower 来 handle 这条 msg
- 如果 candidate 收到 term == self.term 的 heartbeat msg，则会降级为 follower 来 handle 这条 heartbeat


### Leader 的逻辑
个人觉得这个还可以改