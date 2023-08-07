---
layout: post
classes: wide
title:  "Defending VMware vSphere from ransomware and other sophisticated attacks"
date:   2023-08-08 01:00:00 +0800
--- 
This post provides detail on defense controls to secure VMware vSphere infrastructure from ransomware and other sophisticated attack

 
## Defending VMware vSphere from ransomware and other sophisticated attacks

### Restrict Services/Ports and limit Administrative Access

- Ensure the firewall rules are configured to restrict access to ESXi host and vCenter from limited approved IP addresses    
- Disable SSH and Shell Access to ESXi host and vCenter    
- Isolate the management interface of vCenter, ESXi, vRealize, vMotion and vSAN interfaces to a restricted VLANS    
- Implement privileged access workstations for administrators to administratively access ESXi host and vCenter    
- Disable Internet access for vSphere infrastructure    

### Restrict Malicious VIB execution in ESXi host

- Enable UEFI SecureBoot in ESXi host to restrict loading of unsigned VIB files during the booting process  
- SecureBoot prevents changing of the VIB acceptance level settings in ESXI host  
- Restrict forceful VIB installations option in ESXi host  
- Query ESXi host to identify unsigned VIBs  
 `esxcli software vib signature verify`  
- Query all the ESXi hosts at scale to identify unsigned VIBs. Download the PowerCLI script from Vmware Verify_ESXi_VIB_Signature.ps1 and run against your vCenter using the SSO admin credentials  
  https://kb.vmware.com/sfc/servlet.shepherd/version/download/0685G00000vh19IQAQ  

### Restrict execution of non-VIB binaries in ESXi host

- Enable execInstalledOnly setting through kernel option in ESXi host to restrict non-VIB binaries such as ELF binary file, Shell scripts. Typically, ransomware files are in ELF format  
  `esxcli system settings kernel set -s execinstalledonly -v TRUE`  
- To view the status of execInstalledOnly settings  
  `esxcli system settings kernel list -o execinstalledonly`  
- Restriction activity of non-VIB binary execution is recorded in log files vobd.log,vmkernel.log and vmkwarning.log  
- execInstalledOnly settings will not prevent execution of python based binary file. Python based ransomware or malicious files can circumvent this control  
- From ESXi 8.x onwards, execInstalledOnly settings need to be enabled through kernel and runtime option  
- execInstalledOnly settings in runtime option is enabled by default, but can be disabled without a reboot    

### Enable Trusted Platform Module (TPM) in ESXi host

- In vSphere 7.0U2 and newer, the archived on-disk ESXi configuration file is encrypted and protected from tamper. As a result, attackers cannot read or alter this file, even if they have physical access to the ESXi host's storage  
- ESXi host use Trusted Platform Module(TPM) or Key Derivation Function (KDF) to encrypt configuration file  
- TPM 2.0 chip manages the encryption key and seals the configurations  
- TPM can use Platform Configuration Register (PCR) measurements to implement policies that restrict unauthorized access to sensitive data  
- TPM can send ESXi host attestation report to vCenter  
- To enable TPM mode in ESXi host  
  `esxcli system settings encryption set --mode=TPM`  
- Apply enforcement of execInstalledonly and secureboot settings in ESXi host. TPM will enforce these setting during the booting process  
  `esxcli system settings encryption set --require-secure-boot=T`  
  `esxcli system settings encryption set --require-exec-installed-only=T`  

### Enable Lockdown mode in ESXi host

- Enable Lockdown mode to enforce all the ESXI host operations only to be performed from vCenter  
- Normal Lockdown Mode will enforce ESXi host accessible only through console (DCUI) and vCenter  
- Strict Lockdown mode will enforce ESXi host accessible only through vCenter  
- Exception users can be created to not lose their permissions when the host enters lockdown mode   
- VMware recommends not to add administrators under exception list  

![Lockdown_Mode](/image/esxi/lockdownmode.JPG)

### Password management and perform multifactor authentication

- Enforce Unique and strong passwords for all local accounts in ESXi host and vCenter  
- Secure the Root Account password in Password Vault or PAM tools and automate the password rotation policy  
- Review and limit privileged accounts to manage ESXi host and vCenter  
- If in case ESXi joins active directory domain, then all management of ESXi should happen from vCenter  
- Change the name of the ESXAdmin group in Active Directory to avoid ESXi Administrator membership exposures  
- Enforce and perform multi-factor authentication (MFA) for access to vCenter portal and vSphere API calls  
- Consider using a dedicated identity solution for vSphere infrastructure  

### Centralized Monitoring and Robust Backup Solutions

- Collect and monitor below mentioned logs from ESXi host in SIEM solutions  
![ESXi_logs](/image/esxi/esxilogs.JPG)  
- Collect and monitor below mentioned logs from vCenter server in SIEM solutions  
![vCenter_logs](/image/esxi/vcenterlogs.JPG)  
- Implement robust and secure backup solutions  
- Implement Immutable backups(write once and read many) in an air gapped network to avoid backup file encryptions or deletions  
- Perform multi factor authentication to access backup files  

### Restrict VIX API Guest Operations

- Consider disabling VIX API Guest operations in the guest VM configuration to restrict ESXi host to perform guest operations in VMs  
- This may impact VMware Consolidated Backup (VCB) and VMware Update Manager (VUM), both of which call the VIX API for guest operations  
  `/etc/vmware/config`  
  `guest.commands.enabled = "FALSE"`  
- Create custom roles to limit the privileges to perform guest operation such as executions  

### Perform YARA scan against ESXi host
- Perform Yara Scans against all ESXI hosts to look for malicious artefacts 
- Mount ESXi host in any of the Linux machine using utilities such as sshfs  
- Create Yara rules and specify the identified malicious artefacts such as strings, filenames, paths, file hash and scan the Mounted ESXi directory and identify the impacted ESXi hosts  
  `sshfs -o allow_other,default_permissions root@<ESXi Host IP Address>:/ /mnt/esxi`  
  `yara <rules> -r /mnt/esxi`  
