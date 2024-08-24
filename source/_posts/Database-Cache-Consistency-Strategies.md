---
title: "Database Cache Consistency Strategies"
date: 2023-09-07 23:28:11
description: "Database Cache Consistency Strategies"
categories: System Design
tags:
- Database
- Cache
---

Database cache consistency strategies include:
- Cache Aside
- Read/Write Through
- Write Behind Caching

<!--more-->
# Cache Aside Pattern

*Eventually Database Cache Consistency*

- Cache miss: The application retrieves data from the cache. If data doesn’t exists in the cache, retrieves data from database and stores it in the cache. The requested data is returned from database.
- Cache hit: The application retrieves data from the cache and returns.
- Update data: Stores the data in the database first, and then invalidates the cache after success.

[Cache-aside caching strategy](https://catsincode.com/caching-strategy/)
<img src="./1.png" width="40%" height="40%" alt="1">


Cache-aside strategy fits well for data that is read often and not so often stored or updated. One downside of this strategy is possible data inconsistency between the cache and the database.

Consider the following situation.

<img src="./inconsistency_between_database_and_cache.png" alt="inconsistency_between_database_and_cache">

inconsistency between the cache and the database

However, the probability of occurrence may be very low in practice, because the write operation of the database is slower than the read operation. To avoid inconsistency of data, you can set a timeout on data in cache and it will automatically be deleted in a time period.

# **Read/Write Through Pattern**

*Strong Database Cache Consistency*

Application views the backend as a single storage, and the storage maintains its own Cache.

**Read Through**

The application doesn’t orchestrate where to read the data from (cache or database). Instead, it always reads data through the cache. When the cache is invalid (expired or LRU swapped out) during query operation(request), the cache service itself loads data from database.

**Write Through**

The Write Through is similar to Read Through, but happens when data is updated. When there is data updated,

- if the cache is not hit, update the database directly, and then return.
- If the cache is hit, the cache is updated, and updates the data in database. Both operations happen in a single transaction(this is a synchronous operation)

[A write-through cache with no-write allocation](https://en.wikipedia.org/wiki/Cache_(computing))
<img src="./2.png" width="40%" height="40%" alt="2">

It does create some extra latency on writes but at least it improves the data inconsistency problem greatly as the data in the cache and data storage is the same. Another advantage is, because application only talks to the cache, the code is much cleaner and simpler.

# **Write Behind Caching Pattern**

*Eventually Database Cache Consistency*

Write-behind(also call Write Back) caching Pattern is similar to the write-through cache in a way that application only communicates with the cache. The difference is that the data gets written to the cache first. And then, written to the underlying database asynchronously. The advantage of this design is that the I/O operation of the data is extremely fast. Also, due to asynchronous update in database, the writes can be collected and then does a batch write to the database.

However, the problem with this is that the data is not strongly consistent and may be lost. To tackle the data inconsistency problem a system could combine the write-behind strategy with the read-through strategy. In this way up to date data is always to be read from the cache. In addition, the implementation logic of Write Back is more complicated, because it needs to track which data has been updated and needs to be stored in database.

When we compare it to the write-through strategy, it fits more for systems with large write and read volume that tolerate some data inconsistency. Another useful case is that when execution order of data updated doesn’t matter, like the number of page views, every time the user clicks, this field in the database is incremented by one. Execution order of SQL/request would not cause thread safety problem in concurrently write and read, since each SQL/request does the same(page views field incremented by one).

# **Final**

We do not consider the transaction of the cache and under layer storage. For example, the cache is updated successfully, but the update of the database fails. Or the other way reverse. If you need strong consistency, you need to consider Transaction Processing in a Distributed System, like “two-phase commit protocol” — — prepare, commit/rollback.

# **Reference**

[Cache Consistency with Database](https://danielw.cn/cache-consistency-with-database#concepts)

[Caching Strategies Overview](https://catsincode.com/caching-strategy/)

[缓存更新的套路](https://catsincode.com/caching-strategy/)

[Cache (computing)](https://en.wikipedia.org/wiki/Cache_(computing))

[Cache-Aside pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/cache-aside)