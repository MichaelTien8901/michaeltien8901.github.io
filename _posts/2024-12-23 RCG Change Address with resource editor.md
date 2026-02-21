# RCG Change Address with Resource Editor

## Install in VM

* in RCGServer7 VM, create a snap shot (Before Resource Editor Install)
* Network Setting with "Host Only" in order to expose server port.
* Start VM
* Install [Resource Editor](https://github.com/katahiromz/RisohEditor/releases/download/5.8.5/RisohEditor-5.8.5-setup.exe) 

* Create Share Folder with Project/2024/RCG2024


## Use XN Resource Editor from [github](https://stefansundin.github.io/xn_resource_editor/)

* TDEALTICKDEPOSITRPTFORM: QRLabel18, QRLabel15
* TDEALTICKREPFORM:
* TFORWARDRECEIPTREPORT:

*These two resource editor compile the resource and make the file difference much bigger.*


## [*Final Choice: ImHex github](https://github.com/WerWolv/ImHex)

* Use `Find` Panel:
  * Find/Binary Pattern: "6081"
* use left side `Content` panel to correct binary directly

## `NSIS` to package installation

* Run `NSIS`
  * load to script for installation of Broker 2024. And it will auto compile it