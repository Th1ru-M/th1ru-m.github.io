---
layout: post
classes: wide
title:  "PowerPlay: The Dark Side of Power Automate Flows"
date:   2024-12-07 01:00:00 +0800
--- 
This post provides details on how adversaries abuse Power Automate Flows

 
### Introduction
Power Automate, part of the Microsoft Power Platform, is a cloud-based service that helps users automate workflows and tasks across a variety of applications and services. It enables the creation of flows that connect multiple components, allowing actions to be triggered automatically based on specific events.

While Power Automate provides various benefits by automating processes, it also introduces potential security risks when misused. Adversaries can exploit these flows to maintain persistent access and perform data exfiltrations after compromising user accounts in the cloud.


### Threat Scenarios

<u>Email Extractions:</u>

After compromising a user account and credentials, Adversaries can create a workflow to autoforward emails of the compromised account. Whenever a new email arrives, flow will be triggered and execute an action to forward email to threat actor Email ID.

![Autoforward](/image/flows/autoforward.JPG)


<u>Data Extractions:</u>

After compromising a user account and credentials, Adversaries can create a workflow to extract data from the OneDrive of the compromised account to adversaries cloud storage account or services. In the below diagram, When a new file is created which matches the trigger, so flow will be executed with an action to upload a copy of the file to Threat Actors cloud storage Account (Drop box).

![DataExfil](/image/flows/dataexfil.JPG)


### Artefacts Recorded

<u>Email Extractions:</u>

There will be a `CreateFlow` event recorded in the unified audit log in Microsoft 365. This will include the account information that created the flow, which is the compromised acount and the flow details URL.

![createflow](/image/flows/createflow.JPG)


For arrival of all the new emails, the flow will be executed and the email will be autoforwarded to adversaries Email ID. There will be a `Send` operation recorded in the unified audit log and that includes the Client IP range of the Microsoft.


![Send](/image/flows/send.JPG)

<u>Data Extractions:</u>

For creation of any new files in One Drive for the compromised user account, the flow will be executed and the file will be uploaded to adversaries cloud storage account. There will be a `FileDownloaded` operation recorded in the unified audit log and that includes the Client IP range of the Microsoft.


![Download](/image/flows/download.JPG)

### Query the Power Automate Flow

Query Power Automate as Power Automate Administrator and list all the flows created. The below query will list all the flow created in the tenant.

`PS C:\> $flowCollection = @()
Connect-MsolService
$users = Get-MsolUser -All | Select-Object UserPrincipalName, ObjectId
$flows = get-AdminFlow
foreach($flow in $flows){
$flowProperties = $flow.internal.properties
$Creator = $users | where-object{$_.ObjectId -eq $flowProperties.creator.UserID}
$triggers = $flowProperties.definitionsummary.triggers
$actions = $flowProperties.definitionsummary.actions | where-object {$_.swaggerOperationId}
[datetime]$modifiedTime = $flow.LastModifiedTime
[datetime]$createdTime = $flowProperties.createdTime
$flowCollection += new-object psobject -property @{displayName
= $flowProperties.displayName;environment =
$flowProperties.Environment.name;State = $flowProperties.State;Triggers =
$triggers.swaggerOperationId;Actions = $actions.swaggerOperationId;Created = $createdTime.ToString("ddMM-yyyy HH:mm:ss");Modified = $modifiedTime.ToString("dd-MMyyyy HH:mm:ss");CreatedBy = $Creator.userPrincipalName
}
$flowCollection
}`

To extract the entire configuration of the flows, you can leverage PnP powershell module. The output file will include details of the Triggers and Actions created in the flow.

`Export-PnPFlow [-Environment <PowerAutomateEnvironmentPipeBind>] -Identity <PowerAutomateFlowPipeBind>
 [-AsZipPackage] [-PackageDisplayName <String>] [-PackageDescription <String>] [-PackageCreatedBy <String>]
 [-PackageSourceEnvironment <String>] [-OutPath <String>] [-Force] [-Connection <PnPConnection>]`

### Defense Controls

- Configure Tenant Isolation access control in Powerplatform adminstrator portal to control data from moving into or out of the tenant in Microsoft Entra connectors for apps and flows. 
You can configure exception rule for inbound and outbound direction to allow data exchange through flow to happen only within your tenant. This will restrict users to authenticate to external tenants when they create flows.

![TenantIsolation](/image/flows/isolation.JPG)


- When Power Automate flow is used to send or forward emails via connectors like Outlook or other Microsoft-based email services, often the email header includes the `x-ms-mailapplication`. This header helps to identify the application or service that initiated the email. Configure a Transport rule in Exchange online to block emails with the `x-ms-mailapplication` header with value `Microsoft Power Automate` for the mail operation type as send or forward.


 

