# Redis Server horizantal Scaling with Hashbased Cluster

#### To install redis server 
```
yum install gcc gcc-c++ make -y
wget http://download.redis.io/releases/redis-3.0.5.tar.gz
tar xzf redis-3.0.5.tar.gz
cd redis-3.0.5
make MALLOC=jemalloc
make MALLOC=libc
cp src/redis-server src/redis-cli src/redis-sentinel  src/redis-benchmark  src/redis-check-aof  src/redis-check-dump  /usr/local/bin
cd utils
./install_server.sh
```
To run above command (./install_server.sh) multiple times and enter eachtime different ports  aprromatly 8 times cluster requrires 3 master 3 slave and 2 more master and salve  to add existing  cluster
After run to edit each server configuretion file below perameter and change portnumber

```
#daemonize yes
#port 
bind 192.168.1.15

#logfile /var/log/redis_6381.log
#dir /var/lib/redis/6382
dbfilename dump.rdb
appendonly yes
appendfilename "appendonly.aof"

cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
```



[root@localhost redis]# tree 
```
.
|-- 6379
|   `-- dump.rdb
|-- 6380
|   |-- appendonly-6380.aof
|   |-- dump.rdb
|   `-- nodes-6380.conf
|-- 6381
|   |-- appendonly_6381.aof
|   |-- dump.rdb
|   `-- nodes-6381.conf
|-- 6382
|   |-- appendonly-6382.aof
|   |-- dump.rdb
|   `-- nodes-6382.conf
|-- 6383
|   |-- appendonly-6383.aof
|   |-- dump.rdb
|   `-- nodes-6383.conf
|-- 6384
|   |-- appendonly-6384.aof
|   |-- dump.rdb
|   `-- nodes-6384.conf
|-- 6385
|   |-- appendonly-6385.aof
|   |-- dump.rdb
|   `-- nodes-6385.conf
|-- 6386
|   |-- appendonly-6386.aof
|   |-- dump.rdb
|   `-- nodes-6386.conf
|-- 6387
|   `-- dump.rdb
`-- 6389
    `-- dump.rdb
```

### To install Cluster Requires ruby library
```
yum install gcc-c++ patch readline readline-devel zlib zlib-devel  install libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison iconv-devel


curl -L get.rvm.io | bash -s stable

 source /etc/profile.d/rvm.sh

rvm install 2.1.2

rvm use 2.1.2 --default

gem install redis 
```

### Create replicaset with shard:
```

cd /root/redis-3.0.5/src
./redis-trib.rb  create --replicas 1  192.168.1.15:6380 192.168.1.15:6381 192.168.1.15:6382 192.168.1.15:6383 192.168.1.15:6384 192.168.1.15:6385
>>> Creating cluster
Connecting to node 192.168.1.15:6380: OK
Connecting to node 192.168.1.15:6381: OK
Connecting to node 192.168.1.15:6382: OK
Connecting to node 192.168.1.15:6383: OK
Connecting to node 192.168.1.15:6384: OK
Connecting to node 192.168.1.15:6385: OK
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.1.15:6380
192.168.1.15:6381
192.168.1.15:6382
Adding replica 192.168.1.15:6383 to 192.168.1.15:6380
Adding replica 192.168.1.15:6384 to 192.168.1.15:6381
Adding replica 192.168.1.15:6385 to 192.168.1.15:6382
M: 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 192.168.1.15:6380
   slots:0-5460 (5461 slots) master
M: 6da1ccad0318594e640460c9573e10f217e63875 192.168.1.15:6381
   slots:5461-10922 (5462 slots) master
M: fe645952d483fb0fc9bd0ece629831d862debce0 192.168.1.15:6382
   slots:10923-16383 (5461 slots) master
S: a0b13c3a1358e4d1a10c9eff3169103625656793 192.168.1.15:6383
   replicates 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62
S: c0e72943efed20531e542caf8bf4bbfe375b6e78 192.168.1.15:6384
   replicates 6da1ccad0318594e640460c9573e10f217e63875
S: ab3ba7d3ba1fe0aadd86e830fc0efccb92d04272 192.168.1.15:6385
   replicates fe645952d483fb0fc9bd0ece629831d862debce0
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join......
>>> Performing Cluster Check (using node 192.168.1.15:6380)
M: 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 192.168.1.15:6380
   slots:0-5460 (5461 slots) master
M: 6da1ccad0318594e640460c9573e10f217e63875 192.168.1.15:6381
   slots:5461-10922 (5462 slots) master
M: fe645952d483fb0fc9bd0ece629831d862debce0 192.168.1.15:6382
   slots:10923-16383 (5461 slots) master
M: a0b13c3a1358e4d1a10c9eff3169103625656793 192.168.1.15:6383
   slots: (0 slots) master
   replicates 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62
M: c0e72943efed20531e542caf8bf4bbfe375b6e78 192.168.1.15:6384
   slots: (0 slots) master
   replicates 6da1ccad0318594e640460c9573e10f217e63875
M: ab3ba7d3ba1fe0aadd86e830fc0efccb92d04272 192.168.1.15:6385
   slots: (0 slots) master
   replicates fe645952d483fb0fc9bd0ece629831d862debce0
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

```


### To check weather Cluster node list:
```
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6380
192.168.1.15:6380> cluster nodes
a0b13c3a1358e4d1a10c9eff3169103625656793 192.168.1.15:6383 slave 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 0 1445945454988 4 connected
c0e72943efed20531e542caf8bf4bbfe375b6e78 192.168.1.15:6384 slave 6da1ccad0318594e640460c9573e10f217e63875 0 1445945454485 5 connected
ab3ba7d3ba1fe0aadd86e830fc0efccb92d04272 192.168.1.15:6385 slave fe645952d483fb0fc9bd0ece629831d862debce0 0 1445945451970 6 connected
792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 192.168.1.15:6380 myself,master - 0 0 1 connected 0-5460
fe645952d483fb0fc9bd0ece629831d862debce0 192.168.1.15:6382 master - 0 1445945453981 3 connected 10923-16383
6da1ccad0318594e640460c9573e10f217e63875 192.168.1.15:6381 master - 0 1445945455994 2 connected 5461-10922
192.168.1.15:6380> exit
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6380 CLUSTER MEET 192.168.1.15 6382
OK
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6380 CLUSTER MEET 192.168.1.15 6381
OK
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6380 CLUSTER NODES
a0b13c3a1358e4d1a10c9eff3169103625656793 192.168.1.15:6383 slave 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 0 1445945757780 4 connected
c0e72943efed20531e542caf8bf4bbfe375b6e78 192.168.1.15:6384 slave 6da1ccad0318594e640460c9573e10f217e63875 0 1445945756775 5 connected
ab3ba7d3ba1fe0aadd86e830fc0efccb92d04272 192.168.1.15:6385 slave fe645952d483fb0fc9bd0ece629831d862debce0 0 1445945754764 6 connected
792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 192.168.1.15:6380 myself,master - 0 0 1 connected 0-5460
fe645952d483fb0fc9bd0ece629831d862debce0 192.168.1.15:6382 master - 0 1445945759793 3 connected 10923-16383
6da1ccad0318594e640460c9573e10f217e63875 192.168.1.15:6381 master - 0 1445945760797 2 connected 5461-10922

[root@localhost src]# redis-cli -h 192.168.1.15 -p 6381 CLUSTER NODES
c0e72943efed20531e542caf8bf4bbfe375b6e78 192.168.1.15:6384 slave 6da1ccad0318594e640460c9573e10f217e63875 0 1445945853940 5 connected
6da1ccad0318594e640460c9573e10f217e63875 192.168.1.15:6381 myself,master - 0 0 2 connected 5461-10922
792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 192.168.1.15:6380 master - 0 1445945855447 1 connected 0-5460
ab3ba7d3ba1fe0aadd86e830fc0efccb92d04272 192.168.1.15:6385 slave fe645952d483fb0fc9bd0ece629831d862debce0 0 1445945858465 6 connected
fe645952d483fb0fc9bd0ece629831d862debce0 192.168.1.15:6382 master - 0 1445945856452 3 connected 10923-16383
a0b13c3a1358e4d1a10c9eff3169103625656793 192.168.1.15:6383 slave 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 0 1445945857459 4 connected
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6382 CLUSTER NODES
a0b13c3a1358e4d1a10c9eff3169103625656793 192.168.1.15:6383 slave 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 0 1445945865701 4 connected
ab3ba7d3ba1fe0aadd86e830fc0efccb92d04272 192.168.1.15:6385 slave fe645952d483fb0fc9bd0ece629831d862debce0 0 1445945864696 6 connected
792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 192.168.1.15:6380 master - 0 1445945863688 1 connected 0-5460
c0e72943efed20531e542caf8bf4bbfe375b6e78 192.168.1.15:6384 slave 6da1ccad0318594e640460c9573e10f217e63875 0 1445945867714 5 connected
fe645952d483fb0fc9bd0ece629831d862debce0 192.168.1.15:6382 myself,master - 0 0 3 connected 10923-16383
6da1ccad0318594e640460c9573e10f217e63875 192.168.1.15:6381 master - 0 1445945866707 2 connected 5461-10922



[root@localhost src]# redis-cli -h 192.168.1.15 -p 6380 -c
192.168.1.15:6380> SET ze zt
-> Redirected to slot [7057] located at 192.168.1.15:6381
OK
192.168.1.15:6381> exit
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6381 -c
192.168.1.15:6381> KEYS *
1) "ze"
192.168.1.15:6381> exit



[root@localhost src]# /etc/init.d/redis_6387 start 
Starting Redis server...
[root@localhost src]# 
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6387 CLUSTER NODES
9f8c99f649b2d8f68b7773d92a5dc2c94d4386c6 :6387 myself,master - 0 0 0 connected
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6380 CLUSTER NODES
a0b13c3a1358e4d1a10c9eff3169103625656793 192.168.1.15:6383 slave 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 0 1445951769509 4 connected
c0e72943efed20531e542caf8bf4bbfe375b6e78 192.168.1.15:6384 slave 6da1ccad0318594e640460c9573e10f217e63875 0 1445951769006 5 connected
ab3ba7d3ba1fe0aadd86e830fc0efccb92d04272 192.168.1.15:6385 slave fe645952d483fb0fc9bd0ece629831d862debce0 0 1445951768502 6 connected
792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 192.168.1.15:6380 myself,master - 0 0 1 connected 0-5460
fe645952d483fb0fc9bd0ece629831d862debce0 192.168.1.15:6382 master - 0 1445951767494 3 connected 10923-16383
6da1ccad0318594e640460c9573e10f217e63875 192.168.1.15:6381 master - 0 1445951767998 2 connected 5461-10922
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6380 CLUSTER MEET 192.168.1.15 6387
OK
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6380 CLUSTER NODES
9f8c99f649b2d8f68b7773d92a5dc2c94d4386c6 192.168.1.15:6387 master - 0 1445951802504 0 connected
a0b13c3a1358e4d1a10c9eff3169103625656793 192.168.1.15:6383 slave 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 0 1445951801701 4 connected
c0e72943efed20531e542caf8bf4bbfe375b6e78 192.168.1.15:6384 slave 6da1ccad0318594e640460c9573e10f217e63875 0 1445951798682 5 connected
ab3ba7d3ba1fe0aadd86e830fc0efccb92d04272 192.168.1.15:6385 slave fe645952d483fb0fc9bd0ece629831d862debce0 0 1445951802705 6 connected
792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 192.168.1.15:6380 myself,master - 0 0 1 connected 0-5460
fe645952d483fb0fc9bd0ece629831d862debce0 192.168.1.15:6382 master - 0 1445951803709 3 connected 10923-16383
6da1ccad0318594e640460c9573e10f217e63875 192.168.1.15:6381 master - 0 1445951800695 2 connected 5461-10922
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6380 CLUSTER NODES
9f8c99f649b2d8f68b7773d92a5dc2c94d4386c6 192.168.1.15:6387 master - 0 1445951825331 0 connected
a0b13c3a1358e4d1a10c9eff3169103625656793 192.168.1.15:6383 slave 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 0 1445951828346 4 connected
c0e72943efed20531e542caf8bf4bbfe375b6e78 192.168.1.15:6384 slave 6da1ccad0318594e640460c9573e10f217e63875 0 1445951823821 5 connected
ab3ba7d3ba1fe0aadd86e830fc0efccb92d04272 192.168.1.15:6385 slave fe645952d483fb0fc9bd0ece629831d862debce0 0 1445951825835 6 connected
792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 192.168.1.15:6380 myself,master - 0 0 1 connected 0-5460
fe645952d483fb0fc9bd0ece629831d862debce0 192.168.1.15:6382 master - 0 1445951826840 3 connected 10923-16383
6da1ccad0318594e640460c9573e10f217e63875 192.168.1.15:6381 master - 0 1445951827845 2 connected 5461-10922
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6380 CLUSTER NODES
9f8c99f649b2d8f68b7773d92a5dc2c94d4386c6 192.168.1.15:6387 master - 0 1445951831869 0 connected
a0b13c3a1358e4d1a10c9eff3169103625656793 192.168.1.15:6383 slave 792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 0 1445951829856 4 connected
c0e72943efed20531e542caf8bf4bbfe375b6e78 192.168.1.15:6384 slave 6da1ccad0318594e640460c9573e10f217e63875 0 1445951834884 5 connected
ab3ba7d3ba1fe0aadd86e830fc0efccb92d04272 192.168.1.15:6385 slave fe645952d483fb0fc9bd0ece629831d862debce0 0 1445951833377 6 connected
792f25a2933a4cda2e36ef3d20d0d86b4fb37f62 192.168.1.15:6380 myself,master - 0 0 1 connected 0-5460
fe645952d483fb0fc9bd0ece629831d862debce0 192.168.1.15:6382 master - 0 1445951833879 3 connected 10923-16383
6da1ccad0318594e640460c9573e10f217e63875 192.168.1.15:6381 master - 0 1445951832873 2 connected 5461-10922
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6382 -c
192.168.1.15:6382> KEYS *
(empty list or set)
192.168.1.15:6382> exit
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6381 -c
192.168.1.15:6381> KEYS *
1) "v3"
2) "ze"
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6381 CLUSTER SETSLOT 5481 MIGRATING 9f8c99f649b2d8f68b7773d92a5dc2c94d4386c6
OK
[root@localhost src]# redis-cli -h 192.168.1.15 -p 6387 CLUSTER SETSLOT 5481 IMPORTING  9f8c99f649b2d8f68b7773d92a5dc2c94d4386c6
OK
[root@localhost src]# redis-cli  -h 192.168.1.15 -p 6387
192.168.1.15:6387> KEYS *
(empty list or set)
192.168.1.15:6387> exit
```


### To ADD one node as a master node : ()new node 192.168.2.130:6779)
```
./redis-trib.rb add-node   192.168.2.130:6779 192.168.2.130:6179(masterip existing replica )
```

### To ADD one node as a Slave node 
```
./redis-trib.rb add-node  --slave   192.168.2.130:6879 192.168.2.130:6179(masterip existing replica )
```
./redis-trib.rb add-node  --slave --master-id 2234000be1f0f846d148075deb7608c6b7bd3d28   192.168.2.130:6879 192.168.2.130:6179

 reshard --from <node-id> --to <node-id> --slots <number of slots> --yes <host>:<port>
 

