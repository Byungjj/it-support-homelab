# IT Support Homelab (Active Directory, DNS, Printing)

This repo documents a small AD/DNS/Win11 lab used to practice Tier-1/Tier-2 support tasks for an onsite IT Support Specialist interview.

**Stack:** Server 2022 Core (AD DS + DNS) • Win11 client • VirtualBox (Host-Only 192.168.52.0/24 + NAT)  
**Domain:** `lab.local` • **DC:** `SRV-DC` @ `192.168.52.10` • **DNS client:** Win11 points to DC

## Outcomes (evidence-first)
- Fixed multi-NIC DC DNS pollution (NAT 10.0.2.4 records removed).  
- Win11 domain-joined, can resolve `srv-dc.lab.local` → `192.168.52.10` only.  
- Group-based share **EngShare** mapped to `Z:` with write access.

| Proof | Screenshot |
|------|------------|
| `nslookup srv-dc.lab.local 192.168.52.10` → `192.168.52.10` | ![nslookup](assets/nslookup_srv-dc.png) |
| DNS zone shows no 10.0.2.4 A records | ![dns zone](assets/dns_zone_clean.png) |
| `Z:` mapped to `\\192.168.52.10\EngShare` and test file saved | ![drive](assets/mapped_drive_Z.png) |

## Key commands used
```powershell
# Stop NAT NIC from publishing to DNS
Set-DnsClient -InterfaceAlias "NAT" -RegisterThisConnectionsAddress $false

# Remove stale A records pointing to 10.0.2.4
Get-DnsServerResourceRecord -ZoneName 'lab.local' -RRType A |
 ? { $_.RecordData.IPv4Address -eq ([ipaddress]'10.0.2.4') } |
 Remove-DnsServerResourceRecord -ZoneName 'lab.local' -Force

ipconfig /registerdns
nltest /dsregdns
