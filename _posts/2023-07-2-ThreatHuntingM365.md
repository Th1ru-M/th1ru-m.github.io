---
layout: post
classes: wide
title:  "Threat Hunting in Microsoft 365 Environment"
date:   2023-07-2 01:00:00 +0800
--- 
This post provides details about the queries that can be used to hunt various attack techniques in Microsoft 365 environment.

 
## Queries to Threat Hunt in Microsoft 365 Environment


### 1.Hunt for Mailboxes configured with Email Forwarding

Query Exchange Online Configuration:

`Get-Mailbox -ResultSize Unlimited | Where-Object {($Null -ne $_.ForwardingSmtpAddress)} | Select Identity,Name,ForwardingSmtpAddress`

Query Unified Audit Logs:

`$logs = Search-UnifiedAuditLog -Operations set-mailbox -StartDate 2022-01-01 -EndDate 2022-06-30
ForEach ($record in $logs){
    $AuditData = $record.AuditData | ConvertFrom-Json
    if ( $AuditData.Parameters | Where-Object Name -eq 'forwardingsmtpaddress' ) 
    {$record}}` 

### 2.Hunt for Mailboxes configured with Inbox Rules to Auto Email Forward

Query Exchange Online Configuration:

`$Mailboxes = Get-Mailbox ; foreach ($Mailbox in $Mailboxes) { Get-InboxRule -mailbox $Mailbox.Name | Where-Object {($Null -ne $_.ForwardTo) -or ($Null -ne $_.RedirectTo) -or ($Null -ne $_.ForwardAsAttachmentTo) } | select-object identity,Name,Enabled,ForwardAsAttachmentTo,ForwardTo,RedirectTo }`

Query Unified Audit Logs:

`$logs = Search-UnifiedAuditLog -operations new-inboxrule,set-inboxrule -StartDate 2022-01-01 -EndDate 2022-07-08
ForEach ($record in $logs){
    $AuditData = $record.AuditData | ConvertFrom-Json
    if ( $AuditData.Parameters | Where-Object {($_.Name -like 'ForwardTo') -or ($_.Name -eq 'RedirectTo') -or ($_.Name -eq 'ForwardAsAttachmentTo')}) 
    {$record}} `

### 3.Hunt for Transport Rules configured with Auto Email Forward

Query Exchange Online Configuration:

`Get-TransportRule | where-object{($Null -ne $_.BlindCopyTo)}`

Query Admin Audit Log:

`Search-AdminAuditLog -Cmdlets New-TransportRule,Set-TransportRule -parameter BlindCopyTo`

Query Unified Audit Logs:

`Search-UnifiedAuditLog -Operations New-TransportRule, Set-TransportRule -StartDate 2022-01-01 -EndDate 2022-06-30`
 
### 4.Hunt for mailboxes configured with Full Access Delegation Settings

Query Exchange Online Configuration:

`Get-Mailbox -Resultsize Unlimited | Get-MailboxPermission | Where-Object { ($_.Accessrights -like "FullAccess")}`

Query Unified Audit Logs:

`$logs = Search-UnifiedAuditLog -operations add-mailboxpermission -StartDate 2022-01-01 -EndDate 2022-07-08
ForEach ($record in $logs){
    $AuditData = $record.AuditData | ConvertFrom-Json
    if ( $AuditData.Parameters | Where-Object {($_.Value -eq 'FullAccess')}) 
{$record}}`
    
### 5.Hunt for mailboxes configured with SendAs Delegation Settings

Query Exchange Online Configuration: 

`Get-Mailbox -Resultsize Unlimited | Get-RecipientPermission | where-Object { ($_.Accessrights -like "SendAs")}` 
  
Query Unified Audit Logs:

`$logs = Search-UnifiedAuditLog -operations Add-RecipientPermission -StartDate 2022-01-01 -EndDate 2022-07-20
  ForEach ($record in $logs){
    $AuditData = $record.AuditData | ConvertFrom-Json
    if ( $AuditData.Parameters | Where-Object {($_.Value -eq 'SendAs')}) {$record}}` 

### 6.Hunt for mailboxes configured with Mailbox Folder Permissions

Query Exchange Online Configuration: 

`$mailboxes = Get-Mailbox -ResultSize Unlimited
ForEach ($record in $logs){
$AuditData = $record.AuditData | ConvertFrom-Json
if ( $AuditData.Parameters | Where-Object {($_.Value -like 'Anonymous') -or ($_.Value -eq 'Default') }) {$record}}`

Query Unified Audit Logs:

`$logs = Search-UnifiedAuditLog -operations add-MailboxFolderPermission,Set-MailboxFolderPermission -StartDate 2022-01-01 -EndDate 2022-07-08
ForEach ($record in $logs){
    $AuditData = $record.AuditData | ConvertFrom-Json
    if ( $AuditData.Parameters | Where-Object {($_.Value -like ''Anonymous'') -or ($_.Value -eq 'Default') }) {$record}}` 

### 7.List all the flows. Look for email autoforward and data extraction flows.

Query Power Automate:

`$flowCollection = @()
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
    $flowCollection += new-object psobject -property @{displayName = $flowProperties.displayName;environment = $flowProperties.Environment.name;State = $flowProperties.State;Triggers = $triggers.swaggerOperationId;Actions = $actions.swaggerOperationId;Created = $createdTime.ToString("dd-MM-yyyy HH:mm:ss");Modified = $modifiedTime.ToString("dd-MM-yyyy HH:mm:ss");CreatedBy = $Creator.userPrincipalName
}
    $flowCollection
}`

### 8.Hunt for identities with Application Impersonation Role assigned
 
Query Azure AD Tenant:

`$AppImperGroups = Get-RoleGroup | Where-Object Roles -like ApplicationImpersonation
ForEach ($Group in $AppImperGroups)
{
 Get-RoleGroupMember $Group.Name
 }`

`Get-ManagementRoleAssignment -Role ApplicationImpersonation`

Query Unified Audit Logs:

`$logs = Search-UnifiedAuditLog -operations 'New-RoleGroup, New-ManagementRoleAssignment,set-ManagementRoleAssignment'  -StartDate 2022-01-01 -EndDate 2022-07-08
ForEach ($record in $logs){
$AuditData = $record.AuditData | ConvertFrom-Json
if ( $AuditData.Parameters | Where-Object {($_.Value -like 'ApplicationImpersonation')})
{$record}}`

### 9.Hunt for Service principals and their OAuth permission Grants

Query Azure AD Tenant:

`Get-AzureADServicePrincipal  | ForEach-Object{
$spn = $_;
$objID = $spn.ObjectID;
$grants = Get-AzureADServicePrincipalOAuth2PermissionGrant -ObjectId $objID;
foreach ($grant in $grants)
{
$user = Get-AzureADUser -ObjectId $grant.PrincipalId;
$OAuthGrant = New-Object PSObject;
$OAuthGrant | Add-Member Noteproperty 'ObjectID' $grant.objectId;
$OAuthGrant | Add-Member Noteproperty 'User' $user.UserPrincipalName;
$OAuthGrant | Add-Member Noteproperty 'AppDisplayName' $spn.DisplayName;
$OAuthGrant | Add-Member Noteproperty 'AppPublisherName' $spn.PublisherName;
$OAuthGrant | Add-Member Noteproperty 'AppReplyURLs' $spn.ReplyUrls;
$OAuthGrant | Add-Member Noteproperty 'GrantConsentType' $grant.consentType;
$OAuthGrant | Add-Member Noteproperty 'GrantScopes' $grant.scope;
}
Write-Output $OAuthGrant
}`

### 10.Hunt for Anonymous Link Created/Updated in Sharepoint Online

Query Unified Audit Logs:

`Search-UnifiedAuditLog -recordtype SharePointSharingOperation -operations 'anonymouslinkcreated,anonymouslinkupdated' -startdate 2022-07-30 -enddate 2022-08-01`

### 11. Hunt for Anonymous Link usage in Sharepoint Online

Query Unified Audit Logs:

`Search-UnifiedAuditLog -recordtype SharePointSharingOperation -operations 'AnonymousLinkUsed' -startdate 2022-07-30 -enddate 2022-08-01`

### 12.List all Service Principals configured  with secrets in Azure AD

Query Azure AD Tenant:

`$Spns = Get-AzureADServicePrincipal -All $true
foreach ($Spn in $Spns) {
    if ($Spn.PasswordCredentials.Count -ne 0 -or $Spn.KeyCredentials.Count -ne 0) {
    Write-Host 'Application Display Name::'$Spn.DisplayName
    Write-Host 'Application Password Count::' $Spn.PasswordCredentials.Count
    Write-Host 'Application Key Count::' $Spn.KeyCredentials.Count
    Write-Host ''
    } }`