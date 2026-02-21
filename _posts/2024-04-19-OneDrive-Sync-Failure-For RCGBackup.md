# OneDrive Sync failure for RCGBackup

## Resolution

* Check BestSync in VirtualBox Windows
  * Show token errors in logs
* Open BestSync (service tab) to find the OneDrive Task
* Edit Task and use OAuth login
  * Failed to login with errors for TLS 1.0, 1.1, 1.2
  * Use manual login
    * Copy URL in manual login and paste to Edge Browser
    * Login in Edge and copy the token
    * paste token into manual login
    * log show "OK".
* Search destination again to find the folder "RCGForexBackup/zip"
* When task edit done, the task will automatically sync up again

