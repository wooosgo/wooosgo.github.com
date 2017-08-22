---
layout: post
title: "Redis Administration"
date:   2017-07-3 10:00:01 +0900
categories: redis
layout: post
---

## Persistence

### [RDB vs AOF]

**RDB** - point in time snapshot, dump.rdb, save(bgsave) 60 1000
>
Advantage
1. for backups/disaster recovery
2. fork ; parent does not perform disk i/o (only child does)
3. faster restart with big dataset (compared to AOF)
>
Disadvantage
1. chance of data loss (when power outage) --> save points interval matters
2. fork() is costly (for cpu)

**AOF** - log of operations, appendonly yes
>
Advantage
1. fsync ; no fsync, every second, every query
2. append only log ; no corruption in general (+ redis-check-aof tool)
>
Disadvantage
1. bigger than RDB
2. slower than RDB
3. bug prone

**Conclusion**
>
* use both  
* few minutes are acceptable when failure, use RDB only  
* (will be)unifying into a single model in the future  
* RDB file is safe to be copied (for backup into other storage)  
