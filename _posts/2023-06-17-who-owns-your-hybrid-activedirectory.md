---
layout: post
classes: wide
title:  "Who owns your hybrid active directory ! Hunting for adversary techniques"
date:   2023-06-17 18:00:00 +0800
--- 
This post provides three areas of security exposure in hybrid active directory and the related techniques that the threat actors can use to target hybrid Active Directory and how to detect these attack techniques. 

 
## Introductions
tl;dr

Active directory (AD) is the most commonly used identity provider (IdP) in corporate IT environments. It is the underlying fabric of IT Infrastructure and provides Identity services for many organizations. Active directory has traditionally supported on-premises deployment of Microsoft Windows networks. With cloud adoption on the rise, organizations are increasingly leveraging services hosted in cloud. Often the most effective way of providing access to such resources is to leverage an IdP in the cloud. Microsoft Azure Active Directory (Azure AD) (Microsoft Azure, 2021) is the Microsoft solution for a cloud-based Identity Provider. 
Organisations willing to leverage the value that cloud provides, while still hosting the identity provider on-premises, are increasingly using Hybrid identity by combining Azure and On-premises Active Directory. Hybrid Active Directory solution spans across on-premises and cloud-based environments. It can be used to provide a common digital identity that provides authentication and authorization to resources irrespective of where they reside. 
In this paper I discuss the security of Azure Active Directory and its hybrid implementations. The premise is that once a threat actor has gained access to the Azure AD implementation, they can configure a number of backdoors and long-term persistence techniques in the environment. This could provide threat actor long term, at will, access to organizational resources. 
I discuss three areas of security exposure in hybrid active directory and the related techniques that the threat actors can use to target them. I have also provide detection and threat hunting advise to help security teams detect these techniques in real time and in retrospect. 
I recommend organizations take a proactive approach of identifying adversary behaviour, techniques, and tools in their cloud environment as they do for their on-premises environments.  

Refer the below linked whitepaper for details.


## Abusing Azure Applications

An Azure application is a software that is used to provide functionality to users.  Azure Active Directory is Microsoft’s cloud-based Identity platform. To work with the Azure AD, Applications (Apps) need to be registered in Azure AD.
New applications can be created and registered in Azure AD in the form of Application objects. Application registration is created as an application object in the Azure AD and is referred to as Application. Application object is the global representation of an application in the Azure environment. 
Applications can be single or multi-tenant. Single-tenant applications are registered for a specific Azure AD directory and only available in the home tenant where they have been registered in. Multi-tenant applications are available for usage for multiple tenants. In case of a multi-tenant application, the initial application registration is performed in the home tenant and the application object also resides there. When a user from a different tenant (consumer tenant) signs-in the application for the first time, a consent is requested by the application. If the consent is granted, a service principal, also known as enterprise application in Azure portal, is created in the consumer tenant. To interact with and use Azure applications, a service principal should be present in a tenant. Service principal is the local tenant level reference to the global Azure application object. The details of the permissions for which consent has been provided are also registered with the service principal.
A common use case of such an implementation is a multi-tenant Software as a Service applications (SaaS).  The SaaS providers can register their application in their home tenant and then consumers can create service principals in their tenants to use the functionality provided by the application. A single instance of software application is shared by multiple clients or tenants.  

### Types of Permissions
There are two types of permissions that are supported by Azure applications and service principals.
Application Permissions: These permissions are used by an application without the need of a user signed in into the application. Application permissions require the consent to be provided by an administrator. Once the administrator provides the consent to the application, the application permissions are assigned to the service principal associated with the Application. Service principals assigned with application permissions have permanent access over the resources.

Delegated Permissions: These permissions are used by an application when there is a user signed into the application. Service principals assigned with delegated permissions impersonate the permissions of the signed-in user in the application. 
Effective Permissions: Effective permissions of an application are the permissions that the application has when it tries to access a resource. Effective permissions may be constrained based on the permission of the signed in user, especially in the delegated permission model. 

Consent: Application permissions always require a consent from an Administrator (admin consent). Some delegated permissions can be consented by the user while privileged permissions may require an administrator consent. Once appropriate consent has been provided, the consent approval is registered with the service principal of the application in the Azure AD tenant. 

Certificates & secrets: Certificates or secrets can be configured for application objects in the home tenant or on service principal in home tenant or the consumer tenants. These certificates or secrets can then be used to access the tenant with the effective permissions of the service principal. 
  

## Abusing Identity Federation Configuration

Identity federation is a system of trust between two parties that is used to outsource authentication and authorization to an Identity Provider (IdP). 
Azure Active Directory (Azure AD) can act as an identity provider or can establish a federated trust with an external Identity provider. Federation enables secure sharing of digital identity and entitlement claims across security boundaries.  Using Federation in Azure AD, users can authenticate with credentials registered with another Identity provider and access resources (service providers) that are registered in the Azure tenant. 
Federation Authentication flow
Active Directory Federation Services (AD FS) is a commonly used on-premises Identity provider that can be used as a federated identity solution with Azure AD. When a user logs in to an Azure AD registered application, the user is redirected to the AD FS server, the response from the AD FS server is used to authenticate and authorize the user security principal. 

### Threat Actor Workflow:
It is worth noting that Azure AD trusts the AD FS (an external Identity Provider) and the authentication is based on the signed SAML tokens provided by the AD FS. The user validation is based on the SAML token that is signed by the token signing certificate (TSC) present in the AD FS server and is provided to the service provider. 
A very well-known attack called Golden SAML, (Reiner, 2017) leverages this trust, and allows a threat actor to use stolen certificates to create valid SAML tokens and leverage them to sign into the service provider. This extremely powerful technique can be used to laterally move from on-premises environment to the cloud environment by a threat actor. Threat actor can add a federated domain in Azure AD and leverage that to create SAML tokens that can be used to impersonate any hybrid identity user in the environment. 



## Backdooring Pass Through Authentication 
Pass-through Authentication allows organizations to enforce user authentication to happen on their on-premises Active Directory servers. This helps organizations to enforce their on-premises Active Directory security and password policies and maintain control on the identities. 
In case of PTA the security principal password is not synchronized to the Azure AD. When a user signs-in to an application integrated with Azure AD, the authentication is performed by on-premises AD servers using the PTA agent. Once registered PTA agent maintains a persistent outbound connection to the Azure AD. 

### Threat Actor Workflow:
If an organization has Pass-through Authentication configured for their Azure AD then the threat actor can register their own Pass-through Authentication server as an additional Pass-through agent in Azure AD of the organizations tenant.  This registration requires the threat actor to have access to a compromised account with Global Administrator privileges in the target Azure AD.
Once the threat actor has successfully registered their PTA server in Victim Azure AD tenant, the user credential validation requests for the tenant will be load balanced between the servers configured in the tenant including the TA managed PTA server. 



Reference for the detailed whitepaper
- Whitepaper published in Virus Bulletin : <https://vblocalhost.com/uploads/VB2021-Thirumalai-Khanna.pdf>

