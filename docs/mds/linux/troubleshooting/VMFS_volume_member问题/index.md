!!! note "问题背景"
    利旧的一台机器做初始化的操作，之前是用来做Exsi的，在格式化分区的时候遇到了`VMFS_volume_member`的报错，针对这个问题进行处理

## 1. 问题分析

该机器之前是一台vmware exsi的机器，所以分区的格式是VMFS，这个我们使用`lsblk -f`是可以发现的。

```shell
# lsblk -f
NAME   FSTYPE             LABEL UUID                                 MOUNTPOINT
sda
├─sda1 xfs                      02243750-46f6-4654-9a6b-413da6ba6e49 /boot
├─sda2 swap                     bd9aa0f9-f9b5-49d1-a195-ebf2fe580be9 [SWAP]
└─sda3 xfs                      62314882-535f-44f0-bd1b-43e8d84bc728 /
sdb
└─sdb1 VMFS_volume_member
```
直接格式化成xfs的时候会收到如下的报错

```shell
# mkfs.xfs /dev/sdb1
mkfs.xfs: /dev/sdb1 appears to contain an existing filesystem (VMFS_volume_member).
mkfs.xfs: Use the -f option to force overwrite.
```
虽然上面提示我们说可以使用-f参数强制进行覆盖，但是实际上我们强制覆盖以后，sdb仍然是`VMFS_volume_member`格式的。

根据blkid的信息，`VMFS_volume_member`的SuperBlock位于分区开始往后1024KiB的位置，`VMFS_filesystem`的超级块位于2048KiB的偏移位置，所以说抹掉4MiB正好就可以把VMFS的信息给抹掉（下面是原文）

```
According to the blkid sources, the VMFS volume member superblock is located at the 1024 KiB offset from the start of partition, and the VMFS filesystem superblock is at the 2048 KiB offset, so erase 4 MiB just to be sure.
```

## 解决办法

```shell
# dd if=/dev/zero of=/dev/sdb1 bs=4M count=1
1+0 records in
1+0 records out
4194304 bytes (4.2 MB) copied, 0.232973 s, 18.0 MB/s
```
此时再查看信息

```shell
# lsblk -f
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda
├─sda1 xfs          02243750-46f6-4654-9a6b-413da6ba6e49 /boot
├─sda2 swap         bd9aa0f9-f9b5-49d1-a195-ebf2fe580be9 [SWAP]
└─sda3 xfs          62314882-535f-44f0-bd1b-43e8d84bc728 /
sdb
└─sdb1
```

我们通过parted重新分区格式化

```shell
# parted /dev/sdb
GNU Parted 3.1
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to continue?
Yes/No? Yes
(parted) print
Model: ATA ST2000NM0055 (scsi)
Disk /dev/sdb: 2000GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start  End  Size  File system  Name  Flags

(parted) mkpart primary 0% 100%
(parted) print
Model: ATA ST2000NM0055 (scsi)
Disk /dev/sdb: 2000GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  2000GB  2000GB               primary

(parted) quit
Information: You may need to update /etc/fstab.

# mkfs.xfs /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=122094592 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=488378368, imaxpct=5
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=238466, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@xeq-pm-192-97-prometheus data]# lsblk -f
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda
├─sda1 xfs          02243750-46f6-4654-9a6b-413da6ba6e49 /boot
├─sda2 swap         bd9aa0f9-f9b5-49d1-a195-ebf2fe580be9 [SWAP]
└─sda3 xfs          62314882-535f-44f0-bd1b-43e8d84bc728 /
sdb
└─sdb1 xfs          9fa4d4dc-7de2-47d5-b7bb-8542babc1500
```

此时数据就正常了。

## 附录：

```shell
dd if=/dev/zero of=/dev/sdb bs=1M count=16 
```
This should clear the GPT and allow you to use fdisk or any other partition/slice system you want.
Warning: For others, if it isn’t completely obvious, this will destroy whatever is currently on the disk.

    