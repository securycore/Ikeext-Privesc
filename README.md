# IKEEXT DLL Hijacking Exploit Tool

This tool is intended for automatically detecting and exploiting the IKE and AuthIP IPsec Keyring Modules Service (IKEEXT) Missing DLL vulnerability. 

## Description 

A major weakness is present in Windows Vista, 7, 8, Server 2008, Server 2008 R2 and Server 2012, which allows any authenticated user to gain system privileges under certain circumstances. 

In Windows there is a service called IKEEXT (IKE and AuthIP IPsec Keyring Modules), which runs as SYSTEM and tries to load a DLL that doesn't exist. The default DLL search order of Windows includes several system directories and ends by searching into PATH folders. To put it simple, if one of these folders is configured with weak permissions, any authenticated user can plant a malicious DLL to execute code as SYSTEM and thus elevate his/her privileges. 

IKEEXT trying to load 'wlbsctrl.dll':

![Procmon_DLL_Search_Order](https://github.com/itm4n/Ikeext-Privesc/raw/master/screenshots/01_procmon-dll-search-order.png)

## Usage 

This PowerShell script consists of 2 Cmdlets: 
* ___Invoke-IkeextCheck___ - Only checks whether the machine is vulnerable. 
* ___Invoke-IkeextExploit___ - If the machine is vulnerable, exploit it by dropping a specifically crafted DLL to the first weak folder.

### Detection

The ___Invoke-IkeextCheck___ Cmdlet performs the following checks:
- __OS version__ - If the OS is Windows Vista/7/8 then the machine is potentially vulnerable. 
- __IKEEXT status__ and __start type__ - If the service is enabled, the machine is potentially vulnerable (default).
- __PATH folders with weak permissions__ - If at least one folder is found, the machine is potentially vulnerable.
- Is ___wlbsctrl.dll___ already present in some system folder (i.e. a folder where DLLs can be loaded from)? - If 'wlbsctrl.dll' doesn't exist, the machine is potentially vulnerable (default).

Syntax:
```
Invoke-IkeextCheck [-Verbose] 
```

Example: 
```
PS C:\temp> . .\Ikeext-Privesc.ps1
PS C:\temp> Invoke-IkeextCheck -Verbose
```

![InvokeIkeext-Check_Verbose](https://github.com/itm4n/Ikeext-Privesc/raw/master/screenshots/03_ikeextcheck-verbose.png)

### Exploit 

If the machine is vulnerable, the ___Invoke-IkeextExploit___ Cmdlet will perform the following:

- Depending on the IKEEXT start type: 
    - ___AUTO___ - The exploit files will be dropped to the first weak PATH folder, provided that the switch '-Force' was set in the command line (safety override).
    - ___MANUAL___ - The exploit files will be dropped to the first weak PATH folder. Then, the service start will be triggered by trying to open a dummy VPN connection (using rasdial). 

- The following exploit files will be dropped to the first vulnerable folder:
    - ___wlbsctrl.dll___ - A specifically crafted DLL (32 & 64 bits), that starts a new CMD process and execute a BATCH file. 
    - ___wlbsctrl_payload.bat___ - The BATCH file that will be executed. 

- BATCH payload:
    - __Default__ - A default BATCH payload is included in this script. It will use ___net user___ and ___net localgroup___ to create a new user ('hacker' with password 'SuperP@ss123') and add it to the local administrators group (the name of the group is automatically retrieved by the script).
    - __Custom__ - A custom set of commands can be specified using the parameter ___-Payload .\Path\To\File.txt___ and a file containing one command per line. 

Syntax:
```
Invoke-IkeextExploit [-Verbose] [-Force] [[-Payload] <String>]
```

Example:
```
PS C:\temp> . .\Ikeext-Privesc.ps1
PS C:\temp> Invoke-IkeextExploit
```

![InvokeIkeext-Exploit](https://github.com/itm4n/Ikeext-Privesc/raw/master/screenshots/04_ikeextexploit.PNG)

## Credits 

High-Tech Bridge - Frédéric Bourla, who initially disovered the vulnerability and disclosed it on October 9, 2012. - https://www.htbridge.com/advisory/HTB23108 

