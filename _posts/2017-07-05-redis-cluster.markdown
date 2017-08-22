---
layout: post
title: "Redis Cluster"
date:   2017-07-5 10:00:01 +0900
categories: redis
layout: post
---

# Cluster #

## redis cluster basics ##


redis cluster
1. split dataset among multiple nodes
2. H/A

2 TCP ports - 6379 & 16379 (+10000 offset is fixed)

Sharding
- hash slots : CRC16(of key) % tot number of hash slots
- hash tags

Master - slave in a cluster  
A, B, C with slave A1, B1, C1  
when B fails, B1 gets promoted as a master  

>
No guarantee on strong consistency - can lose writes  

### Scenario 1 ###
1. client --> master B
2. B --> OK to client
3. B --> replicate to slaves B1, B2, B3

>
when fails before 3, writes can be lost

### Scenario 2 ###  
network partitioned 1 : A, C, A1, B1, C1  
network partitioned 2 : B, Z(client)  

when network partition took places,
- Node timeout : amount time that majority side to elect a slave as master,
The maximum amount of time a Redis Cluster node can be unavailable, without it being considered as failing.
- maximum window : amount of writes that Z can send to B

---

minimum setting of cluster
- port 7000
- cluster-enabled yes
- cluster-config-file nodes.conf
- cluster-node-timeout 5000
- appendonly yes

minimal cluster : 3 sets of nodes

* Node ID : unique identifier unchanged

create cluster by
- redis-trib script
- create-cluster script
- redis-cli -c

* CLUSTER NODES
<pre>
Node ID
ip:port
flags: master, slave, myself, fail, ...  
if it is a slave, the Node ID of the master  
Time of the last pending PING still waiting for a reply.  
Time of the last PONG received.  
Configuration epoch for this node (see the Cluster specification).  
Status of the link to this node.  
Slots served...  
</pre>

CLUSTER FAILOVER  
CLUSTER MEET  
CLUSTER REPLICATE  
