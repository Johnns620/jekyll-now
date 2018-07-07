---
layout: post
title: Linux Won't Boot!
---

In this post I will discuss how to fix the issue of your Linux OS (Mint, Ubuntu, CentOS, etc.) not booting, say, after doing an update or installing disagreeable software.

Failure to boot a Linux OS can be caused by numerous issues. In my case, the problem occured after updating GRUB bootloader. This is one the worst problems I've encountered while using Linux. Personally, I use Linux Mint 18 Sarah, but these instructions will work for most all distributions. This is an alternative to using the [Boot-Repair tool](https://help.ubuntu.com/community/Boot-Repair).

## Overview of steps
1. Boot Arch Linux with bootable USB.
2. Establish Internet connection.
3. Mount file system and use `chroot`.
4. Fix bootloader issue and other necessary tasks like system updates.
5. Unmount and reboot.

## Step 1: Boot in Arch Linux with Bootable USB
I recommend always having an Arch Linux bootable USB ready. Reason being is that if you have really mucked up your current system with updates or whatever software, a bootable USB of your current OS or similar may not work either. Arch Linux is a great, light weight distribution and most likely will always boot with no problems. You can download the Arch Linux distribution [here](https://www.archlinux.org/download/), and instructions for creating the installer USB are [here](https://wiki.archlinux.org/index.php/USB_flash_installation_media).

## Step 2: Establish an Internet Connection
Once you've booted with Arch Linux, if you want to use WiFi, you need to configure you wireless network. Instructions to set up are [here](https://wiki.archlinux.org/index.php/Wireless_network_configuration), but I'll summarize below; just run the following commands.
1. Determine wireless interface name:
```
$ iw dev
```
2. Bring the interface up:
```
$ ip link set *interface* up
```
3. Configure *wpa_supplicant*:
```
$ wpa_supplicant -B -i *interface* -c <(wpa_passphrase MYSSID passphrase)
```

## Step 3: Mount File System
1. Use `fdisk -l` to determine the drive with your broken Linux OS, and use `uname -m` to determine the architecture (32bit or 64bit).
2. Change to root user with `su -`.
3. Mount the necessary directories:
```bash
$ cd /
$ mount -t ext4 /dev/sda1 /mnt
$ mount -t proc proc /mnt/proc
$ mount -t sysfs sys /mnt/sys
$ mount -o bind /dev /mnt/dev
```
**NB:** I did not need to mount the /boot directory to install a different version of the GRUB bootloader. However, if your `/boot` directory is on a different partition from `/`, you will need to mount it with `mount -t ext4 /dev/sda2 /mnt/boot` (if it is located on `/dev/sda2`).

## Step 4: Perform `chroot` and Fixes
1. Perform the chroot:
```bash
chroot /mnt /bin/bash
```
Now the root directory is that of your broken Linux installation. Perform `sudo apt-get update` or `grub-install` or other installations to fix your specific issue. For my issue, I rolled back my version of GRUB bootloader and all was well.

## Step 5: Exit Cleanly
Unmount all partitions with `umount /mnt/{proc, sys, dev}` (and `/mnt/boot` if needed), and then unmount the system drive with `umount /mnt`.
Reboot the system with `reboot`.

That should do it!. Take a look at the references below if necessary.

## References
- [Arch Linux Wireless Network configuration](https://wiki.archlinux.org/index.php/Wireless_network_configuration)
- [WPA_supplicant Setup](https://wiki.archlinux.org/index.php/WPA_supplicant#Advanced_usage)
- [What's the proper way to prepare chroot to recover a broken Linux installation?, Post on superuser.com](https://superuser.com/questions/111152/whats-the-proper-way-to-prepare-chroot-to-recover-a-broken-linux-installation)
- [Updating Debian Under a Chroot](http://shallowsky.com/blog/tags/chroot/)
- [Installing GRUB using grub-install](https://www.gnu.org/software/grub/manual/grub/html_node/Installing-GRUB-using-grub_002dinstall.html)
