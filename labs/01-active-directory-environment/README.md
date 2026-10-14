# Lab 01 — Building a Windows Active Directory Environment

## Objective

Build a virtualized Windows IT environment to practice centralized user management, client authentication, domain administration, and troubleshooting.

## Technologies

- Oracle VirtualBox
- Windows Server 2019
- Windows 10 Enterprise
- Active Directory Domain Services (AD DS)
- Windows DNS
- PowerShell and Command Prompt

## Lab Architecture

| Machine | Operating System | Role |
|---|---|---|
| DC | Windows Server 2019 | Domain Controller and DNS Server |
| Helpdesk | Windows 10 Enterprise | IT Support workstation |
| Staff | Windows 10 Enterprise | Standard employee workstation |

**Active Directory domain:** `jobskillshare.org`

**NetBIOS domain:** `JSS`

## Implementation

### 1. Virtual Machine Setup

Installed Oracle VirtualBox and created virtual machines using Windows Server 2019 and Windows 10 Enterprise installation images.

Created VM clones to support future lab exercises and reduce repeated installation work.

### 2. Active Directory Configuration

Configured the Windows Server 2019 VM as a Domain Controller and installed Active Directory Domain Services and DNS.

Created the `jobskillshare.org` domain for centralized identity and account management.

### 3. User and Client Configuration

Created the Helpdesk and Staff domain user accounts and configured two Windows 10 client machines.

Joined both clients to the Active Directory domain.

### 4. Authentication Verification

Verified domain identities using the following commands on the respective Windows 10 client machines:

```cmd
whoami
echo %USERDOMAIN%
echo %USERNAME%
```

**Verified Helpdesk identity:** `jss\helpdesk`

**Verified Staff identity:** `jss\staff`

Also verified the Domain Controller administrator identity as `jss\admin`.

## Challenges Encountered

The initial Helpdesk domain-joining process failed because Windows could not consistently discover the Domain Controller through DNS.

Troubleshooting involved examining DNS settings, SRV records, network connectivity, Windows services, and domain-joining logs.

A competing IPv6 DNS configuration was identified. Temporarily disabling IPv6 on the client Ethernet adapter allowed the lab to proceed. A permanent DNS configuration improvement remains outstanding.

Documenting this investigation made troubleshooting a similar issue on the Staff client easier.

## Security Observations

Observed that Domain Admins and standard Domain Users have different permissions for interactive sign-in to the Domain Controller.

The Helpdesk account was assigned Domain Admin membership during the initial lab. Replacing this broad access with appropriately delegated permissions is a planned security improvement.

## Lessons Learned

- DNS plays a critical role in Active Directory domain discovery.
- Successful network connectivity does not guarantee successful domain joining.
- Domain accounts and local machine accounts are different identities.
- Security group membership affects authorization.
- Troubleshooting documentation makes repeated technical issues easier to resolve.

## Future Improvements

- Establish stable Domain Controller IP addressing.
- Correct IPv4 and IPv6 DNS configurations.
- Review VM cloning and Windows machine identity practices.
- Implement least-privilege Helpdesk administration.
- Practice Group Policy and organizational unit management.
- Extend the lab with additional users, workstations, and support scenarios.
