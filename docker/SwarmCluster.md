[**Linux**](../Linux.md)
# Swarm Cluster
###  I taken total 5 nodes  relatively hostnames and ip below 
```
node1	10.0.85.3
node2	10.0.85.4
node3	10.0.85.5
node4	10.0.85.6
node5	10.0.85.7
```

```
[node1] (local) root@10.0.85.3 ~docker swarm init --advertise-addr 10.0.85.3

$ docker swarm init --advertise-addr 10.0.85.3
Swarm initialized: current node (ucjgqrduodw23m4odcgboa65e) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-5xgzs1gyxk0muizytmr6y33jt1w6u02ka1zl82w9bna407oiux-ehbgd7gq05lus0zka2aumn99l
\
    10.0.85.3:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.



[node1] (local) root@10.0.85.3 ~
$ docker swarm join-token  manager
To add a manager to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-5xgzs1gyxk0muizytmr6y33jt1w6u02ka1zl82w9bna407oiux-521pmd72lqm729fk82bhzczex
\
    10.0.85.3:2377

[node1] (local) root@10.0.85.3 ~
$
```



## I ADDED Node2 & Node3 also masters
```
[node2] (local) root@10.0.85.4 ~
$ docker swarm join --token SWMTKN-1-5xgzs1gyxk0muizytmr6y33jt1w6u02ka1zl82w9bna407oiux-521pmd72lqm729fk82bhzczex    10.0.85.3:2377
This node joined a swarm as a manager.
[node2] (local) root@10.0.85.4 ~
$
```

```
[node3] (local) root@10.0.85.5 ~
$ docker swarm join --token SWMTKN-1-5xgzs1gyxk0muizytmr6y33jt1w6u02ka1zl82w9bna407oiux-521pmd72lqm729fk82bhzczex    10.0.85.3:2377
This node joined a swarm as a manager.
[node3] (local) root@10.0.85.5 ~
$
```

### I ADDED Node4 & Node5 also WORKSERS
```
[node4] (local) root@10.0.85.6 ~
$ docker swarm join  --token SWMTKN-1-5xgzs1gyxk0muizytmr6y33jt1w6u02ka1zl82w9bna407oiux-ehbgd7gq05lus0zka2aumn99l    10.0.85.3:2377
This node joined a swarm as a worker.
[node4] (local) root@10.0.85.6 ~
$
```

```
[node5] (local) root@10.0.85.7 ~
$ docker swarm join  --token SWMTKN-1-5xgzs1gyxk0muizytmr6y33jt1w6u02ka1zl82w9bna407oiux-ehbgd7gq05lus0zka2aumn99l    10.0.85.3:2377
This node joined a swarm as a worker.
[node5] (local) root@10.0.85.7 ~
$
```


~~~yml
[node1] (local) root@10.0.85.3 ~
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
lzlpnwsb7b437885cbhqtanch    node2     Ready   Active        Reachable
r5d62fxqotp1k8s5hg8tgk05u    node3     Ready   Active        Reachable
ucjgqrduodw23m4odcgboa65e *  node1     Ready   Active        Leader
y3i0cmf3nrtwer9o9wyz5y6rv    node4     Ready   Active
zqzehi7qucky0e1qttdqltmtb    node5     Ready   Active
[node1] (local) root@10.0.85.3 ~
$ docker node demote node1
Manager node1 demoted in the swarm.
[node1] (local) root@10.0.85.3 ~
$ docker node ls
Error response from daemon: rpc error: code = 14 desc = grpc: the connection is unavailable
[node1] (local) root@10.0.85.3 ~
$
~~~


```
[node2] (local) root@10.0.85.4 ~
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
lzlpnwsb7b437885cbhqtanch *  node2     Ready   Active        Leader
r5d62fxqotp1k8s5hg8tgk05u    node3     Ready   Active        Reachable
ucjgqrduodw23m4odcgboa65e    node1     Ready   Active
y3i0cmf3nrtwer9o9wyz5y6rv    node4     Ready   Active
zqzehi7qucky0e1qttdqltmtb    node5     Ready   Active
[node2] (local) root@10.0.85.4 ~
$
```
~~~bash
[node1] (local) root@10.0.85.3 ~
$ docker swarm leave
Node left the swarm.
$
~~~

```
[node2] (local) root@10.0.85.4 ~
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
lzlpnwsb7b437885cbhqtanch *  node2     Ready   Active        Leader
r5d62fxqotp1k8s5hg8tgk05u    node3     Ready   Active        Reachable
ucjgqrduodw23m4odcgboa65e    node1     Down    Active
y3i0cmf3nrtwer9o9wyz5y6rv    node4     Ready   Active
zqzehi7qucky0e1qttdqltmtb    node5     Ready   Active
[node2] (local) root@10.0.85.4 ~
$
```

## NOW again promote master node1
~~~bash
[node1] (local) root@10.0.85.3 ~
$  docker swarm join --token SWMTKN-1-5xgzs1gyxk0muizytmr6y33jt1w6u02ka1zl82w9bna407oiux-521pmd72lqm729fk82bhzczex    10.0.85.4:2377
This node joined a swarm as a manager.
[node1] (local) root@10.0.85.3 ~
$
~~~

```
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
ies2uh6xwt7eun1vk14aoe4xa *  node1     Ready   Active        Reachable
lzlpnwsb7b437885cbhqtanch    node2     Ready   Active        Leader
r5d62fxqotp1k8s5hg8tgk05u    node3     Ready   Active        Reachable
ucjgqrduodw23m4odcgboa65e    node1     Down    Active
y3i0cmf3nrtwer9o9wyz5y6rv    node4     Ready   Active
zqzehi7qucky0e1qttdqltmtb    node5     Ready   Active

[node1] (local) root@10.0.85.3 ~
$ docker node rm node1
Error response from daemon: node node1 is ambiguous (2 matches found)

[node1] (local) root@10.0.85.3 ~
$ docker node rm ucjgqrduodw23m4odcgboa65e
ucjgqrduodw23m4odcgboa65e

[node1] (local) root@10.0.85.3 ~
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
ies2uh6xwt7eun1vk14aoe4xa *  node1     Ready   Active        Reachable
lzlpnwsb7b437885cbhqtanch    node2     Ready   Active        Leader
r5d62fxqotp1k8s5hg8tgk05u    node3     Ready   Active        Reachable
y3i0cmf3nrtwer9o9wyz5y6rv    node4     Ready   Active
zqzehi7qucky0e1qttdqltmtb    node5     Ready   Active
[node1] (local) root@10.0.85.3 ~
```
~~~bash
[node1] (local) root@10.0.85.3 ~
$ docker network create --driver overlay veerun1
isehtwmtr9cgfbouan2ypp0x3
[node1] (local) root@10.0.85.3 ~
$ docker service create --name nginx --replicas 10 --network veerun1   --publish 80:80 nginx
vg3l31wq71yittxnnt3a449y9
[node1] (local) root@10.0.85.3 ~
~~~

```
[node1] (local) root@10.0.85.3 ~
$ docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.04.0-ce
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host ipvlan macvlan null overlay
Swarm: active
 NodeID: ies2uh6xwt7eun1vk14aoe4xa
 Is Manager: true
 ClusterID: m4tkoktproy2z8262pgwde3gc
 Managers: 3
 Nodes: 5
 Orchestration:
  Task History Retention Limit: 5
 Raft:
  Snapshot Interval: 10000
  Number of Old Snapshots to Retain: 0
  Heartbeat Tick: 1
  Election Tick: 3
 Dispatcher:
  Heartbeat Period: 5 seconds
 CA Configuration:
  Expiry Duration: 3 months
 Node Address: 10.0.85.3
 Manager Addresses:
  10.0.85.3:2377
  10.0.85.4:2377
  10.0.85.5:2377
Runtimes: runc
Default Runtime: runc
Init Binary:
containerd version: 422e31ce907fd9c3833a38d7b8fdd023e5a76e73
runc version: 9c2d8d184e5da67c95d601382adf14862e4f2228
init version: 949e6fa
Security Options:
 apparmor
 seccomp
  Profile: default
Kernel Version: 4.4.0-66-generic
Operating System: Alpine Linux v3.5 (containerized)
OSType: linux
Architecture: x86_64
CPUs: 16
Total Memory: 120.1GiB
Name: node1
ID: MWDZ:XTNN:IDRF:OM4L:KI6P:VGQW:JMJP:AYQB:HUFB:X76E:BDML:IFXW
Docker Root Dir: /graph
Debug Mode (client): false
Debug Mode (server): true
 File Descriptors: 44
 Goroutines: 150
 System Time: 2017-04-12T16:48:22.709294828Z
 EventsListeners: 0
Registry: https://index.docker.io/v1/
Experimental: true
Insecure Registries:
 127.0.0.1
 127.0.0.0/8
Live Restore Enabled: false

WARNING: No swap limit support
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```

~~~bash
[node1] (local) root@10.0.85.3 ~
$ docker node ps
ID                  NAME                IMAGE               NODE                DESIRED STATE
  CURRENT STATE            ERROR               PORTS
lmrvrwxjfqfc        nginx.2             nginx:latest        node1               Running
  Starting 3 seconds ago
wqbrybedz5sx        nginx.6             nginx:latest        node1               Running
  Starting 3 seconds ago
~~~

```
[node1] (local) root@10.0.85.3 ~
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS
           PORTS               NAMES
8e2a648576ca        nginx:latest        "nginx -g 'daemon ..."   16 seconds ago      Up Less than
a second   80/tcp, 443/tcp     nginx.6.wqbrybedz5sxo041bqal6ti4h
ad27a5ab3fbb        nginx:latest        "nginx -g 'daemon ..."   16 seconds ago      Up Less than
a second   80/tcp, 443/tcp     nginx.2.lmrvrwxjfqfc59xifjciqfhot
```

~~~bash
[node1] (local) root@10.0.85.3 ~
$ docker service ps nginx
ID                  NAME                IMAGE               NODE                DESIRED STATE
  CURRENT STATE            ERROR               PORTS
ur0n6axku124        nginx.1             nginx:latest        node4               Running
  Running 15 seconds ago
lmrvrwxjfqfc        nginx.2             nginx:latest        node1               Running
  Running 14 seconds ago
n0geaaq9mdlp        nginx.3             nginx:latest        node4               Running
  Running 16 seconds ago
8eplgjqpd5w3        nginx.4             nginx:latest        node5               Running
  Running 16 seconds ago
8ng1yyxzpgzn        nginx.5             nginx:latest        node3               Running
  Running 16 seconds ago
wqbrybedz5sx        nginx.6             nginx:latest        node1               Running
  Running 14 seconds ago
lkw9pmvj9hrr        nginx.7             nginx:latest        node2               Running
  Running 15 seconds ago
xnt2ehmxyg12        nginx.8             nginx:latest        node5               Running
  Running 19 seconds ago
j9g327lgafi8        nginx.9             nginx:latest        node2               Running
  Running 18 seconds ago
sfk6hz0a8paj        nginx.10            nginx:latest        node3               Running
  Running 16 seconds ago
~~~  

```
[node1] (local) root@10.0.85.3 ~
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
589c95cf4230        bridge              bridge              local
e4a82521f939        docker_gwbridge     bridge              local
4e694df7d372        host                host                local
0cgfbgqsz5wd        ingress             overlay             swarm
90189b832e13        none                null                local
isehtwmtr9cg        veerun1             overlay             swarm
[node1] (local) root@10.0.85.3 ~
$
```

