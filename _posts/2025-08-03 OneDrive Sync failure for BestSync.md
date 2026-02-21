---
layout: post
title: OneDrive BestSync Failure for 2 weeks
date: 2025-08-03
---

## OneDrive BestSync Failure for 2 weeks

* Remote login to virtualBox Windows 7
* BestSync version is 2020, shown BestSync2025 
* In "Services", stop "Borland Socker Server"
* Open BestSync 2020, in "Run as Service", right click the "OneDrive"->View Log, it doesn't show log of current sync.  
* Right click-> Start Task: showing "logging", forever.
* Right click-> Stop Task, just crashing, no responding
* Log off windows and log in again.
* Use Windows Task Manager to stop "BestSync.exe".
* Open "BestSync"->Preview Task:

```log
>POST /common/oauth2/v2.0/token
 400 Bad Request
 invalid_grant
 Get Refresh Token.
Error=invalid_grant
Error_Des=AADSTS70000: The user could not be authenticated as the grant is expired. The user must sign in again. Trace ID: 7f659044-51ef-471f-8164-71585941ab00 Correlation ID: b5c4c523-d3ac-41f2-bd91-675ce44d55c0 Timestamp: 2025-08-03 16:54:57Z(Error response.)
>POST /common/oauth2/v2.0/token
 400 Bad Request
 invalid_grant
 Get Refresh Token.
Error=invalid_grant
Error_Des=AADSTS70000: The user could not be authenticated as the grant is expired. The user must sign in again. Trace ID: 70fdce4f-60c7-409f-a4e4-d4f765a8c300 Correlation ID: 4254e34e-1976-4ccd-9056-6d4f36f2445f Timestamp: 2025-08-03 16:54:58Z(Error response.)
 Unspecified error
```

* Get the OneDrive account information from My Drive > Software > RCG > "RCG Off Site Backup" under google drive "

  * User name: rcgforex@gmail.com
  * password: Microsoft authenticator

* right click -> Modify Task:
  * use "search icon"
  * use OAuth2 Manually
    * when ask enter password, select use mobile app, which is authenticator in mobile phone.
    * approve login in mobile phone.
    * use Edge browser to post the url, and copy the token string

        ```log
        Login to server successfully. Please copy the following code to the clipboard, then paste it to BestSync setting dialogbox. Code=

        M.C519_SN1.2.U.a0863a88-4000-a6fb-960f-cf2bca11ff15
        ```

    * paste token string
    * OK

  * right click -> preview Task

    ```log
    >POST /common/oauth2/v2.0/token
    <200 OK
    >GET /me/drive/items/root/children?$top=900&$select=id,name,size,lastModifiedDateTime,folder,file,fileSystemInfo,remoteItem
    <200 OK
    >GET /me/drive/items/65E1DC5223349C23!191/children?$top=900&$select=id,name,size,lastModifiedDateTime,folder,file,fileSystemInfo,remoteItem
    <200 OK
    >GET /me/drive/items/65E1DC5223349C23!192
    <200 OK
    >GET /me/drive/items/65E1DC5223349C23!192/children?$top=900&$select=id,name,size,lastModifiedDateTime,folder,file,fileSystemInfo,remoteItem
    <200 OK

    ```

  * click "Start" and all back tasks done

* start the "Borland Socker Server"

