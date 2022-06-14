# **Services**

Get-Service

`PS> Get-Service -Name se*`

Status  | Name            |   DisplayName
------ |  ----        |       -----------
Running | seclogon     |      Secondary Logon
Running | SENS         |      System Event Notification
Stopped | ServiceLayer  |     ServiceLayer

### **Stopping, Starting, Suspending, and Restarting Services**
---
 `Stop-Service -Name spooler`

 `Start-Service -Name spooler`

 `Suspend-Service -Name spooler`

 `Restart-Service -Name spooler`


[More](https://docs.microsoft.com/en-us/powershell/scripting/samples/managing-services?view=powershell-7.2)
