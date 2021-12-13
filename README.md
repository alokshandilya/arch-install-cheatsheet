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
| nvme0n1p2 | linux-swap    | 4G     |[SWAP]         |         |
| nvme0n1p3 | btrfs         |160G    |/              | BTRFS   |
|           |               |        |/home          |         |
|           |               |        |/var/log       |         |
|           |               |        |/var/cache     |         |

* `nvme0n1p3` remaining size.

## Format the Partitions

* `mkfs.vfat /dev/nvme0n1p1 -n "EFI"`
  * or `mkfs.fat -F32 /dev/nvme0n1 -n "EFI"`
* `mkswap /dev/nvme0n1p2`
* `mkfs.btrfs /dev/nvme0n1p3 -L "BTRFS"`

## Mount the partitions

* `swapon /dev/nvme0n1p2`
* `mount /dev/nvme0n1p3 /mnt`
* `btrfs su cr /mnt/@`, `btrfs su cr /mnt/@home`, `btrfs su cr /mnt/@cache`,
`btrsfs su cr /mnt@log`
* `umount /mnt`
* `mount -o defaults,noatime,compress=zstd,discard=async,space_cache,subvol=@
/dev/nvme0n1p3 /mnt`
* `mkdir -p /mnt/{home,boot/efi,var/cache,var/log}`
* `mount -o defaults,noatime,compress=zstd,discard=async,space_cache,subvol=@home
/dev/nvme0n1p3 /mnt/home`
* `mount -o defaults,noatime,compress=zstd,discard=async,space_cache,subvol=@cache
/dev/nvme0n1p3 /mnt/var/cache`
* `mount -o defaults,noatime,compress=zstd,discard=async,space_cache,subvol=@log
/dev/nvme0n1p3 /mnt/var/log`
* `mount /dev/nvme0n1p1 /mnt/boot/efi`

## Install Arch Linux

* `reflector -c India --sort rate --save /etc/pacman.d/mirrorlist`
* uncomment ***ParallelDownloads*** in ***/etc/pacman.conf***
  * repeat after *chroot*.
* `pacstrap -i /mnt base btrfs-progs linux linux-headers linux-firmware vim nano intel-ucode
  git`
* `genfstab -U /mnt >> /mnt/etc/fstab`
* `arch-chroot /mnt`

> :octocat: *clone* `https://github.com/alokshandilya/arch-install-scripts` ,
> modify it as needed, and *make it executable* by `chmod +x`

* edit `/etc/mkinitcpio.conf`
  * add ***i915 nvidia*** in MODULES - `MODULES=(crc32c-intel)`
* `mkinitcpio -P`
* do `exit` , `umount -a` , `reboot`

## Post Installation

I usually increase font by `setfont ter-132n` and connect to wifi with `nmtui`

* Adding Desktop Environment Or Window Manager
  * I use `dwm` (as for now)
  * > :octocat: *clone* `https://github.com/alokshandilya/dwm`
    * `sudo make clean install`
* put `vm.swappiness=10` in `/etc/sysctl.d/100-arch.conf`
  * check with `cat /proc/sys/vm/swappiness` after reboot
