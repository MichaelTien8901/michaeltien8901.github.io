---
layout: post
title: "Build OpenBMC for Raspberry Pi3 and Raspberry Pi Zero W"
date: 2022-12-29
---

## Choose OpenBMC branch `dunfell`

* Master Branch problem  
   The `master` branch define the `/meta-evb/meta-evb-raspberrypi` with different folder structure, which will break the `openbmc-env` action.

* Branch we use: `dunfell`

```console
cd ~/openbmc
git checkout dunfell
```

## Using docker to build OpenBMC

We will create CROP command file yocto-poky to build OpenBMC

* File `yocto-poky`

In `~/bin/`, we create the file `yocto-poky` as following,

```console
OBMC_PROJECT=~/
docker run --rm -it -v $OBMC_PROJECT:/workdir crops/poky --workdir=/workdir
```

* Set file attribute to executable

```console
chmod +777 yocto-poky
```

* Add PATH at end of ~/.bashrc

```.bashrc
export PATH=$PATH:~/bin
```

* Enter CROP docker container

```console
$ yocto-poky
```

This will enter docker compiler environments.

## Setup Raspberry Pi

* Setup Build

In the docker compiler environment,

```console
$ /workdir$ cd openbmc
$ /workdir/openbmc$ export TEMPLATECONF=meta-evb/meta-evb-raspberrypi/conf/
/workdir/openbmc$ . openbmc-env
```

This will create `/workdir/openbmc/build` folder.

* Setup Shared Download Directory `build/downloads/`  

Create symbolic link `downloads` to shared download directory `~/openbmc_downloads`.

```console
/workdir/openbmc/build$ ln -sf /workdir/openbmc_downloads/ downloads
```

* Adjust MACHINE
    In local.conf, change MACHINE settings to `raspberrypi3` from `raspberrypi`.

## Test Builds

### First Build

* bitbake image

```console
$ bitbake obmc-phosphor-image
```

* error message

```console
ERROR: obmc-phosphor-image-1.0-r0 do_generate_static: Image '/workdir/openbmc/build/tmp/deploy/images/raspberrypi3/fitImage-obmc-phosphor-initramfs-raspberrypi3-raspberrypi3' is too large!
ERROR: Logfile of failure stored in: /workdir/openbmc/build/tmp/work/raspberrypi3-openbmc-linux-gnueabi/obmc-phosphor-image/1.0-r0/temp/log.do_generate_static.2145213
ERROR: Task (/workdir/openbmc/meta-phosphor/recipes-phosphor/images/obmc-phosphor-image.bb:do_generate_static) failed with exit code '1'
```

### Adjust FLASH_SIZE to "131072" for the build error

Two options,

1. Define it at end of `openbmc/meta-raspberrypi/conf/machine/raspberrypi3.conf`

    ```conf
    FLASH_SIZE ?= "131072"
    ```

2. Define it at end of `build/conf/local.conf`.  This will just override the settings in raspberrypi3.conf.

    ```conf
    FLASH_SIZE ?= "131072"
    ```

***Both Build Image Successfully***.  We will just use option 2, changes in the `build/conf/local.conf`.


### Change Machine to "raspberrypi3-64"

At `build/conf/local.conf`

```conf
    MACHINE ??= "raspberrypi3-64" 
```

***Build Image Successfully***

### SD card Image Type

In order to generate sd card image, we will also add image type to IMAGE_FSTYPES in local.conf

```conf
IMAGE_FSTYPES += " rpi-sdimg"
```

## Finale

The following are the local.conf settings to generate OpenBMC images.

* Configuration for Raspberry Pi Zero WIFI

At `build/conf/local.conf`

```conf
MACHINE ??= "raspberrypi0-wifi"
...

GPU_MEM = "16"
DISTRO_FEATURE_append = " systemd"
VIRTUAL-RUNTIME_init_manager = " systemd"
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_initscripts = ""
IMAGE_FSTYPES += " rpi-sdimg"
IMAGE_INSTALL_append = " ipmitool wpa-supplicant"
FLASH_SIZE ?= "131072"
```

* Configuration for Raspberry Pi3

At `build/conf/local.conf`

```conf
MACHINE ??= "raspberrypi3"

...

GPU_MEM = "16"
DISTRO_FEATURE_append = " systemd"
VIRTUAL-RUNTIME_init_manager = " systemd"
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_initscripts = ""
IMAGE_FSTYPES += " rpi-sdimg"
IMAGE_INSTALL_append = " ipmitool wpa-supplicant"
FLASH_SIZE ?= "131072"
```

* Build Result

```console
okyuser@4a64fe10fd12:/workdir/openbmc/build$ bitbake obmc-phosphor-image
Parsing recipes: 100% |################################################################| Time: 0:00:04
Parsing of 2286 .bb files complete (0 cached, 2286 parsed). 3391 targets, 354 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies

Build Configuration:
BB_VERSION           = "1.46.0"
BUILD_SYS            = "x86_64-linux"
NATIVELSBSTRING      = "ubuntu-18.04"
TARGET_SYS           = "arm-openbmc-linux-gnueabi"
MACHINE              = "raspberrypi3"
DISTRO               = "openbmc-phosphor"
DISTRO_VERSION       = "0.1.0"
TUNE_FEATURES        = "arm vfp cortexa7 neon vfpv4 thumb callconvention-hard"
TARGET_FPU           = "hard"
meta                 
meta-poky            
meta-oe              
meta-networking      
meta-python          
meta-webserver       
meta-phosphor        
meta-raspberrypi     = "dunfell:61a2d43a172b70aa34fd7ec33fc048a211fa5c4c"

Initialising tasks: 100% |#############################################################| Time: 0:00:03
Sstate summary: Wanted 64 Found 17 Missed 47 Current 1389 (26% match, 96% complete)
NOTE: Executing Tasks
NOTE: Tasks Summary: Attempted 4230 tasks of which 4038 didn't need to be rerun and all succeeded.
```
