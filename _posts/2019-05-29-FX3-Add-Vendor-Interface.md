---
layout: post
title: "FX3 Add Vendor Interface"
date: 2019-05-29
---

Cypress Community has official document [Vendor Interface in UVC] (https://community.cypress.com/docs/DOC-10532).

Summary as the following

### USB Descriptor Update

1. ***Update Length of Descriptor***

   Update the length of the descriptor and all sub-descriptor fields from D9 to E2 (in CyFxUSBSSConfigDscr) and CD to D6 (in CyFxUSBHSConfigDscr).
  
2. ***Increment of the interface***

   Change the number of interfaces from 2 to 3. 
  
3. ***Add Interface Descriptor For Vendor Command***

   After the Endpoint Descriptor for BULK Streaming Video Data (in CyFxUSBHSConfigDscr) and Super Speed Endpoint Companion Descriptor for BULK endpoint (in CyFxUSBSSConfigDscr), include the following:    

```c
     0x09,                           /* Descriptor size */
     CY_U3P_USB_INTRFC_DESCR,        /* Interface descriptor type */
     0x02,                           /* Interface number */
     0x00,                         /* Alternate setting number */
     0x00,                           /* Number of end points */
     0xFF,                           /* Interface class */
     0x00,                           /* Interface sub class */
     0x00,                           /* Interface protocol code */
     0x00                            /* Interface descriptor string index */
```
  ### Setup Callback Function 
  In the CyFxUVCApplnUSBSetupCB() function of the uvc.c file, before the switch case for UVC Class Requests include the following template to handle vendor requests: 
  ```c
     if ((bmReqType & CY_U3P_USB_TYPE_MASK) == CY_U3P_USB_VENDOR_RQT)
        {
           switch(bRequest)
           {
                 case 0x76:
                     CyU3PDebugPrint(2, "Vendor command received…\n");
                                  /* Implement the code as per the functionality required*/
                        CyU3PUsbAckSetup();
                     return CyTrue;
                 default:
                     return CyFalse;
           }
       }
 ```