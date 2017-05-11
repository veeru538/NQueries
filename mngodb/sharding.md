# MongoDB 3.4  sharding
MongoDB 3.4 was just released, and it has a lot of new features such as views, a better sharding algorithm, facet aggregation, numeric precision data type and more.

The procedure below covers upgrading from a previous version. The process for a fresh installation is slightly different. We don’t need to enable compatibility (as explained later).

Before upgrading, please be aware of a few details/requirements:

Minimum version. The minimum version required to upgrade to MongoDB 3.4 is 3.2.8.
Config servers. Config servers only work as replica sets. The previous method – instances as config servers – is no longer supported.
Driver. The driver must be ready for the new features (such as views and decimal data type). Please make sure that your driver version contains all the required settings.
How to upgrade a MongoDB config server instance (the previous config server architecture) to a replica set
Check if the balancer is running. If it is, disable the balancer process:
~~~yml
$ mongos
mongos> sh.isBalancerRunning()
false
mongos> sh.stopBalancer()
Waiting for active hosts...
Waiting for the balancer lock...
Waiting again for active hosts after balancer is off...
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 0 })

~~~

Connect to the first MongoDB config server and run the command below. Please replace the _id and members with your own settings:

```
configsvr> rs.initiate( {
  _id: "configRS",
  configsvr: true,
  version: 1,
  members: [ { _id: 0, host: "tests:27019" } ]
})
{ "ok" : 1 }
```


Get the storage engine name that is used by this instance. We will use this information later. If you are using MMAPv1, there are some additional commands to run:

```
db.serverStatus().storageEngine
{
"name" : "wiredTiger",
"supportsCommittedReads" : true,
"persistent" : true
}
```

Use the following command to read all the application startup parameters (informational only):

```
db.runCommand('getCmdLineOpts').parsed
{
 "net" : {
   "port" : 27019
   },
 "processManagement" : {
   "fork" : true
 },
 "sharding" : {
   "clusterRole" : "configsvr"
 },
 "storage" : {
   "dbPath" : "config1"
 },
 "systemLog" : {
   "destination" : "file",
   "path" : "config1/logconfig1.log"
 }
}
```

Restart the first MongoDB config instance with the –replSet configRS and  –configsvrMode=sccc parameters:

```
mongod --configsvr --replSet configRS --configsvrMode=sccc --storageEngine wiredTiger --port 27019 --dbpath config1 --logpath &nbsp;config1/logconfig1.log --fork
```
If using a configuration file, you must use the following parameters:

```
sharding:
  clusterRole: configsvr
  configsvrMode: sccc
replication:
  replSetName: csReplSet
net:
  port: <port>
storage:
  dbPath: <path>
  engine: <storageEngine>
```

After the first config server has been restarted, start two empty config servers with the parameters settings chosen before. If you are using MMAPv1, please start three instances instead of two.

```
./mongod --configsvr --replSet configRS --port 27025 --dbpath rsconfig1 --logpath rsconfig1/rsconfig1 --fork
./mongod --configsvr --replSet configRS --port 27026 --dbpath rsconfig2 --logpath rsconfig2/rsconfig2 --fork
```

(only if using MMAPv1) 
```
./mongod --configsvr --replSet configRS --port 27027 --dbpath rsconfig2 --logpath rsconfig3/rsconfig3 --fork
```
Connect to the first config server again and add the new instances to the replica set:

```
configRS:PRIMARY> rs.add({host : "tests:27025", priority : 0 , votes : 0})
{ "ok" : 1 }
configRS:PRIMARY> rs.add({host : "tests:27026", priority : 0 , votes : 0})
{ "ok" : 1 }
(only if using MMAPv1) rs.add({host : "tests:27027", priority : 0 , votes : 0})
{ "ok" : 1 }
```
Check the replication status. All the new instances must be in secondary status (part of the output has been omitted in the rs.status (below):

```
rs.status()
{
  "set" : "configRS",
  "date" : ISODate("2017-02-07T13:11:12.914Z"),
  "myState" : 1,
  "term" : NumberLong(1),
  "configsvr" : true,
  "heartbeatIntervalMillis" : NumberLong(2000),
  "members" : [ 
  {
     "_id" : 0,
     "name" : "tests:27019",
     "stateStr" : "PRIMARY",
     (...)
  },
  {
     "_id" : 1,
     "name" : "tests:27025",
     "stateStr" : "SECONDARY",
     (...)
  },
  {
    "_id" : 2,
    "name" : "tests:27026",
    "stateStr" : "SECONDARY",
    (...)
  },
  {
    "_id" : 3, // (will appear only if using MMAP)
    "name" : "tests:27027",
    "stateStr" : "SECONDARY",
    (...)
  },
],
  "ok" : 1
}
```

Once the replica set is up and running, please stop old config instances running without replica set. Also, add the votes and priority to all members of the replica set. If using MMAPv1, remove votes and priority from cfg.members[0];:

```
var cfg = rs.conf();
cfg.members[0].priority = 1; // 0 if using MMAP
cfg.members[1].priority = 1;
cfg.members[2].priority = 1;
cfg.members[0].votes = 1; // 0 if using MMAP
cfg.members[1].votes = 1;
cfg.members[2].votes = 1;
```

(Only if the first config server is using mmap)

```
cfg.members[3].priority = 1;
cfg.members[3].votes = 1;
rs.reconfig(cfg);
```

Restart the mongos service using the new replica set repl/hostnames:port and the —configdb parameter:

```
./mongos --configdb configRS/tests:27019,tests:27025,tests:27026 --logpath mongos.log --fork
```
Perform a stepDown() and shutDown() in the first server. If this server is using MMAPv1, remove it from the replica set using rs.remove(). If it is using wiredTiger, restart without configsvrMode=sccc in the parameters.
At this point, you have an instance that has been correctly migrated to a replica set.

Please enable the balancer again (use a 3.4 client, or it will fail):
```
sh.startBalancer()
```
How to upgrade config instances server to version 3.4

Please follow these instructions to upgrade a shard to the 3.4 version.

As we already have the configs correctly running as replica sets, we need to upgrade the binaries versions. It is easy to do. Stop and replace the secondaries binaries versions:

Connect to the mongos and stop the balancer.
Stop the secondary, replace the old binary with the new version, and start the service.
When the primary is the only one running with version 3.2, run rs.stepDown()  and force this instance to become secondary.
Replace these (now) secondary binaries with the 3.4 version.
For the shards, add a new parameter before restarting the process in the 3.4 version:

Stop the secondaries, replace the binaries and start it with the –shardsvr parameter.
If using a config file, add (this is a new parameter in this version):

```
sharding:
    clusterRole: shardsvr
```
Step down the primary and perform this same process on it.
After all the shards binaries have been changed to the 3.4 version, upgrade the mongos binaries.
Stop mongos and replace this executable with the 3.4 version.
At this point, all the shards are running the 3.4 version, but by default the new features in MonogDB 3.4 are disabled. If we try to use a decimal data type, mongos reports the following:

```
WriteResult({
  "nInserted" : 0,
  "writeError" : {
  "code" : 22,
  "errmsg" : "Cannot use decimal BSON type when the featureCompatibilityVersion is 3.2. See http://dochub.mongodb.org/core/3.4-feature-compatibility."
 }
})

```
In order to enable MongoDB 3.4 features we need run the following in the mongos:

```
db.adminCommand( { setFeatureCompatibilityVersion: "3.4" } )
{ "ok" : 1 }
mongos> use testing
switched to db testing
mongos> db.foo.insert({x : NumberDecimal(10.2)})
WriteResult({ "nInserted" : 1 })
```

After changing the featureCompatibility to “3.4,”  all the new MongoDB 3.4 features will be available.

I hope this tutorial was useful in explaining upgrading to Percona Server for MongoDB 3.4.
