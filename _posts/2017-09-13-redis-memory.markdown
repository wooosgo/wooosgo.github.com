---
layout: post
title: "Redis memory management"
date:   2017-09-13 10:00:01 +0900
categories: redis
layout: post
---

# redis-cli 에서 제공하는 memory 정보

1. info (memory)
2. memory (stats)


# info memory

127.0.0.1:30003> info memory

used_memory:2715488    // 메모리 allocator(libc, jemallc, tcmalloc) 를 통해 redis 가 사용 중인 총 메모리(단위 : bytes)
used_memory_human:2.59M // 위와 동일함(읽기 쉽게 MB로 단위를 변환)
used_memory_rss:3584000      // OS가 봤을 때 redis 가 점유하고 있는 총 메모리양. resident set size. top이나 ps명령에서 보이는 단위.
used_memory_rss_human:3.42M  
used_memory_peak:2779856     // peak 상태일 때 메모리 점유량(redis가 메모리를 반환 해도 allocator가 os에 즉시 반환하지 않음)
used_memory_peak_human:2.65M
used_memory_peak_perc:97.68%
used_memory_overhead:2643770
used_memory_startup:1511200
used_memory_dataset:71718
used_memory_dataset_perc:5.96%
total_system_memory:17179869184 // 시스템 총 메모리
total_system_memory_human:16.00G
used_memory_lua:37888        // lua 엔진이 사용하고 있는 메모리
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction   // maxmemory 정책. 상세 내용 참조
mem_fragmentation_ratio:1.32    // used와 used_rss 의 비율. 상세 내용 참조
mem_allocator:libc    // 메모리 allocator(libc, jemallc, tcmalloc 등). compile 시 설정 가능.
active_defrag_running:0
lazyfree_pending_objects:0
127.0.0.1:30003>

# maxmemory 정책

noeviction	기존 데이터 삭제 안함. 메모리 한계에 도달하면 OOM 오류 반환. 새 데이터 저장되지 않음
allkeys-lru	LRU로 삭제하여 공간확보하고 새 데이터 저장
volatile-lru	expire set을 가진 것 중 LRU로 삭제하여 공간확보하고 새 데이터 저장
allkeys-random	랜덤으로 삭제하여 공간확보하고 새 데이터 저장
volatile-random	expire set을 가진 것 중에서 랜덤으로 삭제하여 공간확보하고 새 데이터 저장
volatile-ttl	expire set을 가진 것 중 TTL이 짧은 것부터 삭제하여 공간확보하고 새 데이터 저장

일반적인 데이터 보존 용도로는 noeviction
캐시 용도로는 allkeys-lru 사용 권장

# mem_fragmentation_ratio

일반적으로는 rss가 used보다 조금 높은 게 정상이다.
만일 rss > used 차이가 크면 메모리 파편화(fragmentation) 을 의심해 볼 수 있다(레디스 내부 또는 외부적인 파편화)
한편, rss < used 인 경우, redis 메모리 일부가 OS에 의해 swap 이 발생 한 경우로 볼 수 있고 이 때 redis 성능은 크게 떨어진 상태이다.
used_memory_rss 가 정상 범위 이상으로 커지지 않았는지, 해당 수치를 모니터링 하는 것이 필요.


# memory command

127.0.0.1:30003> memory help
1) "MEMORY USAGE <key> [SAMPLES <count>] - Estimate memory usage of key"
2) "MEMORY STATS                         - Show memory usage details"
3) "MEMORY PURGE                         - Ask the allocator to release memory"
4) "MEMORY MALLOC-STATS                  - Show allocator internal stats"

# memory stats

127.0.0.1:30003> memory stats
 1) "peak.allocated"
 2) (integer) 2779856
 3) "total.allocated"
 4) (integer) 2715472
 5) "startup.allocated"
 6) (integer) 1511200
 7) "replication.backlog"
 8) (integer) 1048576
 9) "clients.slaves"
10) (integer) 16858
11) "clients.normal"
12) (integer) 66488
13) "aof.buffer"
14) (integer) 0
15) "db.0"
16) 1) "overhead.hashtable.main"
    2) (integer) 648
    3) "overhead.hashtable.expires"
    4) (integer) 0
17) "overhead.total"
18) (integer) 2643770
19) "keys.count"
20) (integer) 13
21) "keys.bytes-per-key"
22) (integer) 92636
23) "dataset.bytes"
24) (integer) 71702
25) "dataset.percentage"
26) "5.9539704322814941"
27) "peak.percentage"
28) "97.683906555175781"
29) "fragmentation"
30) "1.319782018661499"
127.0.0.1:30003>

info memory 에 비해 조금 더 상세한 metric 제공
memory profiling tool 을 위한 metric 제공

# memory usage keys

각 key 들이 점유하는 메모리 양의 추정치를 보여준다 (단위 bytes)

127.0.0.1:30003> get x
"1"
127.0.0.1:30003> smembers foo
1) "b"
2) "a"
127.0.0.1:30003> memory usage x
(integer) 49
127.0.0.1:30003> memory usage foo
(integer) 231
127.0.0.1:30003>

# memory doctor

몇 가지 metric 을 기반으로 장황한(약간 장난스런) 진단 결과를 제공한다

Peak is > 150% of current used memory? : mh->peak_allocated / mh->total_allocated) > 1.5

Fragmentation is higher than 1.4? : mh->fragmentation > 1.4

Slaves using more than 10 MB each? : mh->clients_slaves / numslaves > (1024*1024*10)

Clients using more than 200k each average? : (mh->clients_normal / numclients(normal.clients - slave.clients) > (1024*200)

총 사용 메모리가 5MB 미만인 경우

127.0.0.1:30003> memory doctor
Hi Sam, this instance is empty or is using very little memory, my issues detector can't be used in these conditions. Please, leave for your mission on Earth and fill it with some data. The new Sam and I will be back to our programming as soon as I finished rebooting.
127.0.0.1:30003>
