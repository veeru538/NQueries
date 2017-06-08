# LVM(Logical Volume Creation)

	1. Physical Volume - pvcreate, pvdisplay, pvchange, pvmove
	2. Volume group - vgcreate, vgdisplay, vgscan, vgextend, vgreduce, vgexport, vgimport, vgcfgbackup, vgcfgrestore, vgchange, vgremove, vgsync 
	3. Logical Volume - lvcreate, lvdisplay, lvremove, lvextend, lvreduce, lvchange, lvsync, lvlnboot


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
```
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
```

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

### pvchange

pvchange used for changing attributes of physical volume. There are 6 options switch which can be used with this command. Lets see them one by one.

-a

This switch used for availability attribute of PV. This should be accompany with y or n argument. y making PV available and n making it unavailable. You can see example below with n argument which turn PV unavailable which can be verified in pvdisplay output.
~~~/bin/bash 
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
~~~ 

-S

This switch enables or disables autoswitching. It also has y or n argument. If activated it makes current access path to switch on better available path. If de activated, path switching happens only if current access path goes to un-available state (due to hardware, cable failure). This applies to disks with multi paths only.

-t

Its a timeout value. This switch should be supplied with number of seconds. This value determines if IO timeout happens and problem can be declared for IO on particular PV. Useful in clustering.

```
pvchange -t 90 /dev/disk/disk3
Physical volume "/dev/disk/disk3" has been successfully changed.
```

-x

It enables or disables extensibility of PV. This switch has y and n arguments which enables admin to add/restrict physical extents to PV. Refer LVM legends to have better understanding.

-s

Immediately begin accessing the associated physical volume.

-z

Defines PV is spare or regular. This option has significance in mirroing only.

### pvmove

As name suggests, it used to move data (in LVM term PE) from one PV to another PV. Command essential moves PEs and hence data within from source to destination PV. If destination PV is not specified then all available PV’s in current volume group are considered for move operation. Command decides himself best suited PV for move operation so that allocation policies can be met correctly.

Also if command is supplied with list of PV names then first PV is always considered as source and rest all PVs are considered as destination.

```
pvmove /dev/disk/disk1 /dev/disk/disk2 /dev/disk/disk3
``` 

In above example PEs will be moved from disk 1 to disk2 & disk3.

You can also move data for particular lvol to new PV.
```
pvmove -n /dev/vg01/lvol2 /dev/disk/disk1 /dev/disk/disk2
Transferring logical extents of logical volume "/dev/vg01/lvol2"...
Physical volume "/dev/disk/disk1" has been successfully moved.
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf

lvdisplay -v /dev/vg01/lvol2

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
```

In above example PE belonging to lvol2 will be moved from disk1 to disk2. PE distribution can be confirmed via lvdisplay output.




## Part 2 : Volume group (vgcreate, vgdisplay, vgscan, vgextend, vgreduce, vgexport, vgimport, vgcfgbackup, vgcfgrestore, vgchange, vgremove, vgsync)

### vgcreate

As we all know volume group i.e. VG is a collection if PV. Refer LVM legands for better understanding. VG creation is the next necessary step after PV creation to use disk space in mount points. Since in Unix everything is a file, to manage VG, kernel creates /dev/vgname/group file. This file represents VG in kernel. Let us see how to create VG from bunch of available PV. Post Mar 2008 releases of HPUX creates group file automatically with vgcreate command. But we will see here to how to make one.

First create a directory with desired name of your VG for ex. lets say testvg is our VG name. After that using mknod command create special group file. In command you need to supply major and minor numbers.
Shell
```
mkdir /dev/testvg
mknod /dev/testvg/group c major 0xminor

major number : 64 for version 1.0 VG and 128 for version 2.0 VG
minor number : Its hexadecimal number. 0xnn0000 for v1.0 and 0x0000 for v2.0 where nn/nnn is unique number.
```
Now that special file is generated go ahead to create VG

```
vgcreate /dev/testvg /dev/disk/disk3 /dev/disk/disk4
Volume group "/dev/testvg" has been successfully created.
Volume Group configuration for /dev/testvg has been saved in /etc/lvmconf/testvg.conf
```
 

vgcreate can be run with single PV argument too. This command has several options as below :

    -V 1.0            To decide version.
    -s PE_size    Size of PE to be used in MB. Default is 4MB
    -e max_PE   Max PE per PV. Default is 1016
    -l max_lv      Max number of LV per VG. Default is 255
    -p max_pv   Max number of PV per VG. Default is 255
    -S vg_size    Max future size of VG. Only in v2.0.

 

Above options can be supplied to command with proper values but its not mandatory. If not supplied then their respective values will be taken into consideration while creating VG. Changes in parameters with these options can be seen in vgdisplay output which you can see in coming section.

### vgdisplay

Same as pvdisplay command, vgdisplay is used to view VG details. vgdisplay command can be run with -v option to get more detailed output. If run without -v option then it will show output like below but PV details portion wont be there.
	
``` 
# vgdisplay -v vg_new
--- Volume groups ---
VG Name                     /dev/testvg
VG Write Access             read/write     
VG Status                   available                 
Max LV                      255    
Cur LV                      0      
Open LV                     0      
Max PV                      16     
Cur PV                      2      
Act PV                      2      
Max PE per PV               6000         
VGDA                        2   
PE Size (Mbytes)            32              
Total PE                    26    
Alloc PE                    0       
Free PE                     26    
Total PVG                   0        
Total Spare PVs             0              
Total Spare PVs in use      0 
 
   --- Physical volumes ---
   PV Name                     /dev/disk/disk3
   PV Status                   available                
   Total PE                    10       
   Free PE                     10       
   Autoswitch                  On        
 
   PV Name                     /dev/disk/disk4
   PV Status                   available                
   Total PE                    10       
   Free PE                     10       
   Autoswitch                  On
 ```

In above output, you can see PE Size (Mbytes), Max PE per PV, Max LV, Max PV fields which can be altered with options -s, -e, -l, -p arguments in vgcreate command we seen above.  The above output is pretty self explanatory. It also shows PV details which are part of this VG.

### vgscan

This command used for scanning all available PVs to get information of VG and related LV’s and rebuild /etc/lvmtab file. This command used below options :

    -a Scan all available PV paths.
    -B Write all persistent and legacy paths to /etc/lvmtab file
    -f vg_name Force to update entries of existing VG in /etc/lvmtab with updated info if any
    -k Avoid probing disks. Build file from kernel known structure.
    -N Populate file with persistent DSF names
    -p Preview mode only. It wont update lvmtab file
    -v verbose mode. Prints messages in console.

```
# vgscan -v
/dev/vg00
/dev/disk/disk1_p2
/dev/disk/disk2_p2
```
Scan of Physical Volumes Complete.
 

Normally we use vgscan -v only. After this command you can verify timestamp of /etc/lvmtab file to verify its been updated.

This concludes first 3 commands of VG. In next post we will see how to extend volume group, how to reduce volume group and how to export/import volume group.

### vgextend

Whenever there is need of extra space on mount points and new disks are added to system, related volume groups needs to accommodate this new disks. Lets say vg01 is existing VG and new disk5 is added to server then you need to use vgextend command to add this new disks into vg01. This will in turns add new free PE in existing VG vg01 and hence free space will hiked in VG.

To add disks into existing VG, you need to initialize it first with pvcreate command (see part one tutorial for pvcreate). Once done disk can be used in vgextend like below :

```
vgextend /dev/vg01 /dev/disk/disk5
Volume group "/dev/vg01" has been successfully extended.
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf
``` 

After successful completion you can verify that new PV is added to VG by looking at vgdisplay -v vg01 output.

This command can be used with below options :

    -f                       Forcefully extend
    -x                      Set allocation permissions
    -z sparepv      Should be used ti mirroring to define spare PV
    -g pvg_name Extend with physical volume group specified.

### vgreduce

Its just reverse of above command. It reduces VG by removing specified PV in command. Its bit risky to use hence extra precaution needed. You should make sure that no PE from targeted PV is being used in any LV of VG. This an be verified by lvdisplay and vgdisplay command. If no then you can use pvmove, lvremove commands to move/delete data. Once you are sure that no PE from targeted PV is being used i.e. all PEs are free form this PV then run below command to remove PV from VG.
```
vgreduce /dev/vg01 /dev/disk/disk5
Volume group "/dev/vg01" has been successfully reduced.
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf
```
If you are using multiple PV links then you should remove all PV paths from VG using above command.

This commands allows two options :

    -f                  Forcefully remove PV
    -l pv_path Remove only specified particular PV path from VG

### vgexport

Using vgexport one can export all VG related information from disks into map file leaving data on disks intact. With help of this map file, same VG can be imported on another system provided all related disks are presented to this another system.

Please note that, running this commands removes all VG related files from /dev directory and all its related entries in /etc/lvmtab file. This means after running vgexport, that particular VG is turns into non-existent VG for system.

Before running vgexport, one has to make sure that no one is using data on LVOLs from this VG. This can be verified using fuser -cu /mount_point command. Also, VG shouldn’t be active at the time of export hence you needs to de-activate VG using vgchange.
```
vgchange -a n /dev/vg01
Volume group "/dev/vg01" has been successfully changed.

vgexport -v -m /tmp/vg01.map vg01
Beginning the export process on Volume Group "/dev/vg01". 
/dev/dsk/c0t1d0 vgexport:Volume Group “/dev/vg01” has been successfully removed.
``` 

This commands allows below options :

    -p        Preview only. It wont actually exports Vg but creates map file. Useful for live VG
    -v        Verbose mode
    -f file Writes all PV paths to file. This file can be used as input to vgimport.

### vgimport

This command imports VG which been exported using above command. Map file extracted from vgimport command should be supplied as argument to vgimport. In case map file is not available, it will still try to import VG by reading LVM information lies of physical disks. In this case, list of all PV should be supplied.

After vgimport successfully imports VG, note that its in inactive state. You need to activate VG to use it on system. Also, its recommended to take a configuration backup once you import new VG on system

```
# vgimport -v -m /tmp/vg01.map /dev/vg01 list_of_disk
vgimport: Volume group “/dev/vg01” has been successfully created.
Warning: A backup of this volume group may not exist on this machine.
```
Please remember to take a backup using the vgcfgbackup command after activating the volume group.
```
vgchange -a y vg01
Volume group “/dev/vg01” has been successfully changed.

vgcfgbackup /dev/vg01
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf
```
This command can be supplied with below options :

    -N Import with persistant device file names
    -s       Scan each disks
    -p      Preview only
    -v       Verbose mode
    -f file Import PV paths from file

This concludes second post of part two. In next post we will be seeing how to backup and restore volume group configurations and how to change volume group states

### vgcfgbackup

As the command reads, it takes volume group configuration backup into a disk file residing under /etc/lvmconf directory in file /etc/lvmconf/vg_name.conf. It reads the LVM header details from system area of the disk and copies it to file. This file helps you to restore configuration on newly added disk in place of old disk which may have got corrupted or failed.

It is recommended to have this backup taken after every LVM level change. By default all LVM commands altering LVM details are designed to take this backup automatically hence manually running command is not necessary.

To take manual configuration backup use command as below :
``` 
# vgcfgbackup /dev/vg01
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf
```

This needs to run for every volume group available on system. This command has couple of options :

    -f file Save backup in file specified rather than default location
    -u       Only updates extra PV which are added to VG after last backup. Only new PVs required to be online on system. IF -u not used all PV under VG should be online while running command.

### vgcfgrestore

This command restores the VG configuration taken using above command. Command mainely needs 2 argument volume group name of which configuration neds to be restored and a PV name on which configuration needs to be restored. There are two things which needs to keep in mind.

If PV on which we are restoring the backup is part of mirror copy, then it should be deactivated first. Then restore backup and then reactivate it. Command sequence will be :
```
# pvchange -a n /dev/disk/disk5
Physical volume "/dev/disk/disk5" has been successfully changed.

# vgcfgrestore -n /dev/vg01 /dev/rdisk/disk5
Volume Group configuration has been restored to /dev/rdisk/disk5

# pvchange -a y /dev/disk/disk5
Physical volume "/dev/disk/disk5" has been successfully changed.
```

If PV is not a part of mirror then you should deactivate volume group first then restore the backup and then activate volume group again. Command sequence will be :
```
# vgchange -a n /dev/vg01
Volume group "/dev/vg01" has been successfully changed.

# vgcfgrestore -n /dev/vg01 /dev/rdisk/disk5
Volume Group configuration has been restored to /dev/rdisk/disk5

# vgchange -a y /dev/vg01
Volume group "/dev/vg01" has been successfully changed.
```

This command has several options which can be used with it :

    -l                   List configuration only
    -F                  Forcefully restore
    -f file             Restore configuration from specified file not from default backup location
    -v                   Verbose mdoe
    -R                  Forcefully restore even if there is mismatch between PV in kernel and in config.
    -o old_path Restore config saved for old PV (path supplied) to new PV mentioned in command.

### vgchange

This command used to make volume group active or inactive. Activated volume group simply means its available for use. There are different mode of volume groups in which they can be activated. They can be listed as below :

    Normal availability mode
    Cluster aware mode
    Shareable mode
    Quoram requirement mode

vgchange command can be used with specified options and their values to activate or deactivate volume group. For example to normally activate it should be supplied with argument -a nd its value y. See below output to activate and then deactivate volume group

``` 
# vgchange -a y vg01
Volume group “/dev/vg014” has been successfully changed.
 
# vgchange -a n vg01
Volume group “/dev/vg014” has been successfully changed.
```

Above stated modes can be activated/deactivated using below options :

    -a y/n Normal availability mode
    -c y/n Cluster aware mode
    -S y/n Shareable mode
    -q y/n Quoram requirement mode
    -p Only activate if all related PVs are online
    -s Disable stale PE sync
    -x Cross activate shareable volume group

This concludes third post of part two related to volume group. In next and last post of part two, we are covering how to remove volume group from system and how to sync stale PE within volume group.

### vgremove

vgremove commands used to remove volume group from system. But this is destroying command since it requires removal of all LV, PV in VG. It is always recommended to use vgexport instead of vgremove. Since vgexport also removes VG information from system but keeps it untouched on PV so that same PV can be imported to new VG on new/same system using vgimport.
Safe removal of VG can be done with below steps :

        Backup all user data in that VG
        Get information of all LV and PV in that VG using vgdisplay -v command
        Make sure no LV is in use using fuser -cu /mount_point command
```
fuser -cu /data
/data:   223412c(user1)
```      
Unmount mount points of related LV
```
umount /data
```      
Remove all LVs with lvremove /dev/vg01/lvol-name command
```
lvremove /dev/vg01/lvol1
The logical volume "/dev/vg01/lvol1" is not empty;
do you really want to delete the logical volume (y/n) : y
Logical volume "/dev/vg01/lvol1" has been successfully removed.
Volume Group configuration for /dev/vg03 has been saved in /etc/lvmconf/vg01.conf
```

Remove all PVs in VG except any one with vgreduce /dev/vg01 /dev/disk/diskX command
```
vgreduce /dev/vg01 /dev/disk/disk4
Volume group "/dev/vg01" has been successfully reduced.
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf
```         
Finally use vgremove command to remove VG from system
```
vgremove /dev/vg01
Volume group "/dev/vg01" has been successfully removed
```           
Remove related group files from system using rm /dev/vg01 command

### vgsync

This command used to sync stale LE of LV mirrors in current VG. This used in mirroring only. Once can observe output of vgdisplay -v and confirm if there are any stale LE in current VG. If you found stale LE then you can synchronize them using this command.

```
# vgsync /dev/vg01
Resynchronized logical volume "/dev/vg01/lvol01".
Resynchronized logical volume "/dev/vg01/lvol02".
Resynchronized volume group "/dev/vg01".
```
 


## Part 3 : Logical Volume (lvcreate, lvdisplay, lvremove, lvextend, lvreduce, lvchange, lvsync, lvlnboot)

### lvcreate

This command used to create new logical volume. Logical volumes are mounted on directories as a mount point. So logical volume size is the size you want for mount point. Use command like below :

```	
lvcreate -L 1024 /dev/vg01
Logical volume "/dev/vg01/lvol1" has been successfully created with character device "/dev/vg01/rlvol1"
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf
``` 

In above command you need to supply size in MB (1 GB in above example) to -L argument and volume group name in which you need to create that LV. If no name suggested in command then by default command creates LV with name /dev/vg01/lvolX (X is next avaibale number).

This command supports below options –

    -l  : Number of LEs
    -n : LV Name

Created LV details can be seen using command lvdisplay.

### lvdisplay

We seen above how to create LV, now we will see how to view details of it. This command is same as pvdisplay for PV and vgdisplay for VG. It shows you details like name, volume group it belongs to, size, permission, status, allocation policy etc.
	
``` 
# lvdisplay /dev/vg01/lvol1
--- Logical volumes ---
LV Name                     /dev/vg01/lvol1
VG Name                     /dev/vg01
LV Permission               read/write
LV Status                   available/syncd
Mirror copies               0
Consistency Recovery        MWC
Schedule                    parallel
LV Size (Mbytes)            1024
Current LE                  32
Allocated PE                32
Stripes                     0
Stripe Size (Kbytes)        0
Bad block                   on
Allocation                  strict
IO Timeout (Seconds)        default
``` 

More detailed output can be obtained with -v option. In this detailed outpout you can get the LE details where they resides and LV distribution across disks.
	
``` 
# lvdisplay -v /dev/vg01/lvol1
--- Logical volumes ---
LV Name                     /dev/vg01/lvol1
VG Name                     /dev/vg01
 
----- Output clipped ----
 
   --- Distribution of logical volume ---
   PV Name                 LE on PV  PE on PV
   /dev/disk/disk22        32        32
 
   --- Logical extents ---
   LE    PV1                     PE1   Status 1
   00000 /dev/disk/disk22        00000 current
   00001 /dev/disk/disk22        00001 current
   00002 /dev/disk/disk22        00002 current
   00003 /dev/disk/disk22        00003 current
   00004 /dev/disk/disk22        00004 current
   00005 /dev/disk/disk22        00005 current
   00006 /dev/disk/disk22        00006 current
   00007 /dev/disk/disk22        00007 current
   00008 /dev/disk/disk22        00008 current
   00009 /dev/disk/disk22        00009 current
   00010 /dev/disk/disk22        00010 current
   00011 /dev/disk/disk22        00011 current
   00012 /dev/disk/disk22        00012 current
   00013 /dev/disk/disk22        00013 current
   00014 /dev/disk/disk22        00014 current
 
----- output truncated -----
``` 
### lvremove

Removing a logical volume is data destroying task. Make sure you take backup of data within mount point then empty it and stop all user/app access to it. If LV is not empty then command will prompt you for confirmation to proceed. 

```
lvremove /dev/vg01/lvol1
The logical volume "/dev/vg01/lvol1" is not empty;
do you really want to delete the logical volume (y/n) : y
Logical volume "/dev/vg01/lvol1" has been successfully removed.
Volume Group configuration for /dev/vg03 has been saved in /etc/lvmconf/vg01.conf
```

Once lvol is delete its number is again available for next new lvol which is being created in same VG. All PE assigned to this LV will be released as free PE and hence free space in VG will increase.

### lvextend

To extend logical volume, you should have enough free space within that VG. Command syntex is pretty much similar to lvcreate command for size. Only thing is you need to supply final required size in command. For example current LV size is 1GB and you want to extend it with 2GB. Then you need to give final 3GB size in command argument.
``` 
# lvextend -L 3072 /dev/vg01/lvol1
Logical volume "/dev/vg01/lvol1" has been successfully extended.
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf
``` 

Another important option is of mirror copies. It plays vital role in root disk mirroring. -m is the option with number of mirror copies as a argument.
	
``` 
# lvextend -m 1 /dev/vg00/lvol1 /dev/disk/disk2_p2
The newly allocated mirrors are now being synchronized. This operation will
take some time. Please wait ....
Logical volume "/dev/vg00/lvol1" has been successfully extended.
Volume Group configuration for /dev/vg00 has been saved in /etc/lvmconf/vg00.conf
```

### lvreduce

This command used for decreasing number of mirror copies or decreasing size of LV. This is data destroying command. Hence make sure you have datat of related file system backed up first. The size and mirror copy options are works same for this command as well . -L for LE_reduce_size, -l number of LE to be reduces and -m is number of copies to be reduced.

``` 
lvreduce -L 500 /dev/vg01/lvol1
When a logical colume is reduced useful data might get lost;
do you really want the command to proceed (y/n) : y
Logical volume "/dev/vg01/lvol1" has been successfully reduced.
Volume Group configuration for /dev/vg01 has been saved in /etc/lvmconf/vg01.conf
``` 

While reducing mirror copies if one of the PV is failed or missing then command wont run successful. you need to supply -k option which will proceed to remove mirror in case PV is missing.

### lvchange
This command is used for changing characteristics of LV. There are numerous options which can be used.

    -a y/n Activate or deactivate LV
    -C y/n Change contiguous allocation policy
    -D y/n Change distributed allocation policy
    -p w/r Set permission
    -t timeout Set timeput in seconds
    -M y/n Change mirror write cache flag
    -d p/s Change scheduling policy

### lvsync

It synchronizes stale PE in given LV. Its used in mirroring environment. Whenever there is any disk failure or disk path issue, PE goes bad and LV in turns has stale PE. Once the issue is corrected we need to sync stale PE with this command if they doesnt sync automatically.

Command dosnt have any options. It should be supplied with LV path only.

```
lvsync /dev/vg00/lvol6
Resynchronized logical volume "/dev/vg00/lvol6".
```
 
### lvlnboot

This command used to define logical volume as a root, dump, swap or boot volume. You have to submit LV path along with specific option of your choice to command. Options are as below :

    -b Boot volume
    -d Dump volume
    -r Root volume
    -s Swap volume
    -R Recover any missing links
    -v Verbose mode

```
# lvlnboot -r /dev/vg00/lvol3
Volume Group configuration for /dev/vg00 has been saved in /etc/lvmconf/vg00.conf
 
# lvlnboot -b /dev/vg00/lvol1
Volume Group configuration for /dev/vg00 has been saved in /etc/lvmconf/vg00.conf
 
# lvlnboot -s /dev/vg00/lvol2
Volume Group configuration for /dev/vg00 has been saved in /etc/lvmconf/vg00.conf
 
# lvlnboot -d /dev/vg00/lvol2
Volume Group configuration for /dev/vg00 has been saved in /etc/lvmconf/vg00.conf
```

We have already seen this command in root disk mirroring.
