# Active Directory Home Lab: Brute Force Detection with Splunk

A home lab simulating a real enterprise environment with Active Directory, brute force attacks from Kali Linux, and detection via Splunk SIEM. Built entirely in VMware Workstation.

---

## Architecture

| Machine | IP | Role |
|---|---|---|
| Ubuntu Server (Splunk) | 192.168.111.136 | Central SIEM, collects and indexes logs |
| Windows Server 2022 | 192.168.111.131 | Domain Controller (mydfir.local) |
| Windows 10 | 192.168.111.130 | Domain-joined target endpoint |
| Kali Linux | 192.168.111.129 | Attacker machine |

All VMs run on a NAT network (192.168.111.0/24) in VMware Workstation.

---

## Tools

- Splunk Enterprise 10.4.0
- Splunk Universal Forwarder
- Sysmon (SwiftOnSecurity config)
- Active Directory Domain Services
- Crowbar (RDP brute force)
- Atomic Red Team (Invoke-AtomicRedTeam)
- VMware Workstation

---

## What This Lab Covers

**1. Active Directory Setup**
- Windows Server 2022 promoted to Domain Controller
- Domain: mydfir.local
- User accounts created (Jsmith, Tsmith)
- Windows 10 joined to the domain

**2. Log Forwarding Pipeline**
- Sysmon installed on Windows 10 and Windows Server 2022
- Splunk Universal Forwarder configured on both machines
- Logs forwarded to Splunk on port 9997
- All data indexed under `index=endpoint`

**3. Brute Force Attack Simulation**
- Crowbar run from Kali Linux against RDP on Windows 10
- Targets domain user accounts
- Generates Event ID 4625 (failed logon) events in Splunk

**4. Detection in Splunk**
- Search: `index=endpoint EventCode=4625`
- Key fields: Account_Name, Source_Network_Address, Logon_Type
- Cross-reference with EventCode 4624 to check for successful logons

**5. Atomic Red Team**
- Invoke-AtomicRedTeam installed on Windows 10
- T1059.001 (PowerShell execution) test run
- Windows Defender detection confirmed
- Sysmon telemetry visible in Splunk

---

## Key Splunk Queries

```splunk
# Failed logon attempts
index=endpoint EventCode=4625

# Successful logons
index=endpoint EventCode=4624

# All Sysmon telemetry
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"

# Filter by specific user
index=endpoint EventCode=4625 Account_Name="Tsmith"
```

---

## inputs.conf (Windows endpoints)

```ini
[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

---

## Related

- [Lab 1: SOC Automation with Wazuh, TheHive, and Shuffle](https://github.com/rader2001-hash/SOC-Automation-Homelab)

---

## References

- [MyDFIR YouTube Series](https://www.youtube.com/@MyDFIR)
- [Splunk Documentation](https://docs.splunk.com)
- [Atomic Red Team](https://github.com/redcanaryco/invoke-atomicredteam)
- [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
