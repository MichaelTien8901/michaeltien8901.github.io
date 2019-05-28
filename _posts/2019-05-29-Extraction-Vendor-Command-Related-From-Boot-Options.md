---
layout: post
title: "Extraction Vendor Command From FX3 Boot Options Document"
date: 2019-05-29
---

The following extracted from document [Z-USB FX3 Boot Options]([http://eng.eewiki.net/app/db_page/get_file.php?docid=55346]).

### USB Setup Packet

   The FX3 bootloader decodes the SETUP packet that
   contains an 8-byte data structure defined in Table 4.

   Table 4. Setup Packet

   | Byte |  Field         | Description  |
   |------|----------------|--------------|
   |  0   | bmRequestType  | Request type: Bit7: Direction, Bit6–0: Recipient |
   |  1   | bRequest       | This byte will be A0h for firmware download/upload vendor command. |
   | 2-3  | wValue         | 16-bit value (little-endian format)  |
   | 4-5  | wIndex         | 16-bit value (little-endian format)  |
   | 6-7  | wLength        | Number of bytes                      |

* Note Refer to the USB 2.0 Specification for the bitwise explanation.

### USB Chapter 9 and Vendor Command

   The FX3 bootloader handles the commands in Table 5.

   Table 5. USB Commands

| bRequest | Descriptions                     |
|----------|----------------------------------|
| 00       | GetStatus: Device, Endpoints, and Interface
| 01 | ClearFeature: Device, Endpoints
| 02 | Reserved: Returns STALL
| 03 | SetFeature: Device, Endpoints
| 04 | Reserved: Returns STALL
| 05 | SetAddress: Handle in FX3 hardware
| 06 | GetDescriptor: Devices’ descriptors in ROM
| 07 | Reserved: Returns STALL
| 08h| GetConfiguration: Returns internal value
| 09h| SetConfiguration: Sets internal value
| 0Ah| GetInterface:Rreturns internal value
| 0Bh| SetInterface: Sets internal value
| 0Ch| Reserved: Returns STALL
| 20h-9Fh | Reserved: Returns STALL
| A0h| Vendor Commands: Firmware upload/download and so on
| A1h-FFh Reserved: Returns STALL

### USB Vendor Command

   The bootloader supports the A0h vendor command for
   firmware download and upload. The fields for the
   command are shown in Table 6 and Table 7

   Table 6. Command Fields for Firmware Download

| Byte | Field | Value | Description |
|------|-------|-------|-------------|
| 0 | bmRequest Type | 40h | Request type: Bit7: Direction Bit6-0: Recipient.
| 1 | bRequest       | A0h | This byte will be A0 for firmware download/upload vendor command.
| 2-3 | wValue       | AddrL(LSB) | 16-bit value (little endian format)
| 4-5 | wIndex       | AddrH(MSB) | 16-bit value (little endian format)
| 6-7 | wLength      | Count      | Number of bytes

   Table 7. Command Fields for Firmware Upload
| Byte | Field | Value | Description |
|------|-------|-------|-------------|
| 0    | bmRequest Type | C0h | Request type: Bit7: Direction Bit6-0: Recipient.
| 1    | bRequest       | A0h | This byte will be A0 for firmware download/upload vendor command.
| 2-3 | wValue       | AddrL(LSB) | 16-bit value (little endian format)
| 4-5 | wIndex       | AddrH(MSB) | 16-bit value (little endian format)
| 6-7 | wLength      | Count      | Number of bytes

 Table 8. Command Fields for Transfer of Execution to
Program Entry

| Byte | Field | Value | Description |
|------|-------|-------|-------------|
| 0    | bmRequest Type | 40h        |Request type: Bit7: Direction Bit6-0: Recipient
| 1    | bRequest       | A0h        | This byte will be A0 for firmware download/upload vendor command.
| 2-3  | wValue         | AddrL(LSB) | 32-bit Program Entry
| 4-5  | wIndex         | AddrH(MSB) | 32-bit Program Entry>>16
| 6-7  | wLength        |  0         | This field must be zero.

 * In the transfer execution entry command, the bootloader
   will turn off all the interrupts and disconnect the USB.

### Examples 

Three examples of vendor command subroutines follow.

1. Example 1. Vendor Command Write Data Protocol With 8-Byte Setup Packet
```
bmRequestType=0x40
bRequest = 0xA0;
wValue = (WORD)address;
wIndex = (WORD)(address>>16);
wLength = 1 to 4K-byte max
```
This command will send DATA OUT packets with a length of transfer equal to wLength and a DATA IN Zero length packet.

2. Example 2. Reading Bootloader Revision With Setup Packet
```
bmRequestType= 0xC0
bRequest = 0xA0;
wValue = (WORD)0x0020;
wIndex = (WORD)0xFFFF;
wLength = 4
```
This command will issue DATA IN packets with a length of transfer equal to wLength and a DATA OUT Zero length packet.

3. Example 3. Jump to Program Entry With 8-Byte Setup Packet (refer to Table 8.)
```
bmRequestType= 0x40
bRequest = 0xA0;
wValue = Program Entry (16-bit LSB)
wIndex = Program Entry >>16 (16-bit MSB)
wLength = 0
```

### USB Device Descriptor for Vendor Command

Table 12. Interface Descriptor (Alt. Setting 0)

| Offset | Field   | Value | Description |
|--------|---------|-------|-------------|
| 0      | bLength | 09h   | Length of this descriptor = 10 bytes(??)
| 1      | bDescType | 04h | Descriptor type = Interface
| 2      | bInterfaceNum | 00h | Zero-based index of this interface = 0
| 4      | bAltSetting   | 00  |Alternative Setting value = 0
| 5      | bNumEndpoints | 00  | Only endpoint0
| 6      | bInterfaceClass | FFh | Vendor Command Class
| 7      | bInterfaceSubClass | 00h |
| 8      | bInterfaceProtocol | 00h
| 9      | iInterface         | 00h | None
