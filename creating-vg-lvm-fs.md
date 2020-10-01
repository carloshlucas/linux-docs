# Creating VG | LVM | FS

## My Lab

* VMware Workstation;
* 1x CentOS 7;
* 2 vCPU | 2GB Memory | 20GB Disk + 10GB Disk

## Let's Go!

**lsblk**

Used to list device and view status.

```
[root@chlabkubn02 ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm
sdb               8:16   0   10G  0 disk
sr0              11:0    1  4.3G  0 rom
```

**pvcreate**
```    
[root@chlabkubn02 ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```

**pvs**

Used to list phisycal Volume.

```
[root@chlabkubn02 ~]# pvs
  PV         VG     Fmt  Attr PSize   PFree
  /dev/sda2  centos lvm2 a--  <19.00g     0
  /dev/sdb          lvm2 ---   10.00g 10.00g
```

**vgcreate**

Used to initialize an entire disk or a disk partition for use LVM.

```
[root@chlabkubn02 ~]# vgcreate chlabvg01 /dev/sdb
  Volume group "chlabvg01" successfully created
  
```
**vgs** 

Used to list a logical volume group

```
[root@chlabkubn02 ~]# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  centos      1   2   0 wz--n- <19.00g      0
  chlabvg01   1   0   0 wz--n- <10.00g <10.00g
```

**lvcreate** 

Used to create a logical volume lvcreate.

```
[root@chlabkubn02 ~]# lvcreate -L 5000 -n chlablv01 chlabvg01
  Logical volume "chlablv01" created.
```

**lvs**

Used to list Logical Volume created.

```
[root@chlabkubn02 ~]# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root      centos    -wi-ao---- <17.00g
  swap      centos    -wi-a-----   2.00g
  chlablv01 chlabvg01 -wi-a-----   4.88g
```

**vgdisplay**

 Used to get details about VG.

```
[root@chlabkubn02 ~]# vgdisplay -v chlabvg01
  --- Volume group ---
  VG Name               chlabvg01
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       1250 / 4.88 GiB
  Free  PE / Size       1309 / 5.11 GiB
  VG UUID               HPzBOj-N5vb-cAsn-JnL8-TBhD-cKYv-lBhVKc

  --- Logical volume ---
  LV Path                /dev/chlabvg01/chlablv01
  LV Name                chlablv01
  VG Name                chlabvg01
  LV UUID                9QFKTY-Ks4F-dS7F-cAJ4-gzpa-PSus-F2gwqG
  LV Write Access        read/write
  LV Creation host, time chlabkubn02.chlab.local, 2019-06-26 10:33:47 +0200
  LV Status              available
  # open                 0
  LV Size                4.88 GiB
  Current LE             1250
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2

  --- Physical volumes ---
  PV Name               /dev/sdb
  PV UUID               08iD6Q-zepH-L8Ce-FI1N-bgjN-aPpS-Ur2mwR
  PV Status             allocatable
  Total PE / Free PE    2559 / 1309

[root@chlabkubn02 ~]# vgdisplay -v chlabvg01 | more
  --- Volume group ---
  VG Name               chlabvg01
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       1250 / 4.88 GiB
  Free  PE / Size       1309 / 5.11 GiB
  VG UUID               HPzBOj-N5vb-cAsn-JnL8-TBhD-cKYv-lBhVKc

  --- Logical volume ---
  LV Path                /dev/chlabvg01/chlablv01
  LV Name                chlablv01
  VG Name                chlabvg01
  LV UUID                9QFKTY-Ks4F-dS7F-cAJ4-gzpa-PSus-F2gwqG
  LV Write Access        read/write
  LV Creation host, time chlabkubn02.chlab.local, 2019-06-26 10:33:47 +0200
  LV Status              available
  # open                 0
  LV Size                4.88 GiB
  Current LE             1250
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2

  --- Physical volumes ---
  PV Name               /dev/sdb
  PV UUID               08iD6Q-zepH-L8Ce-FI1N-bgjN-aPpS-Ur2mwR
  PV Status             allocatable
  Total PE / Free PE    2559 / 1309
```

**mkdir**

Used to create a folder to use as a mount point.

```    
[root@chlabkubn02 ~]# mkdir /chlabdocker
```

**mkfs**

Used to make a FS.

```
[root@chlabkubn02 ~]# mkfs -t xfs /dev/chlabvg01/chlablv01
meta-data=/dev/chlabvg01/chlablv01 isize=512    agcount=4, agsize=320000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=1280000, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

**mount**

Used to mount a volume.

```
[root@chlabkubn02 ~]# mount /dev/chlabvg01/chlablv01 /chlabdocker/
```

**df** 
```   
[root@chlabkubn02 ~]# df -h
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/centos-root           17G  2.2G   15G  13% /
devtmpfs                         899M     0  899M   0% /dev
tmpfs                            910M     0  910M   0% /dev/shm
tmpfs                            910M   90M  821M  10% /run
tmpfs                            910M     0  910M   0% /sys/fs/cgroup
/dev/sda1                       1014M  192M  823M  19% /boot
tmpfs                            182M     0  182M   0% /run/user/0
/dev/mapper/chlabvg01-chlablv01  4.9G   33M  4.9G   1% /chlabdocker
```

**umonunt**

Used do umount the FS.

```
[root@chlabkubn02 ~]# umount /chlabdocker/
```

**fstab**

Used to mount the FS listed automatically.

```
[root@chlabkubn02 ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Mon Jun 24 14:14:07 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=f14e3b1d-fa57-4b16-b67d-9a8d706d2acc /boot                   xfs     defaults        0 0
# /dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/chlabvg01/chlablv01  /chlabdocker xfs   defaults 0 0
```
## How extend VG and LVM?

* https://github.com/carloshlucas/Infrastructure/blob/master/Linux%20-%20Plataform/Extend%20%7C%20VG%20%7C%20LVM

## References

* https://www.youtube.com/watch?v=0H3_LW4w3yI&feature=youtu.be

* https://www.miarec.com/doc/administration-guide/doc1012
