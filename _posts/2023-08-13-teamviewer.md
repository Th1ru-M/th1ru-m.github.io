---
layout: post
classes: wide
title:  "Investigating TeamViewer log files"
date:   2023-08-13 01:00:00 +0800
--- 
This post provides details on analysing and investigating TeamViewer log files recorded in the machine running in Windows operating system.  

 
### Introduction
TeamViewer is a remote desktop application to manage computers and mobile devices remotely. This offers remote management and file transfer capabilities. System administrators use this application to provide remote support to their users. Threat Actors(TA) can abuse this application to remotely manage the compromised devices to exfiltrate data and install malicious binaries.  

Following log files related to TeamViewer applications are recorded in the directory `C:\Program Files\TeamViewer`.  

1.Connections_incoming  
2.TeamViewer15_Logfile  

Along with the above log files, these are the additional artefacts that are recorded in the Windows machine. The below mentioned log files are recorded in the directory `C:\Users\<User profile>\AppData\Roaming\TeamViewer`.    

1.Connections  
2.MRU directory that includes TVC files  

As an investigator, you can analyse these log files to understand the TA activity.     

### Threat Actor exfiltrate data through TeamViewer application

TA after compromising a machine, TA can install TeamViewer application in the victim machine to transfer data to TA infrastructure.   

![TeamViewer_Attacks](/image/teamviewer/dataextraction.JPG)

In `connections` log, you can identify outgoing connection details, that includes TA TeamViewer ID , connection type(Remotecontrol or File Transfer) and session start/end time.  

![TeamViewer_outgoing](/image/teamviewer/outgoing.JPG)

Analyse`TeamViewer15_Logfile` to identify victim TeamViewer ID, TA TeamViewer ID and Public IP address/domain details of TeamViewer relay server/router.  

![TeamViewer_IP](/image/teamviewer/ipdetails.JPG)

`TeamViewer15_Logfile` will also record data transfer activity. We can see the transferred files and file size. 

![TeamViewer_datatransfer](/image/teamviewer/datatransfer.JPG)

`TeamViewer15_Logfile` will also record details about TeamViewer application used in the victim machine.  


### Threat Actor pushes malicious files through TeamViewer application

TA after compromising a machine, TA can install TeamViewer application in the victim machine to push additional malicious binaries from TA infrastructure.   

![TeamViewer_binary](/image/teamviewer/maliciousbinary.JPG)

`connections_incoming` log will record the incoming TeamViewer connection details. This log will record the TA TeamViewer ID, session start/end time, TA Teamviewer Account name .    
  
![TeamViewer_incoming](/image/teamviewer/incoming.JPG)

`TeamViewer15_Logfile` will record the incoming connection details, that includes TA TeamViewer ID, TeamViewer account name, TeamViewer relay server details, Authentication status.  
   
![TeamViewer_incomingip](/image/teamviewer/incomingipdetails.JPG)

`TeamViewer15_Logfile` will record file downloaded activity, that includes the downloaded file name and size.
   
![TeamViewer_incomingbinary](/image/teamviewer/maliciouspush.JPG)



