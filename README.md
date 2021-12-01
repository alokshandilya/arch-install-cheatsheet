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

>connecting with ethernet or mobile USB tethering is enabled by default.

## Update system clock

* `timedatectl set-ntp true`, check with `timedatectl status`

## Partition the disk(s)

I install Arch on my ~233G SSD.

| *nvme0n1* | *File System* | *Size* | *Mount Point* | *Label* |
|-----------|---------------|--------|---------------|---------|
| nvme0n1p1 | fat32         | 300M   |/mnt/boot/efi  | EFI     |
| nvme0n1p2 | linux-swap    | 2G     |[SWAP]         |         |
| nvme0n1p3 | ext4          | 70G    |/mnt           | Arch    |
| nvme0n1p4 | ext4          |160G    |/mnt/home      | Home    |

* `nvme0n1p4` remaining size ~ 160 GB.

## Format the Partitions

* `mkfs.vfat /dev/nvme0n1p1 -n "EFI"`
  * or `mkfs.fat -F32 /dev/nvme0n1 -n "EFI"`
* `mkswap /dev/nvme0n1p2`
* `mkfs.ext4 /dev/nvme0n1p3 -L "Arch"`
* `mkfs.ext4 /dev/nvme0n1p4 -L "Home"`

## Mount the partitions

* `swapon /dev/nvme0n1p2`
* `mount /dev/nvme0n1p3 /mnt`
* mkdir's `/mnt/{home,boot}` and `/mnt/boot/efi`
* `mount /dev/nvme0n1p1 /mnt/boot/efi`
* `mount /dev/nvme0n1p4 /mnt/home`

## Install Arch Linux

* `reflector -c India --sort rate --save /etc/pacman.d/mirrorlist`
* uncomment ***ParallelDownloads*** in ***/etc/pacman.conf***
  * repeat after *chroot*.
* `pacstrap -i /mnt base linux linux-headers linux-firmware vim nano intel-ucode
  git`
* `genfstab -U /mnt >> /mnt/etc/fstab`
* `arch-chroot /mnt`

> :octocat: *clone* `https://github.com/alokshandilya/arch-install-scripts` ,
> modify it as needed, and *make it executable* by `chmod +x`

* edit `/etc/mkinitcpio.conf`
  * add ***i915 nvidia*** in MODULES - `MODULES=(i915 nvidia)`
* `mkinitcpio -P`
* do `exit` , `umount -a` , `reboot`
