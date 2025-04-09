#os-preparation-ansible


## WARNING
It is highly advised NOT to encrypt 4M partition on the primary disk as the partition contains BIOS BOOT Partition. This information can be seen with command: `parted <primary_disk> print`. The 4m paritition is marked with `bios_grub` Flag.
```
# lsblk
xvda     202:0    0   18G  0 disk
├─xvda1  202:1    0   17G  0 part /
├─xvda14 202:14   0    4M  0 part
├─xvda15 202:15   0  106M  0 part /boot/efi
└─xvda16 259:0    0  913M  0 part /boot

# parted /dev/xvda print
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 19.3GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
14      1049kB  5243kB  4194kB                     bios_grub
```



ansible-galaxy collection install community.general
ansible-galaxy collection install community.crypto
