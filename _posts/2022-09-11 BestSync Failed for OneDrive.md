# Bestsync 2020 Failed for Onedrive

   The error message links to the following pages
 
*  [https://support.microsoft.com/en-us/topic/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-winhttp-in-windows-c4bd73d2-31d7-761e-0178-11268bb10392#bkmk_easy](https://support.microsoft.com/en-us/topic/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-winhttp-in-windows-c4bd73d2-31d7-761e-0178-11268bb10392#bkmk_easy)


* [https://docs.microsoft.com/en-us/sharepoint/troubleshoot/administration/authentication-errors-tls12-support](https://docs.microsoft.com/en-us/sharepoint/troubleshoot/administration/authentication-errors-tls12-support)

* Easy Fix Tool provide by Microsoft [MicrosoftEasyFix51044.msi](https://download.microsoft.com/download/0/6/5/0658B1A7-6D2E-474F-BC2C-D69E5B9E9A68/MicrosoftEasyFix51044.msi)

Instead of changing registry entries manually, the easy fix tool did this for us.  After installed "EasyFix", reboot Windows 7 and Modify the "Task for OneDrive", and update the destination folder in "OneDrive".  

This also fixed the DropBox back up problem, which is also related to the TLS1.0, TLS1.1 expired.  
