# IKEEXT DLL Hijacking Exploit Tool

This tool is intended for automatically detecting and exploiting the IKE and AuthIP IPsec Keyring Modules Service (IKEEXT) Missing DLL vulnerability.

<p align="center">
  <img src="https://github.com/itm4n/Ikeext-Privesc/raw/master/screenshots/00_ikeext-exploit-video.gif">
</p>

## Description 

A major weakness is present in Windows Vista, 7, 8, Server 2008, Server 2008 R2 and Server 2012, which allows any authenticated user to gain system privileges under certain circumstances. 

In Windows there is a service called IKEEXT (IKE and AuthIP IPsec Keyring Modules), which runs as SYSTEM and tries to load a DLL that doesn't exist. The default DLL search order of Windows includes several system directories and ends with PATH folders. To put it simple, if one of these folders is configured with weak permissions, any authenticated user can plant a malicious DLL to execute code as SYSTEM and thus elevate his/her privileges. 

IKEEXT trying to load 'wlbsctrl.dll':

<p align="center">
  <img src="https://github.com/itm4n/Ikeext-Privesc/raw/master/screenshots/01_procmon-dll-search-order.png">
</p>

## Usage 

This PowerShell script consists of 2 Cmdlets: 
* ___Invoke-IkeextCheck___ - Only checks whether the machine is vulnerable. 
* ___Invoke-IkeextExploit___ - If the machine is vulnerable, exploit it by dropping a specifically crafted DLL to the first weak folder.

### Detection

The ___Invoke-IkeextCheck___ Cmdlet performs the following checks:
- __OS version__ - If the OS is Windows Vista/7/8 then the machine is potentially vulnerable. 
- __IKEEXT status__ and __start type__ - If the service is enabled, the machine is potentially vulnerable (default).
- __PATH folders with weak permissions__ - If at least one folder is found, the machine is potentially vulnerable.
- Is ___wlbsctrl.dll___ already present in some system folder (i.e. a folder where DLLs can be loaded from)? - If _wlbsctrl.dll_ doesn't exist, the machine is potentially vulnerable (default).

Syntax:
```
Invoke-IkeextCheck [-Verbose] 
```

Example: 
```
PS C:\temp> . .\Ikeext-Privesc.ps1
PS C:\temp> Invoke-IkeextCheck -Verbose
```

<p align="center">
  <img src="https://github.com/itm4n/Ikeext-Privesc/raw/master/screenshots/03_ikeextcheck-verbose.png">
</p>

### Exploit 

If the machine is vulnerable, the ___Invoke-IkeextExploit___ Cmdlet will perform the following:

- Depending on the IKEEXT start type: 
    - ___AUTO___ - The exploit files will be dropped to the first weak PATH folder, provided that the switch '-Force' was set in the command line (safety override).
    - ___MANUAL___ - The exploit files will be dropped to the first weak PATH folder. Then, the service start will be triggered by trying to open a dummy VPN connection (using _rasdial_). 

- The following exploit files will be dropped to the first vulnerable folder:
    - ___wlbsctrl.dll___ - A specifically crafted DLL (32 & 64 bits), that starts a new CMD process and execute a BATCH file. 
    - ___wlbsctrl_payload.bat___ - The BATCH file that will be executed. 

- BATCH payload:
    - __Default__ - A default BATCH payload is included in this script. It will use ___net user___ and ___net localgroup___ to create a new user (___hacker___ with password ___SuperP@ss123___) and add it to the local administrators group (the name of the group is automatically retrieved by the script).
    - __Custom__ - A custom set of commands can be specified using the parameter ___-Payload .\Path\To\File.txt___ and a file containing one command per line. 
- Log file - Each payload command's ouput is redirected to a log file located in the same folder as the DLL's. Its name will be something like ___wlbsctrl_xxxxxxxx.txt___. So if this file is not created, it means that the payload was not executed. 

Syntax:
```
Invoke-IkeextExploit [-Verbose] [-Force] [[-Payload] <String>]
```

Example:
```
PS C:\temp> . .\Ikeext-Privesc.ps1
PS C:\temp> Invoke-IkeextExploit
```

<p align="center">
  <img src="https://github.com/itm4n/Ikeext-Privesc/raw/master/screenshots/04_ikeextexploit.PNG">
</p>

## Credits 

High-Tech Bridge - Frédéric Bourla, who initially disovered the vulnerability and disclosed it on October 9, 2012. - https://www.htbridge.com/advisory/HTB23108 

## Remediation 

First of all, it's important to note that this vulnerability was patched in Windows 8.1 and above. In these versions of Windows, ___wlbsctrl.dll___ search is limited to __C:\\Windows\\System32\\__. 

If you're stuck with Windows 7 / Server 2008R2 because of compatibility issues for example, several counter measures can be applied:
1) __PATH folders__ with weak permissions - Some applications are installed directly in __C:\\__ and add themselves to the PATH environment variable. By default, folders created in __C:\\__ are writable by any authenticated user so make sure to drop these privileges. 
2) __Disable IKEEXT__ - In most cases, IKEEXT could simply be disabled by applying a GPO. This can be a good solution for servers but it is not advised for workstations. 
3) __Deploy you own DLL__ - Deploying a dummy ___wlbsctrl.dll___ in __C:\\Windows\\System32\\__ for example is an efficient solution since this directory has a higher priority than PATH folders. 

