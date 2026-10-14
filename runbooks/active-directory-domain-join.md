
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

## Quick diagnostic path

1. **DC / CMD:** identify the current DC IP; do not assume historical addresses still apply.
2. **Client / CMD:** check IP, ping, and direct DNS SRV queries against the DC.
3. **Client / PowerShell:** compare default DNS resolution with explicit DC DNS resolution and inspect both IPv4 and IPv6 DNS servers.
4. **Client:** check domain discovery, TCP service connectivity, and the join log before changing adapter settings.
5. **Client / PowerShell as Administrator:** if justified, perform the temporary IPv6 isolation test; restore IPv6 and repeat discovery tests afterward.
6. **Client / CMD:** verify actual domain logon with `whoami`.

The original incident was consistent with competing DNS servers; the precise resolver-selection mechanism was not independently established. Do not interpret a working workaround as proof that IPv6 itself is faulty.

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

### Step 8 — Inspect the domain-joining log

**Machine:** Affected Windows client

**Terminal:** PowerShell, run as Administrator

```powershell
Get-Content "C:\Windows\debug\NetSetup.log" -Tail 60
```

Look for the most recent domain-joining attempt and any relevant error codes.

### Step 9 — Temporary workaround used in the original incident

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

After re-enabling the IPv6 binding on the **affected Windows client** in **PowerShell as Administrator**, verify the state and DNS configuration:

```powershell
Get-NetAdapterBinding -Name "Ethernet" -ComponentID ms_tcpip6
Get-DnsClientServerAddress -InterfaceAlias "Ethernet"
Resolve-DnsName -Name "_ldap._tcp.dc._msdcs.jobskillshare.org" -Type SRV
nltest /dsgetdc:jobskillshare.org /force
```

If resolver failures return, investigate the router's IPv6 DNS advertising or DHCP settings and provide Active Directory-aware DNS through the appropriate configuration. **The permanent fix and restoration test remain unverified in this original lab.**

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

## Additional DC diagnostics (if direct DNS queries fail)

**Machine: DC — PowerShell as Administrator**

```powershell
Get-DnsServerZone | Format-Table ZoneName,ZoneType,IsDsIntegrated
Get-DnsServerResourceRecord -ZoneName "_msdcs.jobskillshare.org" -Name "_ldap._tcp.dc" -RRType SRV
```

**Machine: DC — Command Prompt as Administrator**

```cmd
dcdiag /test:dns /v
sc query netlogon
```

These commands inspect the zones, locator record, registration checks, and Netlogon. Avoid recreating zones or reinstalling AD before diagnosing the evidence.

## Preventive Improvements

Maintain stable DC addressing, configure clients to use Active Directory-aware DNS servers, retain a working IPv6 configuration, and document actual test results.

## Lessons Learned

DNS resolution, network connectivity, and domain authentication are separate checks. Successful results from one test do not guarantee success in the others.

Documenting the original Helpdesk incident made troubleshooting a similar issue on the Staff workstation substantially easier.
