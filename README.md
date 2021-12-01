# Arch Linux Installation Cheatsheet
My personal cheatsheet for Arch Installation.

## Download latest ISO and verify checksums.
[***Download Arch Linux***](https://archlinux.org/download)

## Boot to ISO and check Networking.
I usually set bigger font with `setfont ter-132n` & connect to *wifi* :
* `iwctl`
* `device list` 
* `station wlan0 get-networks` *(it's wlan0 in my case)*
* `station wlan0 connect "xxx"`
* `ip a`
* `ping www.archlinux.org` 
>connecting with ethernet or mobile USB tethering is enabled by default.
