
# Runbook — Active Directory Domain Join Troubleshooting

## Purpose

Troubleshoot Windows clients that cannot join the `jobskillshare.org` Active Directory domain.

This runbook is based on an actual incident encountered while configuring the Helpdesk workstation in Oracle VirtualBox.

## Environment

| Machine | Operating system | Role |
|---|---|---|
| DC | Windows Server 2019 | Domain Controller and DNS |
| Helpdesk | Windows 10 Enterprise | Client workstation |
| Staff | Windows 10 Enterprise | Client workstation |

**Lab domain:** `jobskillshare.org`

**Historical DC IP:** `192.168.1.127` (verify before reuse).

## Symptoms

The client displayed an error indicating that an Active Directory Domain Controller could not be contacted.

Domain-joining diagnostics also returned error `1355` (`ERROR_NO_SUCH_DOMAIN`).

## Troubleshooting Procedure

### Step 1 — Check the Domain Controller's network configuration

**Machine:** DC

**Terminal:** Command Prompt, run as Administrator

```cmd
ipconfig /all
```

Record the current IPv4 address, subnet mask, gateway, and DNS servers. Do not assume that the previous DC address is still valid.

### Step 2 — Inspect the client network configuration

**Machine:** Helpdesk or the affected Windows client

**Terminal:** Command Prompt

```cmd
ipconfig /all
```

Verify that the client can reach the DC and uses DNS capable of resolving the Active Directory domain.

### Step 3 — Test basic network connectivity

**Machine:** Affected Windows client

**Terminal:** Command Prompt

```cmd
ping 192.168.1.127
```

Replace the address with the DC's current IP.

Successful replies confirm basic IP connectivity, not full Active Directory functionality.

### Step 4 — Test Active Directory DNS records

**Machine:** Affected Windows client

**Terminal:** Command Prompt

```cmd
nslookup DC.jobskillshare.org 192.168.1.127
nslookup -type=SRV _ldap._tcp.dc._msdcs.jobskillshare.org 192.168.1.127
```

The SRV lookup should identify the Domain Controller as an LDAP service provider.

### Step 5 — Test Domain Controller discovery

**Machine:** Affected Windows client

**Terminal:** Command Prompt

```cmd
nltest /dsgetdc:jobskillshare.org /force
```

A successful result should identify `DC.jobskillshare.org`.

### Step 6 — Test required service connectivity

**Machine:** Affected Windows client

**Terminal:** PowerShell

```powershell
Test-NetConnection 192.168.1.127 -Port 389
Test-NetConnection 192.168.1.127 -Port 88
Test-NetConnection 192.168.1.127 -Port 445
```

These test TCP connectivity to LDAP, Kerberos, and SMB services.

They do not verify every protocol required for domain joining.

### Step 7 — Inspect DNS server selection

**Machine:** Affected Windows client

**Terminal:** PowerShell, run as Administrator

```powershell
Get-DnsClientServerAddress -InterfaceAlias "Ethernet"
```

In the original incident, the client was configured with the DC's IPv4 DNS address and a competing router-provided IPv6 DNS server.

Compare a direct lookup against the DC with a lookup using the client's default DNS configuration.

```powershell
Resolve-DnsName -Name "_ldap._tcp.dc._msdcs.jobskillshare.org" -Type SRV
```

### Step 8 — Temporary workaround used in the original incident

**Machine:** Affected Windows client only

**Terminal:** PowerShell, run as Administrator

The original lab succeeded after temporarily disabling IPv6 on the client's Ethernet adapter.

```powershell
Disable-NetAdapterBinding -Name "Ethernet" -ComponentID ms_tcpip6
```

**Warning:** This is a temporary diagnostic workaround, not a recommended permanent Active Directory configuration. It may interrupt networking and must not be applied indiscriminately.

After testing, restore the adapter binding:

```powershell
Enable-NetAdapterBinding -Name "Ethernet" -ComponentID ms_tcpip6
```

Correct the underlying DNS configuration so the client can consistently resolve Active Directory records without disabling IPv6.

### Step 9 — Inspect the domain-joining log

**Machine:** Affected Windows client

**Terminal:** PowerShell, run as Administrator

```powershell
Get-Content "C:\Windows\debug\NetSetup.log" -Tail 60
```

Look for the most recent domain-joining attempt and any relevant error codes.

### Step 10 — Verify the domain join

**Machine:** Affected Windows client

**Terminal:** Command Prompt

After successfully joining the domain and restarting, sign in with the intended domain account.

```cmd
whoami
echo %USERDOMAIN%
echo %USERNAME%
```

Verify that the output identifies the intended domain and user rather than the local computer account.

## Preventive Improvements

Maintain stable DC addressing, configure clients to use Active Directory-aware DNS servers, retain a working IPv6 configuration, and document actual test results.

## Lessons Learned

DNS resolution, network connectivity, and domain authentication are separate checks. Successful results from one test do not guarantee success in the others.

Documenting the original Helpdesk incident made troubleshooting a similar issue on the Staff workstation substantially easier.
