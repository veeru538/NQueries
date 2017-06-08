# Difference between tmpfs and swap
On social share of our last post about RAM disk in Linux we got a comment “what is the difference between RAM disk and SWAP?” I will try to explain how swap and RAM disk i.e. tmpfs/ramfs is different and how they work.
### What is tmpfs?
Tmpfs also mounted as shared memory /dev/shm. tmpfs is portion of a virtual memory mounted as a file system which helps to speed up applications. It normally being used to transfer data between programs. It appears as a file system but it does not use persistent device such as hard disk. Instead it uses virtual memory (portion of a RAM).

Thats why if you create any file in tmpfs its not created on your system disks but in your memory. Whenever you un-mount tmpfs, everything within is lost. Its volatile storage. Even if you add entry of tmpfs to re-mount at boot, it will be mounted blank. Data does not persist over reboots or shutdowns in tmpfs.
### What is SWAP?

swap is portion of your hard disks used to extend RAM. Its roughly extended RAM by use of persistent storage device. swap only comes in action once your RAM (physical memory) is full. Normal thumb rule is size of swap should be double of your physical ram size. But this changes depends on conditions and system you have. Read how to create extra swap here.

Even if uses persistant devices, it still is a volatile memory. It does not hold data over reboot or shutdowns. Since it plays role of RAM, its characteristics are still of ram even if uses hard disks.
Difference between tmpfs and swap

    tmpfs uses memory while as swap uses persistent storage devices.
    tmpfs can be viewed as file system in df output whereas swap dont
    swap has general size recommendations, tmpsfs not. tmpfs size varies on system purpose.
    tmpfs makes applications fasters on loaded systems. swap helps system breath in memory full situations.
    swap full indicates system heavily loaded, degraded performance and may crash. tmpfs being full not necessarily means heavy load or prone to crash.
    tmpfs is enhancement where as swap is must have feature!



# How to create RAM disk in Linux
### What is RAM disk?
Roghuly RAM disk can be termed as potion of your RAM mounted as a directory. It uses tmpfs or ramfs. Refer below table for difference between ramfs and tmpfs.


### Difference between ramfs and tmpfs

| ramfs         | tmpfs         |
| ------------- |:-------------:|
| Old type      | New type and replacing ramfs now |
| Can not be limited in size     | New type and replacing ramfs now |
| Since can not be limited may lead to system crash	Once limit reached, disk full error written.     | No system crash issue. |
| Entry is not visible in 'df' output. Need to calculate by using 'Cached' number in 'free' output.     | Can be seen in 'df' command output |
| Work mechanism as file system cache     | Work mechanism is as partition of physical disk |


RAM disk is very high speed, high performance and almost zero latency area to store application files. Due to its performance oriented nature, its mostly used for temporary data like caching application files.
How to create RAM disk?

RAM disk can be created in simple two steps. One is to create directory on which it should be mounted and second step is to mount it on that directory using specific FS type. Make sure you have enough free RAM on system so that portion of it can be used in RAM disk. You can check it using free command.

Lets create directory /mnt/ram_disk and mount RAM disk on it.

~~~/bin/bash
mkdir /mnt/ram_disk
mount -t tmpfs -o size=1024m new_ram_disk /mnt/ram_disk
~~~ 

In above mount command, -t should be followed by tmpfs or ramfs type. For ramfs, size is starting size of RAM disk since ramfs has limitless size. Size followed by name of the disk (of your choice ex. new_ram_disk). You can verify if its mounted properly using df command.
Shell

``` 
df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1      5.8G  2.9G  2.7G  53% /
tmpfs           498M     0  498M   0% /dev/shm
new_ram_disk          1.0G     0  1.0G   0% /mnt/ram_disk
```
 

You can see newly created tmpfs of 1GB size is mounted on /mnt/ram_disk (highlighted above).

You can add below entry in /etc/fstab as well to persist it over reboots as well. But keep in mind that data within RAM disk flushes for each reboot since its backed memory is volatile.

~~~/bin/bash
new_ram_disk    /mnt/ram_disk   tmpfs    nodev,nosuid,noexec,nodiratime,size=1024M   0 0
~~~

 
