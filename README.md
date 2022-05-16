# Arch Linux Installation Cheatsheet

My personal cheatsheet for Arch Installation.

## Download latest ISO and verify checksums

[***Download Arch Linux***](https://archlinux.org/download)

## Boot to ISO and check Networking

I usually set bigger font with `setfont ter-132n` & connect to *wifi* :

* `iwctl`
* `device list`
* `station wlan0 get-networks` *(it's wlan0 in my case)*
* `station wlan0 connect "xxx"`
* `ip a`
* `ping www.archlinux.org`

>connecting with ethernet or mobile USB tethering is enabled by ***default***.

## Update system clock

* `timedatectl set-ntp true`, check with `timedatectl status`

## Partition the disk(s)

I install Arch on my ~233G SSD.

| *nvme0n1* | *File System* | *Size* | *Mount Point*                        | *Label* |
| --------- | ------------- | ------ | ------------------------------------ | ------- |
| nvme0n1p1 | fat32         | 550M   | /boot/efi                            | EFI     |
| nvme0n1p2 | btrfs         | 232G   | /<br>/home<br>/var/log<br>/var/cache | BTRFS   |

* `nvme0n1p2` remaining size. ***~232G***
* > later set up `zram`

## Format the Partitions

> script 0 starts; uncomment `ParallelDownloads` in `/etc/pacman.conf`
> use `-f` flag to force btrfs formatting if needed (used BTRFS before too)
* `mkfs.vfat /dev/nvme0n1p1 -n "EFI"`
  * or `mkfs.fat -F32 /dev/nvme0n1 -n "EFI"`
* `mkfs.btrfs /dev/nvme0n1p2 -L "BTRFS"`

## Mount the partitions

* `mount /dev/nvme0n1p2 /mnt`
* `btrfs su cr /mnt/@`, `btrfs su cr /mnt/@home`, `btrfs su cr /mnt/@cache`,
`btrfs su cr /mnt/@log`
* `umount /mnt`
* `mount -o defaults,noatime,compress=zstd,discard=async,space_cache=v2,autodefrag,subvol=@
/dev/nvme0n1p2 /mnt`
> `space_cache` on btrfs ***v5.15*** was creating issues in my drive
(though for small drives ***v1(default)*** is recommended)
* `mkdir -p /mnt/{home,boot/efi,var/cache,var/log}`
* `mount -o defaults,noatime,compress=zstd,discard=async,space_cache=v2,autodefrag,subvol=@home
/dev/nvme0n1p2 /mnt/home`
* `mount -o defaults,noatime,compress=zstd,discard=async,space_cache=v2,autodefrag,subvol=@cache
/dev/nvme0n1p2 /mnt/var/cache`
* `mount -o defaults,noatime,compress=zstd,discard=async,space_cache=v2,autodefrag,subvol=@log
/dev/nvme0n1p2 /mnt/var/log`
* `mount /dev/nvme0n1p1 /mnt/boot/efi`

## Install Arch Linux

* `reflector -f 10 -l 10 -c Germany --sort rate -p https --verbose --save /etc/pacman.d/mirrorlist` or `reflector -c India --sort rate --verbose --save /etc/pacman.d/mirrorlist`
* uncomment ***ParallelDownloads*** in ***/etc/pacman.conf***
  * repeat after *chroot*.
* `pacstrap -i /mnt base btrfs-progs linux linux-headers linux-firmware vim nano intel-ucode
  git`
* `genfstab -U /mnt >> /mnt/etc/fstab`
* `arch-chroot /mnt`
> script 0 ends; uncomment `ParallelDownloads` in `/etc/pacman.conf`

* Delete `subvolid`'s from `/etc/fstab`
* vim `/etc/locale.gen` and uncomment `en_IN` and `en_US` UTF-8
* `locale-gen`
> :octocat: *clone* `https://github.com/alokshandilya/arch-install-scripts` ,
modify it as needed, and *make it executable* by `chmod +x` and run.

* edit `/etc/mkinitcpio.conf`
  * add ***crc32c-intel*** in MODULES -`MODULES=(crc32c-intel)`
  * add ***grub-btrfs-overlayfs*** in **HOOKS** -
`HOOKS=(base udev .... fsck grub-btrfs-overlayfs)`
* `mkinitcpio -P`
* do `exit` , `umount -a` , `reboot`

## Post Installation

Connect to wifi with `nmtui`

* Adding Desktop Environment Or Window Manager
  * I use `dwm` (as for now)
* put `vm.swappiness=10` in `/etc/sysctl.d/100-arch.conf`
  * check with `cat /proc/sys/vm/swappiness` after reboot
* `mkdir ~/.android`, `echo "QuickbootFileBacked = off" >>
 ~/.android/advancedFeatures.ini`
