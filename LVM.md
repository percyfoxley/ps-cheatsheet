## LVM

**ll /dev/sd\***
```
[root@localhost ~]# ll /dev/sd*
brw-rw----. 1 root disk 8,  0 Jun 16 11:40 /dev/sda
brw-rw----. 1 root disk 8,  1 Jun 16 11:40 /dev/sda1
brw-rw----. 1 root disk 8,  2 Jun 16 11:40 /dev/sda2
brw-rw----. 1 root disk 8, 16 Jun 16 11:40 /dev/sdb
brw-rw----. 1 root disk 8, 32 Jun 16 11:40 /dev/sdc
brw-rw----. 1 root disk 8, 48 Jun 16 11:43 /dev/sdd
```

**lsblk** lists information about all available or the specified block devices.

```
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0               2:0    1    4K  0 disk
sda               8:0    0   20G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0    5G  0 disk
sdc               8:32   0    5G  0 disk
sdb               8:48   0    10G  0 disk
sr0              11:0    1  973M  0 rom
 ```

### Disk Scan (host0 . . .)

```
# echo "- - -" > /sys/class/scsi_host/host2/scan
```

**fdisk** used for creating and manipulating disk partition table.

```
[root@localhost ~]# fdisk /dev/sdd
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-20971519, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519): +5G
Partition 1 of type Linux and of size 5 GiB is set

Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (2-4, default 2):
First sector (10487808-20971519, default 10487808):
Using default value 10487808
Last sector, +sectors or +size{K,M,G} (10487808-20971519, default 20971519):
Using default value 20971519
Partition 2 of type Linux and of size 5 GiB is set
Command (m for help): t
Partition number (1,2, default 2): 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

### Physical Volumes

**pvcreate** initialize a disk or partition for use by LVM

usage : `pvcreate </path/to/disk>`

```
[root@localhost ~]# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
[root@localhost ~]# pvcreate /dev/sdd1 /dev/sdd2
  Physical volume "/dev/sdd1" successfully created.
  Physical volume "/dev/sdd2" successfully created.
```
pvs, pvdisplay
```
[root@localhost ~]# pvs
  PV         VG     Fmt  Attr PSize  PFree
  /dev/sda2  centos lvm2 a--  <7.00g     0
  /dev/sdc          lvm2 ---   5.00g  5.00g
  /dev/sdd1         lvm2 ---   5.00g  5.00g
  /dev/sdd2         lvm2 ---  <5.00g <5.00g
```

### Volume Group

usage : `vgcreate <volume group> </path/to/disk>`

```
[root@localhost ~]# vgcreate test_vg /dev/sdc /dev/sdd1
  Volume group "test_vg" successfully created
```

**vgs,vgdisplay**

```
[root@localhost ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  centos    1   2   0 wz--n- <7.00g    0
  test_vg   2   0   0 wz--n-  9.99g 9.99g
[root@localhost ~]# vgscan
  Reading volume groups from cache.
  Found volume group "test_vg" using metadata type lvm2
  Found volume group "centos" using metadata type lvm2
[root@localhost ~]# vgdisplay -S vgname=test_vg -C -o pv_count
  #PV
    2
```
**vgextend**

```
[root@localhost ~]# pvdisplay -S vgname=test_vg -C -o pv_name
  PV
  /dev/sdc
  /dev/sdd1
[root@localhost ~]# vgextend test_vg /dev/sdd2
  Volume group "test_vg" successfully extended
[root@localhost ~]# pvdisplay -S vgname=test_vg -C -o pv_name
  PV
  /dev/sdc
  /dev/sdd1
  /dev/sdd2
```

### Logical Volume Management

**lvcreate** create a new logical volume in a volume group.

usage : `lvcreate -n <lv> -L <size> <volume group(vg)>`

```
[root@localhost ~]# lvcreate -L 5G -n lv1 test_vg
  Logical volume "lv1" created.
```

**lvdisplay** command displays the properties of LVM logical volumes.
The -C or --columns option displays the logical volume information in columns. It is equivalent to the **lvs** command.

```
[root@localhost ~]# ll /dev/test_vg/lv1
lrwxrwxrwx. 1 root root 7 Jun 16 11:58 /dev/test_vg/lv1 -> ../dm-2
```

**mkfs** disk formatting.

usage : `mkfs.ext4 </path/to/lv>` 

```
[root@localhost ~]# mkfs.ext4 /dev/test_vg/lv1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
327680 inodes, 1310720 blocks
65536 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=1342177280
40 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

[root@localhost ~]# mount -t ext4 /dev/test_vg/lv1 /mnt

[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0    8G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0    7G  0 part
  ├─centos-root 253:0    0  6.2G  0 lvm  /
  └─centos-swap 253:1    0  820M  0 lvm  [SWAP]
sdb               8:16   0    5G  0 disk
sdc               8:32   0    5G  0 disk
└─test_vg-lv1   253:2    0    7G  0 lvm  /mnt
sdd               8:48   0   10G  0 disk
├─sdd1            8:49   0    5G  0 part
│ └─test_vg-lv1 253:2    0    7G  0 lvm  /mnt
└─sdd2            8:50   0    5G  0 part
```

**or**

Configure **`/etc/fstab`** 

**`</path/to/lv> </path/to/directory>                        <file system>    defaults        0 0`**
```
vi /etc/fstab 
	/dev/mapper/centos-root /                       xfs     defaults        0 0
	UUID=92219321-4985-438f-a65c-a0776c7368e0 /boot                   xfs     defaults        0 0
	/dev/mapper/centos-swap swap                    swap    defaults        0 0
	#/dev/test_vg/lv1 /test                          ext4    defaults        0 0 <-- Added this line.
```	

`[root@localhost ~]# mount -a`

```
[root@localhost ~]# mount -a
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0    8G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0    7G  0 part
  ├─centos-root 253:0    0  6.2G  0 lvm  /
  └─centos-swap 253:1    0  820M  0 lvm  [SWAP]
sdb               8:16   0    5G  0 disk
sdc               8:32   0    5G  0 disk
└─test_vg-lv1   253:2    0    7G  0 lvm  /test
sdd               8:48   0   10G  0 disk
├─sdd1            8:49   0    5G  0 part
│ └─test_vg-lv1 253:2    0    7G  0 lvm  /test
└─sdd2            8:50   0    5G  0 part
```

