---
layout: post
classes: wide
title:  "Credential scanning in cloud environments"
date:   2023-08-05 01:00:00 +0800
--- 
This post provide details on  credential scanning methodology over various instances in cloud environment.  

 
## Introduction

Organizations are rapidly transforming their infrastructure and hosting in various cloud providers. Sensitive systems and software codes are developed using cloud services and lean toward serverless and container-based architectures. Secure and scalable secret management solutions have to be enforced for seamless development activities and business operations. Lagging on these strategies can end up with unpredicted implications like threat actors privileged access over the environment or access to the crown jewels. Threat actors can extract secrets from the multiple surfaces in cloud environment. Proactively scan the cloud instances and reduce the exposure of hardcoded secrets.  
      
### Secrets in AWS Lambda 

AWS Lambda is a serverless computing services, can be used for running the code without provisioning infrastructure. This is event driven serverless computing platform. The practice of hardcoding sensitive credentials in Lambda functions will get expose to privilege escalation opportunity. This data can be accessed with read only privileged account. Regardless of the password length or complexity, an attacker who gains access to an account or code repository with read-only privileges would be able to use the exposed credentials to escalate privileges.  

LambdaGuard is an AWS Lambda auditing tool designed to create asset visibility. It provides an overview in terms of statistical analysis, AWS service dependencies and configuration checks from the security perspective. This tool will detect the hardened  credentials in the lambda functions through static analysis over the codes.    
For more details,     
https://github.com/Skyscanner/LambdaGuard

### Credential in AWS SSM documents

AWS Systems Manager document defines the actions that AWS Systems Manager performs on the managed instances. Systems Manager includes more than 100 pre-configured documents that we can use by specifying parameters at runtime. These documents can contain passwords, access keys and secret keys to manage multiple resources. Exposing these credentials will provide opportunity to threat actors for privilege escalation and lateral movements. You can reduce the exposure by using parameter stores and AWS secret manager.  

Use tools such as Prowler and perform scan against your AWS accounts to identify AWS SSM documents hardcoded with credentials.  

For more details,  
https://github.com/prowler-cloud/prowler/

### Credential exposures in Git repositories

GitHub is a Git repository hosting service. Git is a command line tool, whereas GitHub provides a Web-based graphical interface. It also provides several collaboration features, such as a wikis and basic code management tools for every project. API Keys and security credentials often embedded in the software codes. Various tools can be leveraged for detecting secrets that have been inadvertently shared in Git repositories. Some of the common tools that can be used to scan git repositories for existence of these secrets are GittyLeaks, Secret Scanning, Git Secrets, Git Hound etc. GittyLeaks is a python-based free utility for finding secrets like a username, password, email in a string, config, or JSON file formats.  

`gittyleaks --find-anyting -link <Github link>`

### AWS EC2 Instance metadata
AWS Instance metadata is data about EC2 instance that needed to configure or manage the running instance. Instance metadata can be used to access user data that is specified when launching EC2 instance. For example, we can specify parameters for configuring EC2 instance, or include a simple script. When we launch an instance in Amazon EC2, we have the option of passing user data to the instance that can be used to perform common automated configuration tasks and even run scripts after the instance starts. Shell scripts can be passed to the instance.  

The practice of hardcoding sensitive credentials over EC2 instances will get expose with read-only access. Regardless of the password length or complexity, threat actor who gains access to an account with read-only privileges would be able to use the exposed credentials in the instance user data to escalate privileges and move laterally.  

Perform periodic or automated review over EC2 instance user-data scripts for any sensitive credentials that have been hardcoded. Multiple open-source and professional software are available to automate scanning for sensitive credentials within AWS environment. Tools like ScoutSuite can be used.  
 
For more details,  
https://github.com/nccgroup/ScoutSuite   

### Infrastructure as a code Templates

Infrastructure as a Code (IaC) templates are used in cloud environment to deploy infrastructure at scale. Configuration templates are used by several IaC applications to describe target infrastructure. Secrets like SSH keys and password are often stored in the configuration files. The practice of using hard coded sensitive credentials may result in exposure of the secrets.  
 
Review all the IaC templates that is used and search for any credentials. There are several tools that are available to perform detection of credentials in cloud IaC templates. One of the tool that could be used is cfn-nag. Cfn-nag can look for patterns in cloud formation templates that may indicate insecure infrastructure. Password literals is of one of the information that Cfn-nag can search for in cloud formation templates and generate alerts.  

`cfn_nag_scan --input-path <path to cloudformation json file>`  

For more details,  
https://github.com/stelligent/cfn_nag

### Credential Scanning over storage services such as S3 bucket

AWS S3 buckets are used as simple storage mechanism for files in the cloud environment. S3 buckets can be accessed from public based on the configurations. Developers sometime may store credentials and other secrets in source code and other material stored in these S3 buckets.   

Scan some of the well-known files and software codes that are expected to have credentials such as .py, .json, .yaml, .php.
Perform recursive search over all the files to identify access keys.  
`grep -RP '(?<![A-Z0-9])[A-Z0-9]{20}(?![A-Z0-9])' *`  

Perform recursive search over all the files to identify secret keys.  
`grep -RP '(?<![A-Za-z0-9/+=])[A-Za-z0-9/+=]{40}(?![A-Za-z0-9/+=])'`  



