# IKEEXT DLL Hijacking Exploit Tool

## Description 
A major weakness is present in Windows Vista, 7, 8, Server 2008, Server 2008 R2 and Server 2012, which allows any authenticated user to gain system privileges under certain circumstances. 

In Windows there is a service called IKEEXT (IKE and AuthIP IPsec Keyring Modules), which runs as SYSTEM and tries to load a DLL that doesn't exist. The default DLL search order includes several system directories and ends by searching into PATH folders. To put it simple, if one of these folders is configured with weak permissions, any authenticated user can plant a malicious DLL to execute code as SYSTEM and thus elevate his/her privileges. 

## Usage 

## Credits 

