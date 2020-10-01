# Extending VG and LVM

## My Lab

* VMware Workstation;

* 1x CentOS 7; 

* 2 vCPU | 2GB Memory | 20GB Disk + 10GB Disk

## Let's Go!

### LVM Extend

I suppose you have already created your VG and LVM.

**lsblk**
```
[root@chlabkubn02 ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm
sdb               8:16   0   10G  0 disk -> My disk with 10GB.
sr0              11:0    1  4.3G  0 rom
```
**pvs**
```
[root@chlabkubn02 ~]# pvs
  PV         VG     Fmt  Attr PSize   PFree
  /dev/sda2  centos lvm2 a--  <19.00g     0
  /dev/sdb          lvm2 ---   10.00g 10.00g
```

**lvcreate**

I created a LVM with 5GB to be able to increase.
```
[root@chlabkubn02 ~]# lvcreate -L 5000 -n chlablv01 chlabvg01
  Logical volume "chlablv01" created.
```
**lvs**

Used to list Logical Volume created.

```
[root@chlabkubn02 /]# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root      centos    -wi-ao---- <17.00g
  swap      centos    -wi-a-----   2.00g
  chlablv01 chlabvg01 -wi-a-----   4.88g -> Here is my LV with 5GB
 ```
**lvextend**
```
[root@chlabkubn02 chlabdocker]# lvextend -L +2GB /dev/chlabvg01/chlablv01
Size of logical volume chlabvg01/chlablv01 changed from 4.88 GiB (1250 extents) to 6.88 GiB (1762 extents).
```

**xfs_growfs**
```
[root@chlabkubn02 chlabdocker]# xfs_growfs /dev/chlabvg01/chlablv01
meta-data=/dev/mapper/chlabvg01-chlablv01 isize=512    agcount=4, agsize=320000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=1280000, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 1280000 to 1804288
```

**df**
```
[root@chlabkubn02 chlabdocker]# df -h
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/centos-root           17G  2.2G   15G  13% /
devtmpfs                         899M     0  899M   0% /dev
tmpfs                            910M     0  910M   0% /dev/shm
tmpfs                            910M   90M  821M  10% /run
tmpfs                            910M     0  910M   0% /sys/fs/cgroup
/dev/sda1                       1014M  192M  823M  19% /boot
tmpfs                            182M     0  182M   0% /run/user/0
/dev/mapper/chlabvg01-chlablv01  6.9G   33M  6.9G   1% /chlabdocker
```

### VG Extend

We added another disk to do this task.

```
[root@chlabkubn02 chlabdocker]# lsblk
NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                     8:0    0   20G  0 disk
├─sda1                  8:1    0    1G  0 part /boot
└─sda2                  8:2    0   19G  0 part
  ├─centos-root       253:0    0   17G  0 lvm  /
  └─centos-swap       253:1    0    2G  0 lvm
sdb                     8:16   0   10G  0 disk
└─chlabvg01-chlablv01 253:2    0  6.9G  0 lvm  /chlabdocker
sdc                     8:32   0    5G  0 disk -> My disk added
sr0                    11:0    1  4.3G  0 rom
```

**pvcreate**
```
[root@chlabkubn02 chlabdocker]# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
```

**pvs**
```
[root@chlabkubn02 chlabdocker]# pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda2  centos    lvm2 a--  <19.00g    0
  /dev/sdb   chlabvg01 lvm2 a--  <10.00g 3.11g
  /dev/sdc             lvm2 ---    5.00g 5.00g
```

**vgs**

Still within 10GB

```
[root@chlabkubn02 chlabdocker]# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  centos      1   2   0 wz--n- <19.00g    0
  chlabvg01   1   1   0 wz--n- <10.00g 3.11g
```

**vgextend**
```
[root@chlabkubn02 chlabdocker]# vgextend chlabvg01 /dev/sdc
  Volume group "chlabvg01" successfully extended
```

**vgs**

Now we can see 15GB

```
[root@chlabkubn02 chlabdocker]# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  centos      1   2   0 wz--n- <19.00g     0
  chlabvg01   2   1   0 wz--n-  14.99g <8.11g
```
