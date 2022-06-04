##### Creating a LVM from a new Hard disk 

```
[root@ip-172-31-86-253 ~]# fdisk -l
Disk /dev/xvda: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 30B4269A-8501-4012-B8AE-5DBF6CBBACA8

Device       Start      End  Sectors Size Type
/dev/xvda1    4096 16777182 16773087   8G Linux filesystem
/dev/xvda128  2048     4095     2048   1M BIOS boot

Partition table entries are not in disk order.


Disk /dev/xvdb: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


[root@ip-172-31-86-253 ~]# lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk 
└─xvda1 202:1    0   8G  0 part /
xvdb    202:16   0   8G  0 disk 


[root@ip-172-31-86-253 ~]# fdisk /dev/xvdb

Welcome to fdisk (util-linux 2.30.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xabdc7a34.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-16777215, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-16777215, default 16777215): +5G

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.



[root@ip-172-31-86-253 ~]# lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk 
└─xvda1 202:1    0   8G  0 part /
xvdb    202:16   0   8G  0 disk 
└─xvdb1 202:17   0   5G  0 part 


[root@ip-172-31-86-253 ~]# pvscan
  No matching physical volumes found
  
  
[root@ip-172-31-86-253 ~]# pvcreate /dev/xvdb1
  Physical volume "/dev/xvdb1" successfully created.
  
  
[root@ip-172-31-86-253 ~]# vgcreate vg00 /dev/xvdb1
  Volume group "vg00" successfully created
  
  
[root@ip-172-31-86-253 ~]# vgdisplay
  --- Volume group ---
  VG Name               vg00
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5.00 GiB
  PE Size               4.00 MiB
  Total PE              1279
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1279 / <5.00 GiB
  VG UUID               yfJGio-6WTf-KTgc-sB4T-7tEv-UA55-6EQuWW
  
  
   
[root@ip-172-31-86-253 ~]# lvcreate -n lv_backup -L 3G vg00
  Logical volume "lv_backup" created.
  
  
[root@ip-172-31-86-253 ~]# lvs
  LV        VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_backup vg00 -wi-a----- 3.00g                                                    
[root@ip-172-31-86-253 ~]# lvs -a -o +devices
  LV        VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices     
  lv_backup vg00 -wi-a----- 3.00g       
  
  /dev/sdb1(0)
  
  
[root@ip-172-31-86-253 ~]# lvdisplay vg00/lv_backup
  --- Logical volume ---
  LV Path                /dev/vg00/lv_backup
  LV Name                lv_backup
  VG Name                vg00
  LV UUID                YvWA4Y-gdqW-LtI2-abUY-sJrO-cPNp-exmhUI
  LV Write Access        read/write
  LV Creation host, time ip-172-31-86-253.ec2.internal, 2022-06-04 16:08:34 +0000
  LV Status              available
  # open                 0
  LV Size                3.00 GiB
  Current LE             768
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
  
  
   
[root@ip-172-31-86-253 ~]# lsblk
NAME               MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda               202:0    0   8G  0 disk 
└─xvda1            202:1    0   8G  0 part /
xvdb               202:16   0   8G  0 disk 
└─xvdb1            202:17   0   5G  0 part 
  └─vg00-lv_backup 253:0    0   3G  0 lvm  
 ```
 
 #### Formating LVM and mounting to /backup directory
 
 ```
 [root@ip-172-31-86-253 ~]# mkfs.ext4 /dev/vg00/lv_backup
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
196608 inodes, 786432 blocks
39321 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=805306368
24 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 


[root@ip-172-31-86-253 ~]# mkdir /backup


[root@ip-172-31-86-253 ~]# mount /dev/vg00/lv_backup /backup/


[root@ip-172-31-86-253 ~]# df -tH
df: no file systems processed
[root@ip-172-31-86-253 ~]# df -TH
Filesystem                 Type      Size  Used Avail Use% Mounted on
devtmpfs                   devtmpfs  497M     0  497M   0% /dev
tmpfs                      tmpfs     506M     0  506M   0% /dev/shm
tmpfs                      tmpfs     506M  484k  506M   1% /run
tmpfs                      tmpfs     506M     0  506M   0% /sys/fs/cgroup
/dev/xvda1                 xfs       8.6G  1.7G  7.0G  20% /
tmpfs                      tmpfs     102M     0  102M   0% /run/user/0
tmpfs                      tmpfs     102M     0  102M   0% /run/user/1000
/dev/mapper/vg00-lv_backup ext4      3.2G  9.5M  3.0G   1% /backup


[root@ip-172-31-86-253 ~]# lsblk
NAME               MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda               202:0    0   8G  0 disk 
└─xvda1            202:1    0   8G  0 part /
xvdb               202:16   0   8G  0 disk 
└─xvdb1            202:17   0   5G  0 part 
  └─vg00-lv_backup 253:0    0   3G  0 lvm  /backup
```

#### Extending the LVM to 1 GB size 

```
[root@ip-172-31-86-253 ~]# lvextend -L +1GB -r /dev/mapper/vg00-lv_backup
  Size of logical volume vg00/lv_backup changed from 3.00 GiB (768 extents) to 4.00 GiB (1024 extents).
  Logical volume vg00/lv_backup successfully resized.
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/mapper/vg00-lv_backup is mounted on /backup; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/vg00-lv_backup is now 1048576 blocks long.
```
#### Removing the LVM 

```
[root@ip-172-31-86-253 /]# lvremove /dev/vg00/lv_backup
Do you really want to remove active logical volume vg00/lv_backup? [y/n]: y
  Logical volume "lv_backup" successfully removed
  
  [root@ip-172-31-86-253 /]# lvs
[root@ip-172-31-86-253 /]# 
```

#### Recovering LVM

```
[root@ip-172-31-86-253 backup]# cd /etc/lvm/archive/
[root@ip-172-31-86-253 archive]# ls -al
total 16
drwx------ 2 root root  132 Jun  4 16:26 .
drwxr-xr-x 6 root root  100 Apr 28 19:54 ..
-rw------- 1 root root  920 Jun  4 16:07 vg00_00000-1671646791.vg
-rw------- 1 root root  928 Jun  4 16:08 vg00_00001-365865624.vg
-rw------- 1 root root 1384 Jun  4 16:20 vg00_00002-2083211155.vg
-rw------- 1 root root 1367 Jun  4 16:26 vg00_00003-351113624.vg
```
##### Getting details about the Archive files in a volume group

```
[root@ip-172-31-86-253 archive]# vgcfgrestore --list vg00
   
  File:		/etc/lvm/archive/vg00_00000-1671646791.vg
  Couldn't find device with uuid nhv405-LClR-Ht1T-QiWk-Dk2d-KqfC-RwxVXS.
  VG name:    	vg00
  Description:	Created *before* executing 'vgcreate vg00 /dev/xvdb1'
  Backup Time:	Sat Jun  4 16:07:06 2022

   
  File:		/etc/lvm/archive/vg00_00001-365865624.vg
  VG name:    	vg00
  Description:	Created *before* executing 'lvcreate -n lv_backup -L 3G vg00'
  Backup Time:	Sat Jun  4 16:08:34 2022

   
  File:		/etc/lvm/archive/vg00_00002-2083211155.vg
  VG name:    	vg00
  Description:	Created *before* executing 'lvextend -L +1GB -r /dev/mapper/vg00-lv_backup'
  Backup Time:	Sat Jun  4 16:20:00 2022

   
  File:		/etc/lvm/archive/vg00_00003-351113624.vg
  VG name:    	vg00
  Description:	Created *before* executing 'lvremove /dev/vg00/lv_backup'
  Backup Time:	Sat Jun  4 16:26:23 2022

   
  File:		/etc/lvm/backup/vg00
  VG name:    	vg00
  Description:	Created *after* executing 'lvremove /dev/vg00/lv_backup'
  Backup Time:	Sat Jun  4 16:26:23 2022
 ```
 
 #### Spoting the correct archive file before the removal process from the above details 
 
 ```
 File:		/etc/lvm/archive/vg00_00003-351113624.vg
  VG name:    	vg00
  Description:	Created *before* executing 'lvremove /dev/vg00/lv_backup'
  Backup Time:	Sat Jun  4 16:26:23 2022
 ```
 
 #### Restoring LVM from the specific file 
 
 ```
 [root@ip-172-31-86-253 archive]# vgcfgrestore -f /etc/lvm/archive/vg00_00003-351113624.vg vg00
  Restored volume group vg00


[root@ip-172-31-86-253 archive]# lvs
  LV        VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
 lv_backup vg00 -wi------- 4.00g 
 ```
 
 #### Currently the LVM is in inactive state 
 
 ```
 [root@ip-172-31-86-253 archive]# lvscan
  inactive          '/dev/vg00/lv_backup' [4.00 GiB] inherit
 ```
 
 #### Changing the LVM in to active state
 
 ```
 [root@ip-172-31-86-253 archive]# lvchange -ay /dev/vg00/lv_backup
[root@ip-172-31-86-253 archive]# lvs
  LV        VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_backup vg00 -wi-a----- 4.00g                                                    
[root@ip-172-31-86-253 archive]# lvscan
  ACTIVE            '/dev/vg00/lv_backup' [4.00 GiB] inherit
 ```
 
 #### Mounting the LVM back to the /backup directory
 
 ```
 [root@ip-172-31-86-253 archive]# mount /dev/vg00/lv_backup /backup/
 ```
 
 
  
