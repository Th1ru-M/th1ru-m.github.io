---
layout: post
classes: wide
title:  "Modern attack techniques in vSphere Infrastructure"
date:   2023-07-18 01:00:00 +0800
--- 
This post provides advanced attack techniques in vSphere infrastructure.

 
## Modern attack techniques in vSphere Infrastructure

### Covert access to guest virtual machines through VIX API

VMware vSphere is the most used virtualization platform. ESXi is an enterprise class, bare-metal hypervisor to create and manage guest virtual machines. ESXi host runs UNIX like operating system, using a VMware proprietary kernel. VMware vCenter is a 
management server to manage multiple ESXi hosts.

ESXi host and vCenter can be managed through Secure Shell(SSH), direct console interface unit(DCUI), Web portal and automation/web application programming interface(API). SSH and shell services are disabled by default.

In addition to automation API to manage vSphere infrastructure, ESXi host use VIX API to manage virtual machines(VM). VIX API is a library for writing scripts and programs to manage guest machines from the ESXi hosts. ESXI host can use VIX API even though the guest VM networking is disabled or the virtual machine is contained. ESXi need to authenticate with guest VMs for successful VIX API calls.

VMware have released software development kit(SDK) for VIX API.
Through VIX API, ESXi host can perform various guest operations methods such as StartProgramInGuest, InitiateFileTransferFromGuest, InitiateFileTransferToGuest.

![Guest_Operations](/image/esxi/guestops.JPG)

Threat actors (TA) can abuse VIX API to covertly perform attack techniques over virtual machines. To achieve this attack, TA need to compromise ESXI host root account and VM administrator accounts. The following snapshot displays an example of attack technique to dump password from guest VMs.

![VIXAPI_Attacks](/image/esxi/vixattack.JPG)

VMware Tools versions 10.3.x,11.x.x and versions less than 12.2.4 contains an authentication bypass vulnerability in the vgauth module. A fully compromised ESXi host can force VMware Tools to fail to authenticate host-to-guest operations. Threat Actor can access guest VM without authentication through a compromised ESXi  by exploiting this vulnerability. This vulnerability is tracked as CVE-2023-20867.

Detection:

1. Event ID 4624 will be recorded in the guest VM Windows operating system on logon type 4

2.EDR events will record write interactions and new processes spawned from vmtoolsd.exe on Windows and from the vmtools daemon on Linux

3.VMware.log file in the guest volume folder records virtual machine-specific activities. This log records guest operations, but additional details are not recorded. For evading detections, TA can stop this logging by adding configuration logging=false�in the virtual machines�.vmx�file. Once the VM is deleted these logs are deleted too.

![VMware_logs](/image/esxi/vmwarelogs.JPG)

4.VMware Tools in guest VM use a configuration file called tools.conf to configure different operations such as logging, upgrade. Enable debug logging level for VMwareService (vmsvc) to record and gain visibility over the guest operations performed by TA compromised ESXI host in guest VM. The below mentioned change will record the guest operation activities, but it generates humongous of logs and noisy too

`[logging]
log = true
vmsvc.level = debug
vmsvc.handler = file
vmsvc.data = c:/Windows/Temp/vmsvc.log`

![VMSVC_logs](/image/esxi/vmsvc.JPG)

5.VMSVC.log includes operation codes that denote specific VIX commands, that is equivalent to guest operations 

![VIXOperations_Code](/image/esxi/operationcodes.JPG)


### Covert access to ESXi host through vCenter

VMware vCenter is a management server to manage multiple ESXi hosts. Once ESXI host is integrated with vCenter, then vCenter will have administrative permissions over ESXI host. vCenter will create user account called VPXUSER in ESXi host. The password of this account VPXUSER will be stored in vCenter and rotated every 30 days once in an automated fashion. This account has administrative permission in ESXi host. vCenter will use this account and its credential to manage ESXi host.

TA can compromise vCenter servers and extract the password of the account VPXUSER. With the compromised account, TA can have a covert access to ESXi host through SSH.

VPXUSER account password will be stored in an encrypted format in vPostgreSQL database in vCenter. The key for the encrypted password is also stored in a file symkey.dat in vCenter. TA can leverage publicly available tools to decrypt the password to gain access to ESXi host. The below snapshot explains the TA attack flow to extract VPXUSER account password. In order to achieve this technique, TA need to compromise root access to vCenter.


![VPXUser_Compromise](/image/esxi/vpxuserattack.JPG)

Detection:

1.PostgreSQL logging is enabled by default and configured in the file /storage/db/vpostgres/postgresql.conf. PostgreSQL logs record connections initiated to the database in vCenter.
The SQL statements that are queried within the database is not recorded in the logs in the default configuration settings

![postgresql_logs](/image/esxi/postgresqllogs.JPG)

2.Modify PostgreSQL.conf file and enable all logging levels to record all the queries executed within database. This change generates humongous of logs and very noisy

`root@localhost [ ~ ]# cat /storage/db/vpostgres/postgresql.conf | grep log_statement
log_statement = 'all'   # none, ddl, mod, all`


Refer below link for details,
https://www.mandiant.com/resources/blog/vmware-esxi-zero-day-bypass
https://github.com/jas502n/VcenterExsi_PwdDecrypt
https://www.vmware.com/products/beta/ws/vixapi20/ReferenceGuide/tasks.html