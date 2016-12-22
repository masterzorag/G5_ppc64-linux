# Linux on a G5 ppc64
- http://www.everymac.com/systems/apple/powermac_g5/specs/powermac_g5_2.0_dp_2.html

Fedora 25
- https://fedoraproject.org/wiki/Architectures/PowerPC
- https://archive.fedoraproject.org/pub/fedora-secondary/releases/25/Everything/ppc64/iso/Fedora-Everything-netinst-ppc64-25-1.2.iso

Debian 8.6.0
- https://www.debian.org/CD/netinst/
- http://cdimage.debian.org/debian-cd/8.6.0/powerpc/iso-cd/debian-8.6.0-powerpc-netinst.iso

# boot setup

### partition map
OpenFirmare load the default ofprogram from the first HFS partition (Apple_Bootstrap) from a mac partition table.

below I use about 16M on sda2 due default partition is smaller to store grub2 image later.

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

lsblk -o NAME,SIZE,UUID,LABEL

    NAME     SIZE UUID                                 LABEL
    sr0     1024M                                      
    sda    149.1G                                      
    ├─sda4    15G 46026799-c14a-4ae0-98fd-d5d70cd21c4c ROOT
    ├─sda2    15M                                      MAC_BOOT
    ├─sda5   992K                                      
    ├─sda3   228M 26507748-8918-49e0-9d3e-8e8c7b3da04d BOOT
    ├─sda1  31.5K                                      
    └─sda6 133.8G                                      
### grub2
- http://cynic.cc/blog/posts/running_grub2_on_powerpc_macs/
- https://www.gnu.org/software/grub/manual/grub.html#Embedded-configuration

grub.img can be cross-compiled on a 32bit HOST too, checkout https://github.com/crosstool-ng/crosstool-ng

embed a grub.cfg that points to (...hd,apple3)\boot\grub2\grub.cfg using UUID:

cat grub.cfg

    search.fs_uuid 26507748-8918-49e0-9d3e-8e8c7b3da04d root
    set prefix=($root)/grub2
    configfile /grub2/grub.cfg

grub2-mkimage -c grub.cfg -o grub -O powerpc-ieee1275 -C xz -p /usr/lib/grub/powerpc-ieee1275/*.mod

    grub: ELF 32-bit MSB executable, PowerPC or cisco 4500, version 1 (SYSV), statically linked, stripped

check OpenFirmware path

grub2-ofpathname /dev/sda2

    /ht@0,f2000000/pci@7/k2-sata-root@c/disk@0:b

### mac-blessing with hfsutils
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

# boot from OpenFirmware
- www.netneurotic.de/mac/openfirmware.html
- http://osxbook.com/book/bonus/ancient/whatismacosx/arch_boot.html

run grub from default HFS (Apple_Bootstrap) partition

    0 > boot hd:,grub

# linux
cat /etc/fedora-release

    Fedora release 25 (Twenty Five)

* autoload modules

cat /etc/sysconfig/modules/something.modules

    modprobe i2c-powermac
    modprobe rack-meter

* thermal and scaling are working

find /sys -iname "*temp*"

    /sys/kernel/debug/tracing/events/thermal/thermal_temperature
    /sys/devices/platform/windfarm.0/cpu-amb-temp-1
    /sys/devices/platform/windfarm.0/cpu-diode-temp-0
    /sys/devices/platform/windfarm.0/cpu-amb-temp-0
    /sys/devices/platform/windfarm.0/hd-temp
    /sys/devices/platform/windfarm.0/cpu-diode-temp-1
    /sys/devices/platform/windfarm.0/backside-temp
    /sys/devices/platform/temperature
    /sys/firmware/devicetree/base/u3@0,f8000000/i2c@f8001000/temp-monitor@96
    /sys/firmware/devicetree/base/u3@0,f8000000/i2c@f8001000/temp-monitor@96/temperature@0
    /sys/firmware/devicetree/base/u3@0,f8000000/i2c@f8001000/temp-monitor@94
    /sys/firmware/devicetree/base/u3@0,f8000000/i2c@f8001000/temp-monitor@94/temperature@0
    /sys/firmware/devicetree/base/u3@0,f8000000/i2c@f8001000/supply-monitor@5a/temperature@0
    /sys/firmware/devicetree/base/u3@0,f8000000/i2c@f8001000/supply-monitor@58/temperature@0
    /sys/firmware/devicetree/base/u3@0,f8000000/i2c@f8001000/temp-monitor@98
    /sys/firmware/devicetree/base/u3@0,f8000000/i2c@f8001000/temp-monitor@98/external-temperature@1
    /sys/firmware/devicetree/base/u3@0,f8000000/i2c@f8001000/temp-monitor@98/internal-temperature@0
    /sys/firmware/devicetree/base/sep/temperatures
    /sys/firmware/devicetree/base/sep/temperatures/cpu-b-diode-temp@10
    /sys/firmware/devicetree/base/sep/temperatures/cpu-a-diode-temp@b
    /sys/firmware/devicetree/base/sep/thermostats/overtemp*-signal@5800
    /sys/bus/platform/devices/temperature
cat /sys/devices/platform/windfarm.0/cpu-diode-temp-*

    34.794
    31.604
cat /proc/cpuinfo

    processor       : 0
    cpu             : PPC970, altivec supported
    clock           : 1304.400000MHz
    revision        : 2.2 (pvr 0039 0202)

    processor       : 1
    cpu             : PPC970, altivec supported
    clock           : 1304.400000MHz
    revision        : 2.2 (pvr 0039 0202)

    timebase        : 33333333
    platform        : PowerMac
    model           : PowerMac7,3
    machine         : PowerMac7,3
    motherboard     : PowerMac7,3 MacRISC4 Power Macintosh 
    detected as     : 336 (PowerMac G5)
    pmac flags      : 00000000
    L2 cache        : 512K unified
    pmac-generation : NewWorld


