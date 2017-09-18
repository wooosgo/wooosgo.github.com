---
layout: post
title: "Redis Transaction"
date:   2017-09-4 10:00:01 +0900
categories: redis
layout: post
---

## AWS

# ec2 ssh 접속
ssh -i jonagear_ec2.pem ec2-user@ec2-13-124-200-222.ap-northeast-2.compute.amazonaws.com

확인사항
.pem 퍼미션 확인 (chmod 600)
access denied(public key) : key pair 인증은 되었으나 인스턴스 계정 상 문제


# elasticache 설정(redis)

ec2 에서 접속 : redis-cli 소스 다운로드 및 컴파일 필요
local 접속

- tunneling
ssh -N -L 16379:rtest.mnzcdl.clustercfg.apn2.cache.amazonaws.com:6379 ec2-user@ec2-13-124-200-222.ap-northeast-2.compute.amazonaws.com -i ./jonagear_ec2.pem &

- 접속
ssh -i jonagear_ec2.pem -v ec2-user@ec2-13-124-200-222.ap-northeast-2.compute.amazonaws.com

# Redis cluster test

서버기동
  redis-server --port 30001 --cluster-enabled yes --cluster-config-file nodes-30001.conf --cluster-node-timeout 2000 --appendonly yes --appendfilename appendonly-30001.aof --dbfilename dump-30001.rdb --logfile 30001.log --daemonize yes

  redis-server --port 30002 --cluster-enabled yes --cluster-config-file nodes-30002.conf --cluster-node-timeout 2000 --appendonly yes --appendfilename appendonly-30002.aof --dbfilename dump-30002.rdb --logfile 30002.log --daemonize yes

  redis-server --port 30006 --cluster-enabled yes --cluster-config-file nodes-30006.conf --cluster-node-timeout 2000 --appendonly yes --appendfilename appendonly-30006.aof --dbfilename dump-30006.rdb --logfile 30006.log --daemonize yes

  redis-server --port 30007 --cluster-enabled yes --cluster-config-file nodes-30007.conf --cluster-node-timeout 2000 --appendonly yes --appendfilename appendonly-30007.aof --dbfilename dump-30007.rdb --logfile 30007.log --daemonize yes


# test 내역
  - 3 노드 (replication=1) 총 6개 서버 기동

M-S 승격 기본 시나리오 확인
  - m1 kill (debug segfault 명령) 후 slave 승격 확인
  - 승격된 master 로 정상 서비스 확인
  - m2, m3 순차적으로 kill : 각각의 slave 가 master 로 승격됨
  - master 만 있는 상태에서 다시 m1을 kill 하는 경우 : 클러스터 상태가 fail 되고 서비스 불가

신규 노드 클러스터 추가 확인
  - master 또는 slave 로 추가 후 reshard 필요
  - 노드 삭제 시에도 reshard 해야 함
./redis-trib.rb add-node 127.0.0.1:30007 127.0.0.1:30001
./redis-trib.rb reshard 127.0.0.1:30007

# todos

다른 버젼의 클러스터 구성
- 3노드 (replication=2) 9 서버
- 복수 AZ 클러스터링

rdb 백업 및 복구

redis.conf 설정 개별 서버로 기동

대량 데이터 또는 부하 생성 후 rdb/aof 옵션 조정

security 옵션 설정

메모리 옵션

jedis driver 연동(bootstrap)

# from redisGate

cluster-require-full-coverage = Y ; slave가 없는 마스터의 경우 어느 하나가 fail나도 cluster가 down됨
cluster-require-full-coverage = N ; slave가 없는 마스터의 경우 어느 하나가 fail나도 cluster가 down되지 않음.
에러 난 샤드를 조회시 에러 발생하나, 나머지는 정상 서비스 가능.
단, 과반수 이상의 마스터가 fail시 클러스터는 다운된다.
(마스터 개수가 짝수인 경우, 과반수가 되지 않을 때 cluster forget 명령으로 노드 개수를 낮추어 과반수를 만들어 줘야 함)
