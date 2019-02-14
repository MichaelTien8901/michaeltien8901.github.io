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
/dev/sda2 partition 2676732 73380 -1
```
```shell
gksu gedit /etc/default/grub
```
& to pull up the boot loader configuration.
Look for the line 
```shell
GRUB_CMDLINE_LINUX="" 
```
and make sure it looks like this (using your UUID of course).
```shell
GRUB_CMDLINE_LINUX="resume=UUID=41e86209-3802-424b-9a9d-d7683142dab7" 
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
resume=UUID=41e86209-3802-424b-9a9d-d7683142dab7
```
(with your UUID of course in place of mine). Save the file! 
```shell
sudo update-initramfs -u 
```

Reboot!
