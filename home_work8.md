## 1. Imagine you was asked to add new partition to your host for backup purposes. To simulate appearance of new physical disk in your server, please create new disk in Virtual Box (5 GB) and attach it to your virtual machine.
Also imagine your system started experiencing RAM leak in one of the applications, thus while developers try to debug and fix it, you need to mitigate OutOfMemory errors; you will do it by adding some swap space.
/dev/sdc - 5GB disk, that you just attached to the VM (in your case it may appear as /dev/sdb, /dev/sdc or other, it doesn't matter)
1.1. Create a 2GB   !!! GPT !!!   partition on /dev/sdc of type "Linux filesystem" (means all the following partitions created in the following steps on /dev/sdc will be GPT as well)
```bash
[root@localhost linux]# yum install gdisk
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.sale-dedic.com
 * extras: mirror.sale-dedic.com
 * updates: mirror.reconn.ru
base                                                                                                                                                            | 3.6 kB  00:00:00
extras                                                                                                                                                          | 2.9 kB  00:00:00
updates                                                                                                                                                         | 2.9 kB  00:00:00
updates/7/x86_64/primary_db                                                                                                                                     |  13 MB  00:00:11
Resolving Dependencies
--> Running transaction check
---> Package gdisk.x86_64 0:0.8.10-3.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=======================================================================================================================================================================================
 Package                                   Arch                                       Version                                           Repository                                Size
=======================================================================================================================================================================================
Installing:
 gdisk                                     x86_64                                     0.8.10-3.el7                                      base                                     190 k

Transaction Summary
=======================================================================================================================================================================================
Install  1 Package

Total download size: 190 k
Installed size: 660 k
Is this ok [y/d/N]: y
Downloading packages:
gdisk-0.8.10-3.el7.x86_64.rpm                                                                                                                                   | 190 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : gdisk-0.8.10-3.el7.x86_64                                                                                                                                           1/1
  Verifying  : gdisk-0.8.10-3.el7.x86_64                                                                                                                                           1/1

Installed:
  gdisk.x86_64 0:0.8.10-3.el7

Complete!
[root@localhost linux]# gdisk /dev/sdb
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): n
Partition number (1-128, default 1):
First sector (34-10485726, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-10485726, default = 10485726) or {+-}size{KMGTP}: +2G
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/sdb: 10485760 sectors, 5.0 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 29FDD7B5-5A5E-4D4C-9363-5413FE293B91
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 10485726
Partitions will be aligned on 2048-sector boundaries
Total free space is 6291389 sectors (3.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     8300  Linux filesystem

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
```
1.2. Create a 512MB partition on /dev/sdc of type "Linux swap"
```bash
[root@localhost linux]# gdisk /dev/sdb
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): n
Partition number (2-128, default 2):
First sector (34-10485726, default = 4196352) or {+-}size{KMGTP}: +512M
Last sector (5244928-10485726, default = 10485726) or {+-}size{KMGTP}: ^C
[root@localhost linux]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0    5G  0 disk
└─sdb1            8:17   0    2G  0 part
sr0              11:0    1  973M  0 rom
[root@localhost linux]# n
bash: n: command not found
[root@localhost linux]# gdisk /dev/sdb
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): n
Partition number (2-128, default 2):
First sector (34-10485726, default = 4196352) or {+-}size{KMGTP}:
Last sector (4196352-10485726, default = 10485726) or {+-}size{KMGTP}: +512M
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): L
0700 Microsoft basic data  0c01 Microsoft reserved    2700 Windows RE
3000 ONIE boot             3001 ONIE config           4100 PowerPC PReP boot
4200 Windows LDM data      4201 Windows LDM metadata  7501 IBM GPFS
7f00 ChromeOS kernel       7f01 ChromeOS root         7f02 ChromeOS reserved
8200 Linux swap            8300 Linux filesystem      8301 Linux reserved
8302 Linux /home           8400 Intel Rapid Start     8e00 Linux LVM
a500 FreeBSD disklabel     a501 FreeBSD boot          a502 FreeBSD swap
a503 FreeBSD UFS           a504 FreeBSD ZFS           a505 FreeBSD Vinum/RAID
a580 Midnight BSD data     a581 Midnight BSD boot     a582 Midnight BSD swap
a583 Midnight BSD UFS      a584 Midnight BSD ZFS      a585 Midnight BSD Vinum
a800 Apple UFS             a901 NetBSD swap           a902 NetBSD FFS
a903 NetBSD LFS            a904 NetBSD concatenated   a905 NetBSD encrypted
a906 NetBSD RAID           ab00 Apple boot            af00 Apple HFS/HFS+
af01 Apple RAID            af02 Apple RAID offline    af03 Apple label
af04 AppleTV recovery      af05 Apple Core Storage    be00 Solaris boot
bf00 Solaris root          bf01 Solaris /usr & Mac Z  bf02 Solaris swap
bf03 Solaris backup        bf04 Solaris /var          bf05 Solaris /home
bf06 Solaris alternate se  bf07 Solaris Reserved 1    bf08 Solaris Reserved 2
bf09 Solaris Reserved 3    bf0a Solaris Reserved 4    bf0b Solaris Reserved 5
c001 HP-UX data            c002 HP-UX service         ea00 Freedesktop $BOOT
eb00 Haiku BFS             ed00 Sony system partitio  ed01 Lenovo system partit
Press the <Enter> key to see more codes: 8200
ef00 EFI System            ef01 MBR partition scheme  ef02 BIOS boot partition
fb00 VMWare VMFS           fb01 VMWare reserved       fc00 VMWare kcore crash p
fd00 Linux RAID
Hex code or GUID (L to show codes, Enter = 8300): 8200
Changed type of partition to 'Linux swap'

Command (? for help): p
Disk /dev/sdb: 10485760 sectors, 5.0 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 29FDD7B5-5A5E-4D4C-9363-5413FE293B91
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 10485726
Partitions will be aligned on 2048-sector boundaries
Total free space is 5242813 sectors (2.5 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     8300  Linux filesystem
   2         4196352         5244927   512.0 MiB   8200  Linux swap

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
```
1.3. Format the 2GB partition with an XFS file system
```bash
[root@localhost linux]# mkfs.xfs /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
1.4. Initialize 512MB partition as swap space
```bash
mkswap /dev/sdb2
swapon /dev/sdb2
KiB Swap:  2621432 total,  2621432 free,        0 used.  2474548 avail Mem
[root@localhost linux]# free -mh
              total        used        free      shared  buff/cache   available
Mem:           2.7G        194M        2.4G         11M        132M        2.4G
Swap:          2.5G          0B        2.5G
[root@localhost linux]# swapon -s
Filename                                Type            Size    Used    Priority
/dev/dm-1                               partition       2097148 0       -2
/dev/sdb2                               partition       524284  0       -3
```
1.5. Configure the newly created XFS file system to persistently mount at /backup
1.6. Configure the newly created swap space to be enabled at boot
```bash
[root@localhost linux]# blkid
/dev/sda1: UUID="2235a5c4-f0ee-432c-9c67-daa9f3fe813e" TYPE="xfs"
/dev/sda2: UUID="sQheNq-tEyv-lnl6-tItI-znvg-WkjM-VT0G6E" TYPE="LVM2_member"
/dev/sdb1: UUID="eaec8a87-2e83-47e5-ae6d-7901f27774e4" TYPE="xfs" PARTLABEL="Linux filesystem" PARTUUID="7f548926-fd5f-4582-8d22-a8a38ff11e35"
/dev/sdb2: UUID="3e6eae9d-6370-4fd7-aa4b-0c7e5bef1eae" TYPE="swap" PARTLABEL="Linux swap" PARTUUID="f7658294-185a-4002-a7d0-575bc470d8c7"
/dev/mapper/centos-root: UUID="ce66bac5-205c-41a2-8f85-a1dc9f01d443" TYPE="xfs"
/dev/mapper/centos-swap: UUID="5de37bc6-0de0-432b-a364-8c8c49236efb" TYPE="swap"

[root@localhost linux]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Thu Nov 25 10:35:32 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=2235a5c4-f0ee-432c-9c67-daa9f3fe813e /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
UUID=3e6eae9d-6370-4fd7-aa4b-0c7e5bef1eae       swap                    swap    defaults        0 0
UUID=eaec8a87-2e83-47e5-ae6d-7901f27774e4       /backup xfs     defaults        0 0

mount -a
[root@localhost linux]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 1.4G     0  1.4G   0% /dev
tmpfs                    1.4G     0  1.4G   0% /dev/shm
tmpfs                    1.4G   12M  1.4G   1% /run
tmpfs                    1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  2.2G   15G  13% /
/dev/sda1               1014M  194M  821M  20% /boot
tmpfs                    280M     0  280M   0% /run/user/1000
/dev/sdb1                2.0G   33M  2.0G   2% /backup
```
1.7. Reboot your host and verify that /dev/sdc1 is mounted at /backup and that your swap partition  (/dev/sdc2) is enabled
```bash
[root@localhost linux]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 1.4G     0  1.4G   0% /dev
tmpfs                    1.4G     0  1.4G   0% /dev/shm
tmpfs                    1.4G   12M  1.4G   1% /run
tmpfs                    1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  2.2G   15G  13% /
/dev/sdb1                2.0G   33M  2.0G   2% /backup
/dev/sda1               1014M  181M  834M  18% /boot
tmpfs                    280M     0  280M   0% /run/user/1000

[root@localhost linux]# swapon -s
Filename                                Type            Size    Used    Priority
/dev/sdb2                               partition       524284  0       -2
/dev/dm-1                               partition       2097148 0       -3
[root@localhost linux]# free -mh
              total        used        free      shared  buff/cache   available
Mem:           2.7G        199M        2.4G         11M        175M        2.3G
Swap:          2.5G          0B        2.5G

```
## 2. LVM. Imagine you're running out of space on your root device. As we found out during the lesson default CentOS installation should already have LVM, means you can easily extend size of your root device. So what are you waiting for? Just do it!
2.1. Create 2GB partition on /dev/sdc of type "Linux LVM"
```bash
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     8300  Linux filesystem
   2         4196352         5244927   512.0 MiB   8200  Linux swap
   3         5244928         9439231   2.0 GiB     8E00  Linux LVM
```
2.2. Initialize the partition as a physical volume (PV)
```bash
partprobe
[root@localhost linux]# pvcreate /dev/sdb3
  Physical volume "/dev/sdb3" successfully created.
```
2.3. Extend the volume group (VG) of your root device using your newly created PV
```bash
[root@localhost linux]# vgextend centos /dev/sdb3
  Volume group "centos" successfully extended
```
2.4. Extend your root logical volume (LV) by 1GB, leaving other 1GB unassigned
```bash
[root@localhost linux]# lvextend -L +1G /dev/mapper/centos-root
  Size of logical volume centos/root changed from <17.00 GiB (4351 extents) to <18.00 GiB (4607 extents).
  Logical volume centos/root successfully resized.
```
2.5. Check current disk space usage of your root device
```bash
[root@localhost linux]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 1.4G     0  1.4G   0% /dev
tmpfs                    1.4G     0  1.4G   0% /dev/shm
tmpfs                    1.4G   12M  1.4G   1% /run
tmpfs                    1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  2.2G   15G  13% /
/dev/sdb1                2.0G   33M  2.0G   2% /backup
/dev/sda1               1014M  194M  821M  20% /boot
tmpfs                    280M     0  280M   0% /run/user/1000
tmpfs                    280M     0  280M   0% /run/user/0
```
2.6. Extend your root device filesystem to be able to use additional free space of root LV
```bash
[root@localhost linux]# xfs_growfs -n /dev/centos/root
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=1113856 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=4455424, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

[root@localhost linux]# xfs_growfs /dev/centos/root
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=1113856 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=4455424, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 4455424 to 4717568

```
2.7. Verify that after reboot your root device is still 1GB bigger than at 2.5.
```bash
[root@localhost linux]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 1.4G     0  1.4G   0% /dev
tmpfs                    1.4G     0  1.4G   0% /dev/shm
tmpfs                    1.4G   12M  1.4G   1% /run
tmpfs                    1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/mapper/centos-root   18G  2.2G   16G  12% /
/dev/sdb1                2.0G   33M  2.0G   2% /backup
/dev/sda1               1014M  194M  821M  20% /boot
tmpfs                    280M     0  280M   0% /run/user/1000
tmpfs                    280M     0  280M   0% /run/user/0
```