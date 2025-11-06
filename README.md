# IT Support Homelab (Active Directory, DNS, Printing)

This repo documents a small AD/DNS/Win11 lab used to practice Tier-1/Tier-2 support tasks for an onsite IT Support Specialist interview.

**Stack:** Server 2022 Core (AD DS + DNS) • Win11 client • VirtualBox (Host-Only 192.168.52.0/24 + NAT)  
**Domain:** `lab.local` • **DC:** `SRV-DC` @ `192.168.52.10` • **DNS client:** Win11 points to DC

## Outcomes (evidence-first)
- Fixed multi-NIC DC DNS pollution (NAT **10.0.2.4** records removed).
- Win11 domain-joined; `srv-dc.lab.local` resolves to **192.168.52.10** only.
- Group-based share **EngShare** mapped to **Z:** with write access.

| Proof | Screenshot |
|---|---|
| `nslookup srv-dc.lab.local 192.168.52.10` → **192.168.52.10** | ![nslookup proof](assets/images/nslookup-srv-dc-192-168-52-10.png) |
| DNS zone shows **no** 10.0.2.4 A records | ![DNS zone clean](assets/images/dns-zone-clean-no-10-0-2-4.png) |
| `Z:` mapped to `\\192.168.52.10\EngShare` and test file saved | ![Mapped drive Z](assets/images/mapped-drive-z.png) |

## Key commands used
```powershell
# On the DC: stop NAT NIC from publishing to DNS
Set-DnsClient -InterfaceAlias "NAT" -RegisterThisConnectionsAddress $false

# Remove stale A records pointing to 10.0.2.4
Get-DnsServerResourceRecord -ZoneName 'lab.local' -RRType A |
  ? { $_.RecordData.IPv4Address -eq ([ipaddress]'10.0.2.4') } |
  Remove-DnsServerResourceRecord -ZoneName 'lab.local' -Force

# Re-register proper records and clear caches
ipconfig /registerdns
nltest /dsregdns
Clear-DnsServerCache -Force
