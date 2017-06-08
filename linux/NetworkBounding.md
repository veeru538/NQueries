# 4 step Network bonding / teaming configuration in Linux
Network bonding or network teaming is binding two physical NIC (Network Interface Card) together to create once virtual NIC. 
This virtual NIC serves purpose of redundancy, fault tolerance and load balancing.

For application running on system its a one NIC they are talking to but on bare metal their requests are being served by two physical cards. 
Hence in case one physical card is failed or unplugged, another one still serves beneath virtual NIC and applications dont even know about failure. 


As of now with RHEL7, there are 7 types of NIC bond available :

1)    Bond 0 : Load balancing (round robin)
2)    Bond 1 : Active backup
3)    Bond 2 : Balance XOR
4)    Bond 3 : Broadcast
5)    Bond 4 : 802.3ad
6)    Bond 5 : Balance TLB
7)    Bond 6 : Balance ALB

We will see in detail about these types in another post. More commonly used are type 0 and type 1 bonds.Lets see step by step procedure to configure network bond in Linux.

For this tutorial, we will consider two ethernet cards eth1 and eth2 to configure bond. It is assumed that both are configured/connected to same network vlan.

### Step 1:

Configure both eth with master bond0 and slave as themselves. For that, open NIC configuration file located in /etc/sysconfig/network-scripts/ifcfg-eth1 & ifcfg-eth2 in vi and edit entries as highlighted below :

```
DEVICE=eth1
ONBOOT=yes
TYPE=Ethernet
BOOTPROTO=none
USERCTL=no
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=no
```
For eth2 file, DEVICE name will be eth2.

### Step 2:

Create bond0 device file under /etc/sysconfig/network-scripts/ifcfg-bond0. Add below details in it.
```
DEVICE=bond0
ONBOOT=yes
IPADDR=10.10.2.5
NETMASK=255.255.255.0
BONDING_OPTS="mode=1 miimon=100"
```
Here under bonding options we choose mode 1. If you choose to select any other mode out of 7 mentioned above, you need to specify here against mode

### Step 3:

Make sure bonding module is loaded into kernel. Add append lines in /etc/modprobe.conf file.

~~~yml
alias bond0 bonding
options bond0 mode=balance-alb miimon=100
~~~

Execute module with below command.

~~~yml
modprobe bonding
~~~

### Step 4:

Thats it. You are done with configuration. You need to restart networking service and you are good to go. Make sure your network manager service is not running.

```
service network restart
 
Shutting down interface bond0:                             [  OK  ]
Shutting down loopback interface:                          [  OK  ]
Bringing up loopback interface:                            [  OK  ]
Bringing up interface bond0:                               [  OK  ]
```

You can confirm your bond0 is up with mentioned IP in ip addr command output. Bonding mode can be verified with below command :

``` 
cat /proc/net/bonding/bond0
 
Bonding Mode: load balancing (round-robin)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 100
Down Delay (ms): 100
 
Slave Interface: eth0
MII Status: up
Link Failure Count: 0
Permanent HW addr: 00:0c:29:b6:be:32
 
Slave Interface: eth1
MII Status: up
Link Failure Count: 0
Permanent HW addr: 00:0c:29:b6:be:56
```

Even ifconfig command output will show you bond0 is up with mentioned IP address.

