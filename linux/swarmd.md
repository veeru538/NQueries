# SwarmKit

SwarmKit is a toolkit for orchestrating distributed systems at any scale. It includes primitives for node discovery, raft-based consensus, task scheduling and more.
Installation  required packages: 
    
    sudo apt-get update
    sudo apt-get install -y docker-engine
    sudo apt-get update && sudo apt-get upgrade
    sudo init 6
    sudo usermod -aG docker $(whoami)
    sudo docker ps -a
    sudo apt-get install git 
    sudo apt-get install make
    sudo apt-get install gcc 
    wget https://storage.googleapis.com/golang/go1.8.linux-amd64.tar.gz
    tar zxf go1.8.linux-amd64.tar.gz 
    sudo mv go /usr/local/
    
To Set environment variables for go language to access  to add .profile file 
    
    vi .profile
```bash    
    export PATH=$PATH:/usr/local/go/bin
```
To download and install swarmkit
```bash
    go get -d github.com/docker/swarmkit
    cd go/src/github.com/docker/swarmkit/
    make setup
    make all
    cd bin/
    mv * /usr/bin/
```

## Setting up a Swarm:

These instructions assume that `swarmd` and `swarmctl` are in your PATH.
(Before starting, make sure `/tmp/nodeN` don't exist)

Initialize the first node (node1) node1 ip is `172.16.0.167`:
```bash
swarmd -d /tmp/node1 --listen-control-api /tmp/node1/swarm.sock --hostname node1
export SWARM_SOCKET=/tmp/node1/swarm.sock
swarmctl cluster inspect default
```
Output look like below:
```yml
swarmctl cluster inspect default
ID          : xf2sd34u2owk19ftcz4ct1ck9
Name        : default
Orchestration settings:
  Task history entries: 5
Dispatcher settings:
  Dispatcher heartbeat period: 5s
Certificate Authority settings:
  Certificate Validity Duration: 2160h0m0s
  Join Tokens:
    Worker: SWMTKN-1-374bf4b518vjczf4azobw3rs28i22ao7lh6562g3exmykgfol1-1bj3dzb62blwag1hmept85oum
    Manager: SWMTKN-1-374bf4b518vjczf4azobw3rs28i22ao7lh6562g3exmykgfol1-4yumc694uj9gjmwqyo0wfjv6h
```

Node2:

Syntax:

`swarmd -d /tmp/nodeN --hostname node-N --join-addr masternodeIP:4242 --join-token <Worker Token>`

```bash
swarmd -d /tmp/node2 --hostname node2 --join-addr 172.16.0.167:4242 --join-token SWMTKN-1-374bf4b518vjczf4azobw3rs28i22ao7lh6562g3exmykgfol1-1bj3dzb62blwag1hmept85oum
```


Node1:

To list nodes:
```bash
swarmctl node ls
```
~~~yml
ID                         Name   Membership  Status  Availability  Manager Status
--                         ----   ----------  ------  ------------  --------------
qmebebet000zd1sf9ew7gidac  node1  ACCEPTED    READY   ACTIVE        REACHABLE *
yg3pt1q8mn6iylsm9b282v714  node2  ACCEPTED    READY   ACTIVE 

~~~

```bash
swarmctl  cluster ls
```
~~~yml
ID                         Name
--                         ----
xf2sd34u2owk19ftcz4ct1ck9  default
~~~


##Creating Services
Start a redis service:

```bash
swarmctl service create --name redis --image redis:3.0.5
```
~~~yml
lhe5tau9qq5b5n7ipyphrf2um
~~~

List the running services:
```bash
swarmctl service ls
```
~~~yml
ID                         Name   Image        Replicas
--                         ----   -----        --------
lhe5tau9qq5b5n7ipyphrf2um  redis  redis:3.0.5  1/1
~~~

Inspect the service:
```bash
swarmctl service inspect redis
```
~~~yml
ID                : lhe5tau9qq5b5n7ipyphrf2um
Name              : redis
Replicas          : 1/1
Template          
 Container        
  Image           : redis:3.0.5

Task ID                      Service    Slot    Image          Desired State    Last State                Node
-------                      -------    ----    -----          -------------    ----------                ----
v22yoys1bm9zj33bcr0udns3b    redis      1       redis:3.0.5    RUNNING          RUNNING 59 seconds ago    node1
~~~

Updating Services:
you can scale the service by changing the instance count:
```bash
swarmctl service update redis --replicas 6
```
~~~yml
lhe5tau9qq5b5n7ipyphrf2um
~~~
```bash
swarmctl service inspect redis
```
~~~yml
ID                : lhe5tau9qq5b5n7ipyphrf2um
Name              : redis
Replicas          : 6/6
Template          
 Container        
  Image           : redis:3.0.5

Task ID                      Service    Slot    Image          Desired State    Last State                Node
-------                      -------    ----    -----          -------------    ----------                ----
v22yoys1bm9zj33bcr0udns3b    redis      1       redis:3.0.5    RUNNING          RUNNING 6 minutes ago     node1
qfxrh0cpudvmdq5d09vtl1582    redis      2       redis:3.0.5    RUNNING          RUNNING 26 seconds ago    node1
ya42z05tubx3c15x1vjmg2mqn    redis      3       redis:3.0.5    RUNNING          RUNNING 23 seconds ago    node2
5j890x4ohlznjirr3d3nsafnd    redis      4       redis:3.0.5    RUNNING          RUNNING 23 seconds ago    node2
wul73c17qcgcrypzuyiy8mwhl    redis      5       redis:3.0.5    RUNNING          RUNNING 25 seconds ago    node1
f503kbsdhv1syrd0k39dz9p12    redis      6       redis:3.0.5    RUNNING          RUNNING 23 seconds ago    node2
~~~


### Let's change the image from redis:3.0.5 to redis:3.0.6 (e.g. upgrade):
```bash
swarmctl service update redis --image redis:3.0.6
```
~~~yml
lhe5tau9qq5b5n7ipyphrf2um
~~~
```bash
swarmctl service inspect redis
```
~~~yml
ID                   : lhe5tau9qq5b5n7ipyphrf2um
Name                 : redis
Replicas             : 5/6
Update Status        
 State               : UPDATING
 Started             : 25 seconds ago
 Message             : update in progress
Template             
 Container           
  Image              : redis:3.0.6

Task ID                      Service    Slot    Image          Desired State    Last State                 Node
-------                      -------    ----    -----          -------------    ----------                 ----
0bsoidg9k4g6e5e3odlh4bkyt    redis      1       redis:3.0.6    RUNNING          RUNNING 6 seconds ago      node1
zlcg9vmf29rripfz2v0bdgsak    redis      2       redis:3.0.6    RUNNING          RUNNING 10 seconds ago     node1
s87qunoe7fegsa2vm8cdfh4z6    redis      3       redis:3.0.6    RUNNING          PREPARING 2 seconds ago    node2
8cv3e2u0u5mmbuelxjlrizgz8    redis      4       redis:3.0.6    RUNNING          RUNNING 19 seconds ago     node2
psswnmsli20luxfp8f5ih31jt    redis      5       redis:3.0.6    RUNNING          RUNNING 2 seconds ago      node1
i33388q5tgytkvo5rsquzi8pi    redis      6       redis:3.0.6    RUNNING          RUNNING 15 seconds ago     node2
~~~
```bash
swarmctl service inspect redis
```
~~~yml
ID                   : lhe5tau9qq5b5n7ipyphrf2um
Name                 : redis
Replicas             : 6/6
Update Status        
 State               : COMPLETED
 Started             : 41 seconds ago
 Completed           : 10 seconds ago
 Message             : update completed
Template             
 Container           
  Image              : redis:3.0.6

Task ID                      Service    Slot    Image          Desired State    Last State                Node
-------                      -------    ----    -----          -------------    ----------                ----
0bsoidg9k4g6e5e3odlh4bkyt    redis      1       redis:3.0.6    RUNNING          RUNNING 22 seconds ago    node1
zlcg9vmf29rripfz2v0bdgsak    redis      2       redis:3.0.6    RUNNING          RUNNING 26 seconds ago    node1
s87qunoe7fegsa2vm8cdfh4z6    redis      3       redis:3.0.6    RUNNING          RUNNING 15 seconds ago    node2
8cv3e2u0u5mmbuelxjlrizgz8    redis      4       redis:3.0.6    RUNNING          RUNNING 35 seconds ago    node2
psswnmsli20luxfp8f5ih31jt    redis      5       redis:3.0.6    RUNNING          RUNNING 18 seconds ago    node1
i33388q5tgytkvo5rsquzi8pi    redis      6       redis:3.0.6    RUNNING          RUNNING 31 seconds ago    node2
~~~


By default, all tasks are updated at the same time.

This behavior can be changed by defining update options.

For instance, in order to update tasks 2 at a time and wait at least 10 seconds between updates:

**` swarmctl service update redis --image redis:3.0.7 --update-parallelism 2 --update-delay 10s `**

**`watch -n1 "swarmctl service inspect redis" `** 

watch the update This will update 2 tasks, wait for them to become RUNNING, then wait an additional 10 seconds before moving to other tasks.

### Node Management:
SwarmKit monitors node health. In the case of node failures, it re-schedules tasks to other nodes.

An operator can manually define the Availability of a node and can Pause and Drain nodes.

Let's put node1 into maintenance mode:
```bash
swarmctl node drain node1

swarmctl node ls
```
~~~yml
ID                         Name   Membership  Status  Availability  Manager Status
--                         ----   ----------  ------  ------------  --------------
qmebebet000zd1sf9ew7gidac  node1  ACCEPTED    READY   DRAIN         REACHABLE *
yg3pt1q8mn6iylsm9b282v714  node2  ACCEPTED    READY   ACTIVE        

swarmctl service inspect redis
ID                   : lhe5tau9qq5b5n7ipyphrf2um
Name                 : redis
Replicas             : 6/6
Update Status        
 State               : COMPLETED
 Started             : 3 minutes ago
 Completed           : 3 minutes ago
 Message             : update completed
Template             
 Container           
  Image              : redis:3.0.6

Task ID                      Service    Slot    Image          Desired State    Last State                Node
-------                      -------    ----    -----          -------------    ----------                ----
vx9kjkzi7rvectego6rcw15qw    redis      1       redis:3.0.6    RUNNING          RUNNING 25 seconds ago    node2
0q9hnf95dd4lwn3dsd5qadwlc    redis      2       redis:3.0.6    RUNNING          RUNNING 26 seconds ago    node2
s87qunoe7fegsa2vm8cdfh4z6    redis      3       redis:3.0.6    RUNNING          RUNNING 3 minutes ago     node2
8cv3e2u0u5mmbuelxjlrizgz8    redis      4       redis:3.0.6    RUNNING          RUNNING 3 minutes ago     node2
p0qfq8b4hz1g0hv98ejmhkanc    redis      5       redis:3.0.6    RUNNING          RUNNING 25 seconds ago    node2
i33388q5tgytkvo5rsquzi8pi    redis      6       redis:3.0.6    RUNNING          RUNNING 3 minutes ago     node2
~~~

As you can see, every Task running on node1 was rebalanced to  node2 or nodeN by the reconciliation loop.


#### To activate drain node1:
```bash
swarmctl node activate node1

swarmctl node ls
```
~~~yml
ID                         Name   Membership  Status  Availability  Manager Status
--                         ----   ----------  ------  ------------  --------------
qmebebet000zd1sf9ew7gidac  node1  ACCEPTED    READY   ACTIVE        REACHABLE *
yg3pt1q8mn6iylsm9b282v714  node2  ACCEPTED    READY   ACTIVE        
~~~

To remove service:
```bash
swarmctl service remove  redis
```
~~~yml
redis
~~~
```bash
swarmctl node ls
```
~~~yml
ID                         Name   Membership  Status  Availability  Manager Status
--                         ----   ----------  ------  ------------  --------------
qmebebet000zd1sf9ew7gidac  node1  ACCEPTED    READY   DRAIN         REACHABLE *
yg3pt1q8mn6iylsm9b282v714  node2  ACCEPTED    READY   ACTIVE        
~~~
```bash
swarmctl service ls
```
~~~yml
ID  Name  Image  Replicas
--  ----  -----  --------
~~~

```bash
service docker stop 
sudo rm /var/lib/docker/network/files/local-kv.db 
cd /tmp
rm -rf node1
service docker start
```
again start  `swarmd -d /tmp/node1 --listen-control-api /tmp/node1/swarm.sock --hostname node1`


### Running background swarm:
```bash
nohup swarmd -d /tmp/node1 --listen-control-api /tmp/node1/swarm.sock --hostname node1 2>> /tmp/swarm.log &


nohup swarmd -d /tmp/node2 --hostname node2 --join-addr 172.16.0.167:4242 --join-token SWMTKN-1-374bf4b518vjczf4azobw3rs28i22ao7lh6562g3exmykgfol1-1bj3dzb62blwag1hmept85oum  2>> /tmp/swarm.log &

```
