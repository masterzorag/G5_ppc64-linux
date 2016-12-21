# G5_ppc64-linux

Fedora 25 ppc64

# mac partition
parted /dev/sda
   
    GNU Parted 3.2
    Using /dev/sda
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) p free                                    
    Model: ATA Hitachi HTS54251 (scsi)
    Disk /dev/sda: 160GB
    Sector size (logical/physical): 512B/512B
    Partition Table: mac
    Disk Flags: 
    Number  Start     End    Size  File system  Name   Flags
    
      1      512B  32.8kB  32.3kB               Apple
           32.8kB  1049kB  1016kB  Free Space
      2    1049kB  16.8MB  15.7MB  hfs                 boot
      3    16.8MB   256MB   239MB  ext4
      4     256MB  16.4GB  16.1GB  ext4         root   root
           16.4GB   160GB   144GB  Free Space

lsblk

    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sr0     11:0    1  1024M  0 rom  
    sda      8:0    0 149.1G  0 disk 
    ├─sda4   8:4    0    15G  0 part /
    ├─sda2   8:2    0    15M  0 part 
    ├─sda5   8:5    0   992K  0 part 
    ├─sda3   8:3    0   228M  0 part /boot
    ├─sda1   8:1    0  31.5K  0 part 
    └─sda6   8:6    0 133.8G  0 part 

# grub.img and blessing
hmount /dev/sda2

    Volume name is "MAC_BOOT"
    Volume was created on Thu Jan  1 03:01:17 1970
    Volume was last modified on Tue Dec 20 19:34:16 2016
    Volume has 12086784 bytes free

hcopy grub :grub

hattrib -t tbxi :grub

hattrib -b :

hdir :

     f  tbxi/UNIX         0   1698620 Dec 20 19:23 grub
     f  tbxi/UNIX         0   1691400 Jan  1  1970 another_grub.img

humount /dev/sda2

# autoload modules
cat /etc/sysconfig/modules/something.modules

    modprobe i2c-powermac
    modprobe rack-meter


