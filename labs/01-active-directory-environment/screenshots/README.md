# Lab 01 — Screenshot Evidence

These images document the original VirtualBox Active Directory lab. They are **historical evidence**, not example outputs. IP addresses are from the lab on 14 October 2026 and may change.

| Image | Evidence |
|---|---|
| [01-domain-controller.png](01-domain-controller.png) | DC: AD DS and DNS roles |
| [02-domain-joined-computers.png](02-domain-joined-computers.png) | DC: HELPDESK and STAFF objects |
| [03-helpdesk-domain-login.png](03-helpdesk-domain-login.png) | Helpdesk: jss\helpdesk verified |
| [04-staff-domain-login.png](04-staff-domain-login.png) | Staff: jss\staff verified |
| [05-dns-client-failure.png](05-dns-client-failure.png) | Helpdesk: default DNS lookup fails |
| [06-directory-service-ports.png](06-directory-service-ports.png) | Helpdesk: TCP 389, 88, 445 tests |
| [07-dc-discovery-success.png](07-dc-discovery-success.png) | Helpdesk: SRV and nltest succeed |
| [08-competing-dns-settings.png](08-competing-dns-settings.png) | Helpdesk: IPv4 + IPv6 DNS evidence |
| [09-staff-dc-logon-denied.png](09-staff-dc-logon-denied.png) | DC: staff interactive logon denied |
| [10-dc-logon-rights-policy.png](10-dc-logon-rights-policy.png) | DC: effective logon rights via secedit |
| [11-admin-domain-login.png](11-admin-domain-login.png) | DC: renamed jss\admin verified |

**Notes:** The IPv6 binding change was a temporary diagnostic workaround, not a recommended permanent fix. Never upload passwords, keys, VM disk images, or ISO files.

## Selected image previews

### Infrastructure and computer objects

![Domain Controller with AD DS and DNS](01-domain-controller.png)

![Domain-joined HELPDESK and STAFF computers](02-domain-joined-computers.png)

### Verified client logons

![Helpdesk account identity](03-helpdesk-domain-login.png)

![Staff account identity](04-staff-domain-login.png)

### DNS failure and recovery

![Initial client DNS failure](05-dns-client-failure.png)

![Successful SRV lookup and DC discovery](07-dc-discovery-success.png)

[Back to Lab 01](../README.md) | [Domain-join runbook](../../../runbooks/active-directory-domain-join.md)
