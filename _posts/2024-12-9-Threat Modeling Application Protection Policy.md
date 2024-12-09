---
layout: post
classes: wide
title:  "Threat Modeling on Unmanaged Mobile Devices: Protecting Corporate Data"
date:   2024-12-9 05:00:00 +0800
--- 
This post provides details on Threat Modeling on Unmanaged Mobile Devices to Protect Corporate data from leakage.

 
### Introduction

Organization employees uses their personal mobile devices to access corporate data via a mobile applications, such as Outlook or Teams. Unmanaged mobile devices is a major security concern as organizations adopt "bring your own device"(BYOD) policies to allow access to corporate data. Unmanaged devices are not under direct control of the organization, making it difficult to enforce strict data protection measures. Corporate data may be inadvertently or maliciously exposed, leading to breaches.

Microsoft Application Protection Policy (APP) is a powerful tool for securing corporate data on mobile devices without requiring full device enrollment or management with organization MDM solutions or Intune. This allows organizations to implement a BYOD strategy while maintaining control over corporate information.

APP creates barrier around specific Microsoft or few other applications,by segregating corporate data from personal data and applications.  Policies define how these mobile applications can access and share corporate data.

App protection policies are supported on iOS and Android. At the time of Writing, App protection policies are supported on Windows in preview for the Microsoft Edge browser only.

### Threat Scenarios

<u>Data Leakage via Personal Device:</u>
An employee uses their personal mobile device to access corporate data via a mobile application, such as Outlook or Teams, but the device is not sufficiently secured or managed by the organization. The employee then accidentally or maliciously shares corporate data with personal apps (e.g., WhatsApp, Gmail, or Dropbox), causing a data leak.

<u>Malicious Keyboard in Mobile Devices:</u>
An employee installs and uses a third-party keyboard on their mobile device (e.g., SwiftKey, Gboard, or any other third-party alternative) for convenience or personal preference. These third-party keyboards can have access to all typed input, which may include sensitive corporate data such as passwords, emails, personal identifiable information (PII), or other confidential corporate data. If the keyboard is malicious or compromised, it can potentially send this data to an external server controlled by adversaries.

<u>Infected Employees BYOD Device:</u>
An employee installs a malicious third-party application on their mobile device. The application may granted access to corporate data in the mobile device, which it then attempts to exfiltrate corporate data.

<u>Insider Threat:</u>
A user in the organization shares a sensitive file via email, Teams, or cloud storage to an external user who should not have access to it. This could be due to intentional malicious activity. A user with legitimate access to a sensitive corporate application can intentionally exfiltrates or misuses corporate data

<u>Lost Employee Mobile Device:</u>
An employee loses their mobile device (e.g., smartphone, tablet, or laptop) containing access to corporate apps and sensitive data. The device could be lost in a public place, stolen, or misplaced, giving an unauthorized individual the potential to access corporate data.

<u>JailBroken Device:</u>
Rooting (Android) or jailbreaking (iOS) is one of the most common ways adversaries can bypass application protection policies on mobile devices. When a device is rooted or jailbroken, the attacker gains full control over the device operating system. This allows them to disable or manipulate security controls enforced by Microsoft APP.

<u>Fake Version of a Corporate App:</u>
A user attempts to download and install a non-approved version of a corporate application (e.g., a tampered version of Outlook or Teams from an unofficial source), which could include backdoors or other malicious functionality.

### Defense Controls using Microsoft App protection Policy

 Application protection Policies can be leveraged for devices that are typically employee owned devices that are not managed or enrolled in Organization Intune or other MDM solutions.

- Enforces PIN or biometrics to access the corporate applications, even if the device itself is unlocked.

- Prevent backup of Organization protected applications and data in mobile devices

- Ensures that data in the applications is encrypted both on the device and in transit to prevent unauthorized access

- Prevents copying data from a corporate app to a personal app and vice versa

- Disable the ability to take screenshots or screen recordings of app data

- Allow approved Keyboards to get installed in the mobile devices

- Prevent policy managed apps from saving data to the device native apps (like Contacts, Calendar and widgets), or to prevent the use of add-ins within the policy managed apps

- Allow orgnaization data as web content to open only in Microsoft Edge, as Edge can also be added as protected apps

- APP cant block jailbroken or rooted devices to access the organization data by itself. However if the device is enrolled with Intune, organization can block through device compliance and conditional access policies

- Block printing Organization data from the BYOD devices

- Enforce Conditional access policies with Android/IOS mobile devices to grant access to Organization data only with a control `Require app protection policy`




