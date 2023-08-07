---
layout: post
classes: wide
title:  "Covert access to guest virtual machines through VIX API"
date:   2023-08-07 01:00:00 +0800
--- 
This post provides advanced attack techniques in vSphere infrastructure.

 
## Covert access to guest virtual machines through VIX API

VMware vSphere is the most used virtualization platform. ESXi is an enterprise class, bare-metal hypervisor to create and manage guest virtual machines. ESXi host runs UNIX like operating system, using a VMware proprietary kernel. VMware vCenter is a 
management server to manage multiple ESXi hosts.

ESXi host and vCenter can be managed through Secure Shell(SSH), direct console interface unit(DCUI), Web portal and automation/web application programming interface(API). SSH and shell services are disabled by default.

In addition to automation API to manage vSphere infrastructure, ESXi host use VIX API to manage virtual machines(VM). VIX API is a library for writing scripts and programs to manage guest machines from the ESXi hosts. ESXI host can use VIX API even though the guest VM networking is disabled or the virtual machine is contained. ESXi need to authenticate with guest VMs for successful VIX API calls.

VMware have released software development kit(SDK) for VIX API.
Through VIX API, ESXi host can perform various guest operations methods such as StartProgramInGuest, InitiateFileTransferFromGuest, InitiateFileTransferToGuest.

Threat actors (TA) can abuse VIX API to covertly perform attack techniques over virtual machines. To achieve this attack, TA need to compromise ESXI host root account and VM administrator accounts. The following snapshot displays an example of attack technique to dump password from guest VMs.

VMware Tools versions 10.3.x,11.x.x and versions less than 12.2.4 contains an authentication bypass vulnerability in the vgauth module. A fully compromised ESXi host can force VMware Tools to fail to authenticate host-to-guest operations. Threat Actor can access guest VM without authentication through a compromised ESXi  by exploiting this vulnerability. This vulnerability is tracked as CVE-2023-20867.

###Detections:

1.Event ID 4624 will be recorded in the guest VM Windows operating system on logon type 4

2.EDR events will record write interactions and new processes spawned from vmtoolsd.exe on Windows and from the vmtools daemon on Linux

3.VMware.log file in the guest volume folder records virtual machine-specific activities. This log records guest operations, but additional details are not recorded. For evading detections, TA can stop this logging by adding configuration logging=false in the virtual machines .vmx file. Once the VM is deleted these logs are deleted too.


4.VMware Tools in guest VM use a configuration file called tools.conf to configure different operations such as logging, upgrade. Enable debug logging level for VMwareService (vmsvc) to record and gain visibility over the guest operations performed by TA compromised ESXI host in guest VM. The below mentioned change will record the guest operation activities, but it generates humongous of logs and noisy too

`[logging]
log = true
vmsvc.level = debug
vmsvc.handler = file
vmsvc.data = c:/Windows/Temp/vmsvc.log`


5.VMSVC.log includes operation codes that denote specific VIX commands, that is equivalent to guest operations 

