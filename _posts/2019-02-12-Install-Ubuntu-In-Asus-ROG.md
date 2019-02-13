---
layout: post
title: "Install Ubuntu in Asus ROG Gaming Laptop(GL552VW)"
categories: linux
date: 2019-02-12
---

# Install Linux ubuntu 18.04LTS into Asus ROG 

   I bought another SSD drive and want to install it with ubuntu in my Asus ROG laptop.  

## Problem 
As per the installation guide page, I downloaded the .ISO file and Rufus. After Rufus has finished processing everything, I restarted my laptop and went to the setup page, setting my USB drive as main boot option and disabling all other options(security boot options). Restart computer.

In grub boot screen, I have options 'Try Ubuntu without installing', 'Install Ubuntu' and other options.  I choose 'Install Ubuntu', but screen froze at the uBuntu loading screen. 

I tried another USB drive and write the ISO again.  But it doesn't work.  

## Solution

* In grub boot screen, use arrow to choose Install Ubuntu, and press 'e' key.
* Go to the line which contains 'quiet splash', replace it with 'noveau.modeset=0'.  
* Press F10 to install.

* After installation, install nvidia driver
```shell
sudo apt-get install nvidia
```
   Press TAB twice and find out the driver name, like
```shell
sudo apt-get install nvidia-352
```


