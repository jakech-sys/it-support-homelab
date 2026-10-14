# IT Support Home Lab

## Overview

This repository documents my hands-on journey in IT support, systems administration, networking, security, and infrastructure troubleshooting.

I am building and managing a virtual IT environment using Oracle VirtualBox to develop practical skills in installing, configuring, maintaining, securing, and troubleshooting enterprise IT systems.

The repository will grow as I complete new labs, encounter technical problems, investigate their causes, and document verified solutions.

## Lab Environment

| Component | Technology | Purpose |
|---|---|---|
| Hypervisor | Oracle VirtualBox | Create and manage virtual machines |
| Domain Controller | Windows Server 2019 | Host Active Directory and DNS |
| Helpdesk workstation | Windows 10 Enterprise | Simulate an IT support workstation |
| Staff workstation | Windows 10 Enterprise | Simulate a standard employee workstation |
| Directory service | Active Directory Domain Services | Centralized identity and account management |
| Domain | `jobskillshare.org` | Lab Active Directory domain |

## Completed Work

### Lab 01 — Virtualized Windows Active Directory Environment

- Installed and configured Oracle VirtualBox.
- Installed Windows Server 2019 and Windows 10 Enterprise.
- Created VM clones for lab reuse.
- Configured Active Directory Domain Services and DNS on Windows Server 2019.
- Created the `jobskillshare.org` domain.
- Created domain user accounts for Helpdesk and Staff.
- Joined the Helpdesk and Staff computers to the domain.
- Verified domain sign-ins using `whoami` and environment variables.
- Investigated and resolved Active Directory domain-join issues.
- Examined differences between standard user and administrative logon rights.

## Troubleshooting Experience

During the initial domain-joining process, the Helpdesk workstation could not locate the Domain Controller.

I investigated network connectivity, DNS server configuration, Active Directory SRV records, domain discovery, and service connectivity using Windows administrative tools.

The client was configured with competing IPv4 and IPv6 DNS servers. Temporarily disabling IPv6 on the client adapter was the workaround that enabled the lab to proceed. Correcting the underlying DNS configuration remains a future improvement.

I documented the troubleshooting process and reused the documentation when configuring the Staff workstation, reducing the time needed to resolve a similar issue.

This experience demonstrated the importance of systematic troubleshooting and reusable technical documentation.

## Skills Practiced

- Windows Server and Windows client administration
- Oracle VirtualBox virtualization
- Active Directory Domain Services
- DNS configuration and troubleshooting
- Domain joining and authentication
- Windows security groups and logon permissions
- PowerShell and Command Prompt
- Technical documentation and troubleshooting

## Future Development

This environment will support additional hands-on work in user administration, Group Policy, access control, networking, security, system maintenance, automation, and common IT support incidents.

## Documentation Approach

Each lab will document its objective, environment, configuration, validation, troubleshooting, resolution, and lessons learned. Commands will identify the machine and terminal in which they were executed.

All findings will be based on actual lab observations and verified results.
