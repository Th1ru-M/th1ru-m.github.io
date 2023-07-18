---
layout: post
classes: wide
title:  "Exploitation of Misconfigured Certificate Templates"
date:   2023-06-19 01:00:00 +0800
--- 
This post provides areas of security exposure in active directory Certificate Services and the related techniques that the threat actors can use to target ADCS and how to detect these attack techniques. 


## Exploitation of Misconfigured Certificate Template with Subject Alternative Name (SAN)

Users can create Certificate Signing Request for the allowed and published certificate templates and receive certificates from AD CS servers. Organizations often commit misconfigurations in the certification templates by permitting subject alternate name field. The Subject Alternative Name (SAN) field permit users to create Certificate Signing Request for other user accounts. The certificate templates AD object comprises  SAN field by including CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT_ALT_NAME flag in the templates object, which allow users to specify SAN in the CSR. 

Furthermore if the templates allow domain authentication, with domain user rights to enroll, no approval process and SAN to be included, then any domain users can mention the privileged user account or the machine account under SAN and receive a certificate of the privileged account from AD CS. Adversaries can use this technique to elevate the privileges to Domain or enterprise administrator. Then after, Adversaries can perform user impersonation and execute binaries in the context of privileged user.

![SAN_Abuse](/image/san.JPG)

### Detections

1.Event ID 4886 and 4887 in AD CS server. Event ID 4886 is generated when client request for certificate and Event ID 4887 is generated for certificate issuance. Both these events will not record Subject Alternative Name fields in the request and response.

2.Event ID 4768 in AD servers will be generated for  Kerberos Authentication Ticket request and this includes details of the Certificate Information fields with the authenticating certificates Issuer, Serial Number, and Thumbprint

3.Leverage public tools like PSPKIAudit to validate and identify certificate issued for and requestor details , which will list the presence of SAN in certificate request 

4.Leverage Certutil executable to list all the certificate issued and filter SAN fields in the certificates

### Mitigations

1.Regularly review the published templates and its settings that includes, SAN Specification and domain authentications in the templates, enrollment permissions assigned in the template.

2.Enforce CA Certificate Manager approval for the templates that include SAN as an Issuance requirement.

3.Leverage public tools like PSPKIAudit to validate and identify misconfigurations in certificate templates. 

## Exploitation of Misconfigured Certificate Template with over-permissive access permissions

Users can create Certificate Signing Request for the allowed and published certificate templates and receive certificates from AD CS servers. Certificate Templates are secured using Security descriptors, which define security principals that hold permissions over it through access control entries (ACE). 

If a published certificate template is misconfigured with overly permissive ACE to allow a non-privileged users to edit any properties in the certificate template, then this provides opportunity for privilege elevation. Adversaries can edit sensitive properties like Subject Alternative Name(SAN) by enabling CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT_ALT_NAME flag in the certificate templates object, which allow adversaries to specify other privileged user account in SAN field in the certificate signing request. Then after, Adversaries can perform user impersonation and execute binaries in the context of privileged user.

### Detections

1.Event ID 4674 in AD server will be triggered for any changes in security descriptors for the certificate templates. The object name in the event will list the certificate template name.

2.Leverage Certutil executable to list all the permissions defined in the certificate templates.
`certutil.exe -v -dsTemplate`

### Mitigations

1.Regularly review the published templates and its settings that includes, SAN Specification and domain authentications in the templates, permissions assigned in the template.

2.Enforce CA Certificate Manager approval for the templates that include SAN as an Issuance requirement.

3.Leverage public tools like PSPKIAudit to validate and identify misconfigurations in certificate templates.


Reference for the details:
- <https://attack.mitre.org/techniques/T1649/>
- <https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf>


