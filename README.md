# Arch Linux Installation Cheatsheet

My personal cheatsheet for Arch Installation.

- download latest ISO and verify checksums
  - [**Download Arch Linux**](https://archlinux.org/download)
- scripts to install, _snapper setup_
  - [**Install Scripts**](https://github.com/alokshandilya/arch-install-scripts.git)

## Boot to ISO and check Networking

I usually set bigger font with `setfont ter-132n` & connect to _WiFi_ :

- `iwctl`
- `device list`
- `station wlan0 get-networks` _(it's wlan0 in my case)_
- `station wlan0 connect "xxx"`
- `ip a`
- `ping www.archlinux.org`

> connecting with Ethernet or mobile USB tethering is enabled by **_default_**

## Update system clock

- `timedatectl set-ntp true`, check with `timedatectl status`

## Partition the disk(s)

I install Arch on my ~233G SSD.

| _nvme0n1_ | _File System_ | _Size_ | _Mount Point_                         | _Label_ |
| --------- | ------------- | ------ | ------------------------------------- | ------- |
| nvme0n1p1 | fat32         | 550M   | /boot/efi                             | EFI     |
| nvme0n1p2 | btrfs         | 232G   | /<br>/home<br>/var/log<br>/.snapshots | BTRFS   |

- `nvme0n1p2` remaining size. **_~232G_**
  > later set up `zram`

## Format and Mount the partitions

uncomment `ParallelDownloads` in `/etc/pacman.conf` and install _git_

```sh
pacman -Sy git archlinux-keyring
```

```sh
git clone https://github.com/alokshandilya/arch-install-scripts.git
```

all scripts are executable but still have a glance on the commands and **modify** accordingly

- run `./1-chroot.sh`
  - formats the partitions
  - makes btrfs subvolumes
  - mounts the partitions
  - pacstraps base, kernel etc to `/mnt`
  - generates fstab based on UUIDs _(remove subvolid later from btrfs subvolume)_

```sh
arch-chroot /mnt
```

## Install Arch Linux

* `reflector -f 10 -l 10 -c Germany --sort rate -p https --verbose --save /etc/pacman.d/mirrorlist` or `reflector -c India --sort rate --verbose --save /etc/pacman.d/mirrorlist`
* uncomment ***ParallelDownloads*** in ***/etc/pacman.conf***
  * repeat after *chroot*.
* `pacstrap -i /mnt base btrfs-progs linux linux-headers linux-firmware vim nano intel-ucode
  git`
* `genfstab -U /mnt >> /mnt/etc/fstab`
* `arch-chroot /mnt`
> `script-1` ENDS.... 🏁
> uncomment `ParallelDownloads` in `/etc/pacman.conf` and enable `multilib`

* Delete `subvolid`'s from `/etc/fstab`
* vim `/etc/locale.gen` and uncomment `en_IN` and `en_US` UTF-8
* `locale-gen`

> `script-2` run 🏃 `2-base-install.sh`
> also run 🏃 `script-3` if planning to use Window Manager (on laptop)

* edit `/etc/mkinitcpio.conf`
  * add ***crc32c-intel*** in MODULES -`MODULES=(crc32c-intel intel_agp i915 nvidia)`
  * add ***grub-btrfs-overlayfs*** in **HOOKS** -
`HOOKS=(base udev .... fsck grub-btrfs-overlayfs)`
* `mkinitcpio -P`
* do `exit` , `umount -a` , `reboot`

## Post Installation

Connect to wifi with `nmtui`

* choose furthur script (dwm or kde)
    * after login to WM or DE run :runner: `5-after-install.sh` script.
