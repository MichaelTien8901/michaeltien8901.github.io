---
layout: post
title: "Ubuntu Black screen after suspend"
categories: Linux
date: 2019-02-13
---
[https://askubuntu.com/questions/248967/laptop-screen-stays-blank-after-lid-is-reopened/1098390#1098390](https://askubuntu.com/questions/248967/laptop-screen-stays-blank-after-lid-is-reopened/1098390#1098390)

* INFO: This will not work for 12.04, resume from hibernate work differently in 12.04.' 

Pull up a Terminal again and run 
```
cat /proc/swaps 
```
and hopefully you see the path to your swap partition listed there. If not chances are something went wrong in the steps above. Here's my output:
```shell
Filename Type Size Used Priority
/dev/dm-1 partition 2676732 73380 -1
```
```shell
Then Find the uuid of the device
```shell
$ cd /dev/disk/by-uuid
$ ll
total 0
drwxr-xr-x 2 root root 160 Feb 13 23:17 ./
drwxr-xr-x 8 root root 160 Feb 13 23:17 ../
lrwxrwxrwx 1 root root  10 Feb 13 23:17 06af334d-676b-4b46-88ec-30105b980e1d -> ../../dm-1
```
gksu gedit /etc/default/grub
```
& to pull up the boot loader configuration.
Look for the line 
```shell
GRUB_CMDLINE_LINUX="" 
```
and make sure it looks like this (using your UUID of course).
```shell
GRUB_CMDLINE_LINUX="resume=UUID=06af334d-676b-4b46-88ec-30105b980e1d" 
```
and save the file 
```shell
sudo update-grub 
```
and wait for it to finish 
```shell
gksu gedit /etc/initramfs-tools/conf.d/resume 
```
& and make sure its contents are 
```shell
resume=UUID=06af334d-676b-4b46-88ec-30105b980e1d
```
(with your UUID of course in place of mine). Save the file! 
```shell
sudo update-initramfs -u 
```

Reboot!
