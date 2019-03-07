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








