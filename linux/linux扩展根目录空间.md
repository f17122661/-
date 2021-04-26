# linux扩展根目录空间
**扩展根目录空间需要提前知道跟目录LVM的name 如何查看**
``` shell
[root@rabbitmq ~]# df -hl
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root
                       92G   12G   75G  14% /
tmpfs                 939M     0  939M   0% /dev/shm
/dev/sda1             477M   52M  400M  12% /boot


[root@rabbitmq ~]# vgdisplay 
  --- Volume group ---
  VG Name               VolGroup #这个是名字 我们往他的/dev/VolGroup/lv_root 这个下扩展磁盘
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  6
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               114.50 GiB
  PE Size               4.00 MiB
  Total PE              29313
  Alloc PE / Size       24194 / 94.51 GiB
  Free  PE / Size       5119 / 20.00 GiB
  VG UUID               R4T7Y8-msWS-vpUM-HpVJ-3PMT-Obdx-0RFqtT

```
## 1. 创建一个新的分区 
``` shell
[root@localhost ~]# fdisk /dev/sdb

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').
Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-13054, default 1): 
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-13054, default 13054): 
Using default value 13054
Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 8e
Changed system type of partition 1 to 8e (Linux LVM)

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.


```

## 2. 指定 /dev/sdb1的文件系统类型:
``` shell
Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 8e
Changed system type of partition 1 to 8e (Linux LVM)
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.
```

## 3. 对新分区进行格式化
``` shell
mkfs -t ext3 /dev/sdb1
```
## 4.  将/dev/sdb1制作为物理卷，即PV
``` shell
[root@localhost ~]# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
```
## 5.  将/dev/sdb1加入到逻辑卷组VolGroup当中
``` shell
[root@localhost ~]# vgextend VolGroup /dev/sdb1
  Volume group "VolGroup" successfully extended
```
## 6. 扩展逻辑卷 VolGroup
``` shell
lvextend -L +80G /dev/VolGroup/lv_root 
  Size of logical volume VolGroup/lv_root changed from 13.01 GiB (3330 extents) to 93.01 GiB (23810 extents).
  Logical volume lv_root successfully resized.
```

## 7. 调整ext文件系统的空间大小
``` shell
[root@rabbitmq ~]# resize2fs /dev/VolGroup/lv_root 
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/VolGroup/lv_root is mounted on /; on-line resizing required
old desc_blocks = 3, new_desc_blocks = 6
Performing an on-line resize of /dev/VolGroup/lv_root to 24381440 (4k) blocks. # 这里会很慢 分区的越大越慢
The filesystem on /dev/VolGroup/lv_root is now 24381440 blocks long.
```

## 8. 查看分区空间
``` shell
[root@rabbitmq ~]# df -hl
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root
                       92G   12G   75G  14% /
tmpfs                 939M     0  939M   0% /dev/shm
/dev/sda1             477M   52M  400M  12% /boo
```