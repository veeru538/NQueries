# LVM(Logical Volume Creation)

1. Physical Volume (pvcreate, pvdisplay, pvchange, pvmove)
2. Volume group (vgcreate, vgdisplay, vgscan, vgextend, vgreduce, vgexport, vgimport, vgcfgbackup, vgcfgrestore, vgchange, vgremove, vgsync)
3. Logical Volume (lvcreate, lvdisplay, lvremove, lvextend, lvreduce, lvchange, lvsync, lvlnboot)


## Part 1 : Physical Volume (pvcreate, pvdisplay, pvchange, pvmove)
Physical volume is raw disk presented to operating system. It can be local disk, LUN from remote storage, disk from local disk array etc. All storage disks from these types are formatted as physical volumes under LVM so that those can be used in definite file systems.

To check list of disks using below command:
lsblk


### pvcreate
Now that you identified new disk presented to server lets say /dev/rdisk/disk3 for example, you can run pvcreate command to create physical volume out of it.

```
pvcreate /dev/rdisk/disk3
Physical volume "/dev/rdisk/disk3" has been successfully created.
```

### pvdisplay

Now, PV has been created one can see its details with pvdisplay command.
~~~yml
pvdisplay /dev/disk/disk3
--- Physical volumes ---
PV Name                     /dev/disk/disk3
VG Name                     
PV Status                   available
Allocatable                 yes
VGDA                        2
Cur LV                      0
PE Size (Mbytes)            32
Total PE                    1557
Free PE                     1557
Allocated PE                0
Stale PE                    0
IO Timeout (Seconds)        default
Autoswitch                  On
Proactive Polling           On
~~~

Here many fields are self explanatory. Refer LVM legends to have better understanding.  From this output you can also calculate disk size. Total PE x PE size = Disk size (available to use i.e. after formatting space loss)

If you want to drill down PE details i.e. which PE is serving which LV then -v option can be used. This is helpful when disk has bad sectors. Below is output in which PV is part of volume group already.

```
pvdisplay -v /dev/disk/disk3
--- Physical volumes ---
PV Name                     /dev/disk/disk3
VG Name                     /dev/vg01
PV Status                   available
Allocatable                 yes
VGDA                        2
Cur LV                      1
PE Size (Mbytes)            32
Total PE                    1557
Free PE                     0
Allocated PE                1557
Stale PE                    0
IO Timeout (Seconds)        default
Autoswitch                  On
Proactive Polling           On
 
   --- Distribution of physical volume ---
   LV Name                 LE of LV  PE for LV
   /dev/vg01/lvol1         1557      1557
 
   --- Physical extents ---
   PE    Status   LV                      LE
   00000 current  /dev/vg01/lvol1         00000
   00001 current  /dev/vg01/lvol1         00001
   00002 current  /dev/vg01/lvol1         00002
   00003 current  /dev/vg01/lvol1         00003
   00004 current  /dev/vg01/lvol1         00004
   00005 current  /dev/vg01/lvol1         00005
   00006 current  /dev/vg01/lvol1         00006
   00007 current  /dev/vg01/lvol1         00007
   00008 current  /dev/vg01/lvol1         00008
   00009 current  /dev/vg01/lvol1         00009
   00010 current  /dev/vg01/lvol1         00010
   00011 current  /dev/vg01/lvol1         00011
   00012 current  /dev/vg01/lvol1         00012
----- output truncated -----
``` 
Here you can see how PV is distributed among different LV. and furthermore it gives you table mapping of PE to LE! There are numerous options can be used with this command but normally -v is used commonly.

pvchange

pvchange used for changing attributes of physical volume. There are 6 options switch which can be used with this command. Lets see them one by one.

-a

This switch used for availability attribute of PV. This should be accompany with y or n argument. y making PV available and n making it unavailable. You can see example below with n argument which turn PV unavailable which can be verified in pvdisplay output.
Shell

# pvchange -a n /dev/disk/disk3
Physical volume "/dev/disk/disk3" has been successfully changed.

# pvdisplay /dev/disk/disk3
--- Physical volumes ---
PV Name                     /dev/disk/disk3
VG Name                     /dev/vg01
PV Status                   unavailable
Allocatable                 yes
VGDA                        2
Cur LV                      2
PE Size (Mbytes)            8
----- output truncated -----
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
	
 
# pvchange -a n /dev/disk/disk3
Physical volume "/dev/disk/disk3" has been successfully changed.
 
# pvdisplay /dev/disk/disk3
--- Physical volumes ---
PV Name                     /dev/disk/disk3
VG Name                     /dev/vg01
PV Status                   unavailable
Allocatable                 yes
VGDA                        2
Cur LV                      2
PE Size (Mbytes)            8
----- output truncated -----
 

-S

This switch enables or disables autoswitching. It also has y or n argument. If activated it makes current access path to switch on better available path. If de activated, path switching happens only if current access path goes to un-available state (due to hardware, cable failure). This applies to disks with multi paths only.

-t

Its a timeout value. This switch should be supplied with number of seconds. This value determines if IO timeout happens and problem can be declared for IO on particular PV. Useful in clustering.
Shell

# pvchange -t 90 /dev/disk/disk3
Physical volume "/dev/disk/disk3" has been successfully changed.
1
2
3
4
	
 
# pvchange -t 90 /dev/disk/disk3
Physical volume "/dev/disk/disk3" has been successfully changed.
 

-x

It enables or disables extensibility of PV. This switch has y and n arguments which enables admin to add/restrict physical extents to PV. Refer LVM legends to have better understanding.

-s

Immediately begin accessing the associated physical volume.

-z

Defines PV is spare or regular. This option has significance in mirroing only.

Command : pvmove

As name suggests, it used to move data (in LVM term PE) from one PV to another PV. Command essential moves PEs and hence data within from source to destination PV. If destination PV is not specified then all available PV’s in current volume group are considered for move operation. Command decides himself best suited PV for move operation so that allocation policies can be met correctly.

Also if command is supplied with list of PV names then first PV is always considered as source and rest all PVs are considered as destination.
Syntex
Shell

# pvmove /dev/disk/disk1 /dev/disk/disk2 /dev/disk/disk3
1
2
3
	
 
# pvmove /dev/disk/disk1 /dev/disk/disk2 /dev/disk/disk3
 

In above example PEs will be moved from disk 1 to disk2 & disk3.

You can also move data for particular lvol to new PV.
Shell

# pvmove -n /dev/vg01/lvol2 /dev/disk/disk1 /dev/disk/disk2
Transferring logical extents of logical volume "/dev/vg01/lvol2"...
Physical volume "/dev/disk/disk1" has been successfully moved.
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf

# /usr/sbin/lvdisplay -v /dev/vg01/lvol2

----- output clipped -----

   --- Distribution of logical volume ---
   PV Name                 LE on PV  PE on PV
   /dev/disk/disk2         1557      1557

   --- Logical extents ---
   LE    PV1                     PE1   Status 1
   00000 /dev/disk/disk2         00000 current
   00001 /dev/disk/disk2         00001 current
   00002 /dev/disk/disk2         00002 current
   00003 /dev/disk/disk2         00003 current
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
	
 
# pvmove -n /dev/vg01/lvol2 /dev/disk/disk1 /dev/disk/disk2
Transferring logical extents of logical volume "/dev/vg01/lvol2"...
Physical volume "/dev/disk/disk1" has been successfully moved.
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf
 
# /usr/sbin/lvdisplay -v /dev/vg01/lvol2
 
----- output clipped -----
 
   --- Distribution of logical volume ---
   PV Name                 LE on PV  PE on PV
   /dev/disk/disk2         1557      1557
 
   --- Logical extents ---
   LE    PV1                     PE1   Status 1
   00000 /dev/disk/disk2         00000 current
   00001 /dev/disk/disk2         00001 current
   00002 /dev/disk/disk2         00002 current
   00003 /dev/disk/disk2         00003 current
 

In above example PE belonging to lvol2 will be moved from disk1 to disk2. PE distribution can be confirmed via lvdisplay output.



## Part 2 : Volume group (vgcreate, vgdisplay, vgscan, vgextend, vgreduce, vgexport, vgimport, vgcfgbackup, vgcfgrestore, vgchange, vgremove, vgsync)
## Part 3 : Logical Volume (lvcreate, lvdisplay, lvremove, lvextend, lvreduce, lvchange, lvsync, lvlnboot)
