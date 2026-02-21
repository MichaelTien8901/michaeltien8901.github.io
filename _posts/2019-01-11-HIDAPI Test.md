---
layout: post
title: "HIDAPI Test"
date: 2019-01-11
---
HIDAPI 
To fix error
'error while loading shared libraries: libhidapi-hidraw.so.0: cannot open shared object file: No such file or directory'

with
```bash
sudo apt install libhidapi-libusb0
```

u2f HIDTest compile, need to add openssl library
```bash
sudo apt-get install libssl-dev
```
sudo apt-get install libusb-1.0-0-dev


Bus 002 Device 008: ID 1050:0120 Yubico.com Yubikey Touch U2F Security Key
Couldn't open device, some information will be missing
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 (Defined at Interface level)
  bDeviceSubClass         0 
  bDeviceProtocol         0 
  bMaxPacketSize0        64
  idVendor           0x1050 Yubico.com
  idProduct          0x0120 Yubikey Touch U2F Security Key
  bcdDevice            5.02
  iManufacturer           1 
  iProduct                2 
  iSerial                 0 
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength           41
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          0 
    bmAttributes         0x80
      (Bus Powered)
    MaxPower               30mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           2
      bInterfaceClass         3 Human Interface Device
      bInterfaceSubClass      0 No Subclass
      bInterfaceProtocol      0 None
      iInterface              0 
        HID Device Descriptor:
          bLength                 9
          bDescriptorType        33
          bcdHID               1.10
          bCountryCode            0 Not supported
          bNumDescriptors         1
          bDescriptorType        34 Report
          wDescriptorLength      34
         Report Descriptors: 
           ** UNAVAILABLE **
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x04  EP 4 OUT
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               2
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x84  EP 4 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               2


root@mtien-VirtualBox:~/usbhid-dump# sudo usbhid-dump -a2:10 -i0 | grep -v : | xxd -r -p | hidrd-convert -o spec
Usage Page (F1D0h),         ; F1D0h, reserved
Usage (01h),
Collection (Application),
    Usage (20h),
    Logical Minimum (0),
    Logical Maximum (255),
    Report Size (8),
    Report Count (64),
    Input (Variable),
    Usage (21h),
    Logical Minimum (0),
    Logical Maximum (255),
    Report Size (8),
    Report Count (64),
    Output (Variable),
End Collection

YMSG STM32
root@mtien-VirtualBox:~/usbhid-dump# sudo usbhid-dump -a2:11 -i0 | grep -v : | xxd -r -p | hidrd-convert -o spec
Usage Page (F1D0h),         ; F1D0h, reserved
Usage (01h),
Collection (Application),
    Usage (20h),
    Logical Minimum (0),
    Logical Maximum (255),
    Report Size (8),
    Report Count (64),
    Input (Variable),
    Usage (21h),
    Logical Minimum (0),
    Logical Maximum (255),
    Report Size (8),
    Report Count (64),
    Output (Variable),
End Collection
