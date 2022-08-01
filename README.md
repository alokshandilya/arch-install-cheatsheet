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

### Update system clock

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

- if mirrors are slow `reflector -f 20 -l 20 -c Germany --sort rate --verbose --save /etc/pacman.d/mirrorlist`

```sh
git clone https://github.com/alokshandilya/arch-install-scripts.git
```

all scripts are executable but still have a glance on the commands and **modify** accordingly

```sh
./1-chroot.sh
```

- run 🏃`./1-chroot.sh`
  - formats the partitions
  - makes btrfs subvolumes
  - mounts the partitions
  - pacstraps base, kernel etc to `/mnt`
  - generates fstab based on UUIDs _(remove subvolid later from `/etc/fstab`)_

```sh
arch-chroot /mnt
```

## Install Arch Linux

- in `/etc/pacman.conf`
  - uncomment `ParallelDownloads`
  - add `ILoveCandy`
  - enable `multilib` repository
- Delete `subvolid` from `/etc/fstab`
- vim `/etc/locale.gen` and uncomment `en_IN` and `en_US` **UTF-8**
- `locale-gen`
- change password in `2-base-install.sh`
- run 🏃`2-base-install.sh`

```sh
./2-base-install.sh
```

> run 🏃 `3-touchpad.sh` if to use Window Manager (on laptop) to enable trackpad reverse scrolling etc.

- edit `/etc/mkinitcpio.conf`
  - `MODULES=(crc32c-intel intel_agp i915 nvidia)`
- `mkinitcpio -P`
- do `exit` , `umount -a` , `reboot`

## Post Installation

- connect to wifi with `nmtui`

### Install DWM :robot:

```sh
./4-dwm-install.sh
```

- reboot
- run 🏃`5-packages-AUR.sh`
  - AURs are commented
- install rest of the packages

```sh
cd ~/arch-install-scripts
paru -S --needed - < pkglist.txt
```

