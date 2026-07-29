# active-directory-lab

Companies use active directory to organize around departments and delgating permissions to certain departments like IT support access
instead of broad admins rights 

# Active Directory Identity & Access Lab

## Overview

Growing organizations need a way to manage user access without giving every support technician full administrative rights. This project simulates that problem: a small company environment built in Azure, structured around departments, with scoped delegated permissions so IT support staff can handle day to day account tasks (password resets, account unlocks) without holding domain-wide admin access.

The lab covers the full identity lifecycle a helpdesk or IT support technician would actually handle: infrastructure setup, domain design, delegated access control, policy enforcement, and full onboarding/offboarding workflows.

## Skills & Tools

| Category | Tools/Skills |
|---|---|
| Cloud Infrastructure | Microsoft Azure (Resource Groups, Virtual Networks, Virtual Machines) |
| Identity & Directory Services | Active Directory Domain Services (AD DS), Domain Controller promotion |
| Access Management | Organizational Unit (OU) design, delegated permissions, least-privilege access control |
| Policy Enforcement | Group Policy Objects (GPO), Group Policy Management |
| Endpoint Management | Domain join, client configuration, DNS |
| Operations | User onboarding/offboarding, password reset and account unlock workflows |

## Environment

- **Domain:** adlab.local
- **Domain Controller:** Windows Server 2022 (vm-dc-adlab)
- **Client Machine:** Windows 11 Pro, domain-joined (vm-client01)
- **Platform:** Microsoft Azure (Azure for Students subscription)
- **Network:** Single VNet with a dedicated subnet for domain resources

## Organizational Design

The domain is structured around two departments, each with dedicated sub-OUs for Users, Computers, and Groups:

\`\`\`
- adlab.local
  - IT
    - Users
    - Computers
    - Groups (Helpdesk-L1)
  - Finance
    - Users
    - Computers
    - Groups
\`\`\`

This structure mirrors how a real business would separate departments for permission scoping and policy targeting.

## Delegated Access Control

Rather than granting broad administrative rights, a `Helpdesk-L1` group was created and delegated **only** the permissions a first-tier support technician needs:

- Reset user passwords
- Force password change at next logon
- Read user account information

The screenshot below shows the resulting permission set: Helpdesk-L1 has specific, scoped rights on the Finance Users OU, with no `Full Control` entry, in contrast to Domain Admins and Enterprise Admins, which do.

![Delegated permissions on Finance Users OU](screenshots/advanced-security-settings-delegation.png)

This was tested end to end: a `Helpdesk-L1` member was able to reset passwords for users in Finance, matching the intended scope.

## Group Policy

An `IT-Desktop-Policy` GPO was created and linked to the IT OU, then verified applying successfully on a domain-joined client after the client's computer object was moved into the correct OU.

![GPO applied on client machine](screenshots/gpo-applied-wallpaper.png)

## Onboarding Simulation

A full onboarding flow was simulated to mirror a real new-hire process:

1. Created a new user account in the correct departmental OU
2. Added the user to the relevant department security group
3. Enforced a mandatory password change at first logon
4. Granted the account remote access rights
5. Verified successful login as the new user

![Successful onboarding login](screenshots/onboarding-login-success.png)

## Offboarding Simulation

A matching offboarding flow was simulated to reflect a real departure process:

1. Disabled the user account
2. Removed the account from its department security group
3. Moved the account to a dedicated Disabled Users OU

![Offboarded user in Disabled Users OU](screenshots/offboarding-disabled-users.png)

## Troubleshooting Notes
- DNS misconfiguration blocking domain join
- Domain controllers restrict Remote Desktop access differently than member servers, requiring a change to the Default Domain Controllers Policy
- UAC elevation prompts are a local machine control, separate from AD-delegated permissions, an important distinction between local rights and directory-level access

## What This Demonstrates

- Designing an OU structure around real organizational boundaries
- Applying least-privilege principles through delegated permissions
- Enforcing policy at scale with Group Policy
- Managing the full account lifecycle: onboarding, access changes, and offboarding
- Diagnosing and resolving real infrastructure and permissions issues
