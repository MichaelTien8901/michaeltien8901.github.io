---
layout: post
title: "Dell T5600 Need To Press Power Button Twice To Start Windows 10"
date: 2019-04-17
---
## Dell Server Can't Start With Power Button Pressed and Shutdown Script Not Running

   I notice that this Dell T5600 server can't boot to Windows 10 with power button pressed.  The Windows Icon displayed and 
   then server stop.  I need to press second time to start to Windows 10.
   
   This happened when I tried to solve another problem that Windows didn't shutdown VirtualBox serveice properly, even I set up
   shutdown script in Group Policy. The strange thing is "Restart" is working fine.  But "Shutdown" isn't working.
   
## Disable Fast Start Option

   I found the solution of first problem that I need to disable "fast startup" in power option.
   From Control Panel, "Harware and Sound"-> "Power Options" -> System Settings", select Change Settings that are currently unavailable.
   Then I can untick the "Turn on fast startup".  
   
   The description is "This helps start your PC faster after shutdown.  Restart isn't affected".  That give me hint about my second 
   problem that only happened in "Shutdown", not in "Restart".  
   
   So the fast start option might also disable the "Shutdown" script defined in group policy.  
   
 ## Disable Fast Start Option Solve the Shutdown Process problem
 
   Now the service can be stop properly and shutdown script is running.  
