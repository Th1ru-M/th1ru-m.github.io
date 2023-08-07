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

ESXi host and vCenter can be managed through Secure Shell(SSH), direct console interface unit(DCUI), Web portal, automation and web application programming interface(API). SSH and shell services are disabled by default. In addition to automation API to manage vSphere infrastructure, ESXi host use VIX API to manage virtual machines(VM). VIX API is a library for writing scripts and programs to manage guest machines from the ESXi hosts. ESXI host can use VIX API even though the guest VM networking is disabled or the virtual machine is contained. ESXi need to authenticate with guest VMs for successful VIX API calls.

VMware have released software development kit(SDK) for VIX API. Through VIX API, ESXi host can perform various guest operations methods such as StartProgramInGuest, InitiateFileTransferFromGuest, InitiateFileTransferToGuest.
