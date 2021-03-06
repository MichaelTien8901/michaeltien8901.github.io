---
layout: post
title: "Create Redfish Test Image"
date: 2019-10-23
tags: [OpenBMC Redfish]
---

# Create Redfish Test Image

Follow the development steps of OpenBMC, create a image for QEMU to test Redfish functions.

## Building for Palmetto

The Palmetto target is palmetto.

```shell
$ cd openbmc
$ TEMPLATECONF=meta-ibm/meta-palmetto/conf . openbmc-env
$ bitbake obmc-phosphor-image
```

## Building the OpenBMC SDK

Looking for a way to compile your programs for 'ARM' but you happen to be running on a 'PPC' or 'x86' system? You can build the sdk receive a fakeroot environment.

```shell
$ bitbake -c populate_sdk obmc-phosphor-image
$ ./tmp/deploy/sdk/openbmc-phosphor-glibc-x86_64-obmc-phosphor-image-armv5e-toolchain-2.1.sh
```
## Get QEMU (OpenBMC version)

   The standard qemu-system-arm doesn't have the machine type palmetto used here.  So we need to get the openbmc version.

```shell
wget https://openpower.xyz/job/openbmc-qemu-build-merge-x86/lastSuccessfulBuild/artifact/qemu/arm-softmmu/qemu-system-arm

chmod u+x qemu-system-arm
```

## Using QEMU

QEMU has a palmetto-bmc machine (as of v2.6.0) which implements the core devices to boot a Linux kernel. OpenBMC also maintains a tree with patches on their way upstream or temporary work-arounds that add to QEMU's capabilities where appropriate.

```shell
export image_path=~/openbmc2/build/tmp/deploy/images/palmetto
export host_ip=192.168.30.163
# use the qemu openbmc version
./qemu-system-arm -m 256 -M palmetto-bmc -nographic \
-drive file=${image_path}/flash-palmetto,format=raw,if=mtd \
-net nic \
-net user,hostfwd=:127.0.0.1:2222-:22,hostfwd=${host_ip}:2443-:443,hostname=qemu \
```
If you get an error you likely need to build QEMU (see the section in this document). If no error and QEMU starts up just change the port when interacting with the BMC...

```
curl -c cjar -b cjar -k -H "Content-Type: application/json" \
-X POST https://${host_ip}:2443/login -d "{\"data\": [ \"root\", \"0penBmc\" ] }"
```

or

```shell
ssh -p 2222 root@localhost
```

or using browser

   https://192.168.30.163:2443/redfish/v1
   

To quit, type Ctrl-a c to switch to the QEMU monitor, and then quit to exit.

## Building QEMU

```
git clone https://github.com/openbmc/qemu.git
cd qemu
git submodule update --init dtc
mkdir build
cd build
../configure --target-list=arm-softmmu
make
```

Built file will be located at: arm-softmmu/qemu-system-arm

