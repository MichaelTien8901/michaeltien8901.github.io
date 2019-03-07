---
layout: post
title: "bmcweb Update for Web UI development"
date: 2019-03-06
---
## Problem

In the document [OpenBMC Web User Interface Development](https://github.com/openbmc/docs/blob/master/development/web-ui.md), 
we should be able to develop Web UI from the local developing PC(localhost:8080) while directing to BMC host or BMC IP address
(localhost:2443).  But it doesn't work for the download Romulus image.  

## Solution

From [https://github.com/openbmc/phosphor-webui/blob/master/README.md](https://github.com/openbmc/phosphor-webui/blob/master/README.md)
**Note** that some OpenBMC implementations use [bmcweb](https://github.com/openbmc/bmcweb)
for its backend. For security reasons, bmcweb will need to be recompiled and
loaded onto the target BMC Host before the above redirect command will work. The
option to turn on within bmcweb is `BMCWEB_INSECURE_DISABLE_XSS_PREVENTION`.

## Changes bmcweb

* Import the bmcweb in [https://github/openbmc/bmcweb](https://github/openbmc/bmcweb) to my github repos.

* clone it into local folder.

* In file CMakeLists.txt, change `BMCWEB_INSECURE_DISABLE_XSS_PREVENTION` option.
```
option (BMCWEB_INSECURE_DISABLE_XSS_PREVENTION "Disable XSS preventions" ON)
```
* commit and push

## Changes in openbmc

* Find the location of the recipes for bmcweb.

```
~/openbmc$ find . -name bmcweb*
```

   We find it in ./meta-phosphor/recipes-phosphor/interfaces/bmcweb_git.bb
   
*  Change bmcweb_git.bb.  Redirect to my github repo and commit version

```
SRC_URI = "git://github.com/MichaelTien8901/bmcweb.git"
SRCREV = "7d044f74293a959a7fc8b63f7c4052a70b56dde9"
```

* Rebuild Image

```
. openbmc-env
bitbake obmc-phosphor-image
```

## Alternatives
[https://github.com/openbmc/bmcweb/blob/master/DEVELOPING.md](https://github.com/openbmc/bmcweb/blob/master/DEVELOPING.md). 


12. ### Developing and Testing
  There are a variety of ways to develop and test bmcweb software changes.
  Here are the steps for using the SDK and QEMU.

  - Follow all [development environment setup](https://github.com/openbmc/docs/blob/master/development/dev-environment.md)
  directions in the development environment setup document. This will get
  QEMU started up and you in the SDK environment.
  - Follow all of the [gerrit setup](https://github.com/openbmc/docs/blob/master/development/gerrit-setup.md)
  directions in the gerrit setup document.
  - Clone bmcweb from gerrit
  ```
  git clone ssh://openbmc.gerrit/bmcweb/
  ```

  - Ensure it compiles
  ```
  cmake ./ && make
  ```
  **Note:** If you'd like to enable debug traces in bmcweb, use the
  following command for cmake
  ```
  cmake ./ -DCMAKE_BUILD_TYPE:type=Debug
  ```

  - Make your changes as needed, rebuild with `make`

  - Reduce binary size by stripping it when ready for testing
  ```
  arm-openbmc-linux-gnueabi-strip bmcweb
  ```
  **Note:** Stripping is not required and having the debug symbols could be
  useful depending on your testing. Leaving them will drastically increase
  your transfer time to the BMC.

  - Copy your bmcweb you want to test to /tmp/ in QEMU
  ```
  scp -P 2222 bmcweb root@127.0.0.1:/tmp/
  ```
  **Special Notes:**
  The address and port shown here (127.0.0.1 and 2222) reaches the QEMU session
  you set up in your development environment as described above.

  - Stop bmcweb service within your QEMU session
  ```
  systemctl stop bmcweb
  ```
  **Note:** bmcweb supports being started directly in parallel with the bmcweb
  running as a service. The standalone bmcweb will be available on port 18080.
  An advantage of this is you can compare between the two easily for testing.
  In QEMU you would need to open up port 18080 when starting QEMU. Your curl
  commands would need to use 18080 to communicate.

  - If running within a system that has read-only /usr/ filesystem, issue
  the following commands one time per QEMU boot to make the filesystem
  writeable
  ```
  mkdir -p /var/persist/usr
  mkdir -p /var/persist/work/usr
  mount -t overlay -o lowerdir=/usr,upperdir=/var/persist/usr,workdir=/var/persist/work/usr overlay /usr
  ```

  - Remove the existing bmcweb from the filesystem in QEMU
  ```
  rm /usr/bin/bmcweb
  ```

  - Link to your new bmcweb in /tmp/
  ```
  ln -sf /tmp/bmcweb /usr/bin/bmcweb
  ```

  - Test your changes. bmcweb will be started automatically upon your
  first REST or Redfish command
  ```
  curl -c cjar -b cjar -k -X POST https://127.0.0.1:2443/login -d "{\"data\": [ \"root\", \"0penBmc\" ] }"
  curl -c cjar -b cjar -k -X GET https://127.0.0.1:2443/xyz/openbmc_project/state/bmc0
  ```

  - Stop the bmcweb service and scp new file over to /tmp/ each time you
  want to retest a change.

  See the [REST](https://github.com/openbmc/docs/blob/master/REST-cheatsheet.md)
  and [Redfish](https://github.com/openbmc/docs/blob/master/REDFISH-cheatsheet.md) cheatsheets for valid commands.



