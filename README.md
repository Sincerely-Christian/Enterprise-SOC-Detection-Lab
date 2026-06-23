# Active Directory SOC Detection & Vulnerability Management Lab

## Project Overview

This lab expands my original Active Directory home lab into a full SOC-style detection and vulnerability management environment.

The project includes:

* Active Directory Domain Services
* Windows 11 domain client monitoring
* Wazuh SIEM log collection
* Sysmon endpoint telemetry
* PowerShell logging
* Kali Linux detection testing
* Greenbone/OpenVAS vulnerability scanning
* Group Policy security hardening
* Vulnerability remediation and risk acceptance documentation

---

## Lab Network

| Role                        | System                    | IP Address   |
| --------------------------- | ------------------------- | ------------ |
| Domain Controller / Gateway | Windows Server 2025       | 172.16.0.1   |
| Windows Client              | Windows 11                | 172.16.0.100 |
| Ubuntu Security Server      | Wazuh + Greenbone/OpenVAS | 172.16.0.101 |
| Kali Linux                  | Scanner / Test Host       | 172.16.0.102 |

### Web Consoles

| Service           | URL                       |
| ----------------- | ------------------------- |
| Wazuh Dashboard   | https://172.16.0.101:443      |
| Greenbone/OpenVAS | https://172.16.0.101:8443 |

---

## Tools Used

* Windows Server 2025
* Windows 11
* Active Directory
* DNS
* DHCP
* Group Policy
* Sysmon
* Wazuh SIEM
* Greenbone/OpenVAS
* Kali Linux
* Nmap
* PowerShell
* Windows Event Viewer
* Windows Defender Firewall
* Ubuntu Server

---

# Phase 1 - Detection Engineering

Phase 1 generated controlled security events and verified that the lab could detect them through Windows logs, Sysmon, and Wazuh.

---

## 1.1 Detection One: Nmap Reconnaissance

### Objective

Simulate network reconnaissance from Kali Linux against internal Windows systems and detect the activity through Sysmon and Wazuh.

### Source

| System     | IP           |
| ---------- | ------------ |
| Kali Linux | 172.16.0.102 |

### Targets

| System            | IP           |
| ----------------- | ------------ |
| Domain Controller | 172.16.0.1   |
| Windows 11 Client | 172.16.0.100 |

### Commands Used

```bash
nmap -sV -Pn 172.16.0.1
nmap -sV -Pn 172.16.0.100
```

### Expected Detection Sources

| Source | Event                                 |
| ------ | ------------------------------------- |
| Sysmon | Event ID 3 - Network Connection       |
| Wazuh  | Events showing source IP 172.16.0.102 |
| Kali   | Nmap scan output                      |

### MITRE ATT&CK Mapping

| Tactic    | Technique                 | ID    |
| --------- | ------------------------- | ----- |
| Discovery | Network Service Discovery | T1046 |

### Evidence

#### Kali scan against Domain Controller

<img width="1728" height="1117" alt="DC Nmap Scan" src="https://github.com/user-attachments/assets/63f2dd26-80dd-47c8-ad45-e0925d27cc2c" />

#### Sysmon network connection event

<img width="1728" height="1117" alt="DC Sysmon Event3" src="https://github.com/user-attachments/assets/78278f19-30c0-4d9e-a5c2-19c1d88b1893" />


#### Wazuh detection of Kali source IP

<img width="1728" height="1117" alt="Wazuh Nmap Detection " src="https://github.com/user-attachments/assets/29e69648-5708-4be4-96cc-8acb869364f5" />


#### Kali scan against Windows 11 Client

<img width="1728" height="1117" alt="Win11 Client Nmap Scan" src="https://github.com/user-attachments/assets/67a53af2-81ae-46e4-896b-4f1d4d5a7b6f" />

### Analyst Notes

The Kali system performed service discovery against internal hosts. The activity was visible through Sysmon Event ID 3 and was ingested into Wazuh, proving that network reconnaissance can be detected in the lab.

---

## 1.2 Detection Two: Failed Domain Logons

### Objective

Generate failed domain authentication attempts from the Windows 11 client and verify that the Domain Controller and Wazuh captured the activity.

### Source

| System            | IP           |
| ----------------- | ------------ |
| Windows 11 Client | 172.16.0.100 |

### Target

| System            | IP         |
| ----------------- | ---------- |
| Domain Controller | 172.16.0.1 |

### Commands Used

```cmd
hostname
whoami
ipconfig

runas /user:mydomain\fakeuser cmd
runas /user:mydomain\Admin-C-James cmd
```

### Expected Event IDs

| Event ID | Description                           |
| -------- | ------------------------------------- |
| 4625     | Failed logon                          |
| 4771     | Kerberos pre-authentication failed    |
| 4776     | NTLM authentication                   |
| 4740     | Account lockout, if threshold reached |

### MITRE ATT&CK Mapping

| Tactic            | Technique   | ID    |
| ----------------- | ----------- | ----- |
| Credential Access | Brute Force | T1110 |

### Evidence

#### Windows client context & Failed runas attempts

<img width="1728" height="1117" alt="Win 11 Context and failed runas attempts" src="https://github.com/user-attachments/assets/069d1371-1fc0-4ec4-b920-8fe59d056c41" />


#### Domain Controller authentication failure

<img width="1728" height="1117" alt="DC Event Viewer Failed Login" src="https://github.com/user-attachments/assets/d71125ff-6e3c-477c-ad02-784b88515dbf" />


#### Wazuh failed authentication event

<img width="1728" height="1117" alt="Wazuh Failed Login Alert" src="https://github.com/user-attachments/assets/9b6765b3-6344-4462-8812-66be0f67ba29" />
<img width="1728" height="1117" alt="Wazuh Detailed Failed Login" src="https://github.com/user-attachments/assets/8a690a89-8c53-4ec8-9523-62bd024640e5" />

### Analyst Notes

Failed authentication attempts were generated from the Windows 11 client. The Domain Controller recorded Kerberos authentication failures, and Wazuh collected the related events. This confirms that the lab can detect failed domain logon activity.

---

## 1.3 Detection Three: Suspicious PowerShell Activity

### Objective

Generate suspicious PowerShell telemetry and confirm detection through PowerShell Operational logs, Sysmon, and Wazuh.

### Source

| System            | IP           |
| ----------------- | ------------ |
| Windows 11 Client | 172.16.0.100 |

### Commands Used

```powershell
whoami
hostname
Get-LocalUser
Get-Process
Get-Service
Get-NetIPAddress
Get-LocalGroupMember Administrators

powershell -ExecutionPolicy Bypass -Command "Get-Process"
```

### Expected Event IDs

| Event ID | Source     | Description          |
| -------- | ---------- | -------------------- |
| 4103     | PowerShell | Module logging       |
| 4104     | PowerShell | Script block logging |
| 4688     | Security   | Process creation     |
| 1        | Sysmon     | Process creation     |

### MITRE ATT&CK Mapping

| Tactic    | Technique  | ID        |
| --------- | ---------- | --------- |
| Execution | PowerShell | T1059.001 |

### Evidence

#### PowerShell commands executed

**Screenshot placement: 12.3 Screenshot 1**

<img width="1728" height="1117" alt="Powershell commands" src="https://github.com/user-attachments/assets/e365cc07-a841-45ae-bace-42eb522f2e7f" />

#### PowerShell Event ID 4104

<img width="1728" height="1117" alt="Powershell event log 2" src="https://github.com/user-attachments/assets/ea072806-a0a7-4750-b350-d025b16361a4" />

#### Sysmon process creation

<img width="1728" height="1117" alt="Powershell Sysmon Event" src="https://github.com/user-attachments/assets/1c6cab9f-9180-4cca-a941-2a3b92b618bd" />

#### Wazuh PowerShell event

<img width="1728" height="1117" alt="Wazuh Powershell Alert" src="https://github.com/user-attachments/assets/0ba3fbd7-09b1-477d-b409-284dc541aeaa" />

#### Wazuh expanded PowerShell document

<img width="1728" height="1117" alt="Wazuh Powershell Detailed Look" src="https://github.com/user-attachments/assets/ba2482d8-ea0b-4e48-994a-f139bf9f7a86" />

### Analyst Notes

PowerShell enumeration and an ExecutionPolicy Bypass command were generated on the Windows 11 client. The activity appeared in PowerShell Operational logs, Sysmon, and Wazuh.

---

## 1.4 Detection Four: Windows Service Creation

### Objective

Create a harmless Windows service to simulate persistence-style behavior and detect it through Windows Event Logs and Wazuh.

### Source

| System            | IP           |
| ----------------- | ------------ |
| Windows 11 Client | 172.16.0.100 |

### Commands Used

```cmd
sc create LabTestService binPath= "cmd.exe /c whoami > C:\Windows\Temp\labtest.txt"
sc start LabTestService
sc query LabTestService
type C:\Windows\Temp\labtest.txt
sc delete LabTestService
```

### Expected Event IDs

| Event ID | Source   | Description                    |
| -------- | -------- | ------------------------------ |
| 7045     | System   | Service installed              |
| 4688     | Security | Process creation               |
| 1        | Sysmon   | Process creation               |
| 12/13    | Sysmon   | Registry object/value activity |

### MITRE ATT&CK Mapping

| Tactic      | Technique                                        | ID        |
| ----------- | ------------------------------------------------ | --------- |
| Persistence | Create or Modify System Process: Windows Service | T1543.003 |

### Evidence

#### Service creation command

<img width="1728" height="1117" alt="Service Creation Commands" src="https://github.com/user-attachments/assets/e2122bc2-95b5-4022-8832-5ce17452ec0e" />

#### Event Viewer 7045

<img width="1728" height="1117" alt="Service Creation Event Viewer" src="https://github.com/user-attachments/assets/45cd6260-c8bf-42e0-b900-d504db5a70f0" />

#### Wazuh service creation detection

<img width="1728" height="1117" alt="Wazuh Service Creation Alert" src="https://github.com/user-attachments/assets/9cc3f010-ca51-493d-aa63-deca61dad81f" />

#### Wazuh single document view

<img width="1728" height="1117" alt="Wazuh Service Creation Detailed" src="https://github.com/user-attachments/assets/f99b76b3-683a-4e78-9106-85c3f90af510" />

### Analyst Notes

The `LabTestService` service was created, started, queried, and deleted. This generated Windows Event ID 7045 and was ingested into Wazuh. This demonstrates visibility into a common persistence technique.

---

# Phase 2 - Vulnerability Management with Greenbone/OpenVAS

## Objective

Perform vulnerability scans against lab systems using Greenbone/OpenVAS and document findings by host.

### Scanned Hosts

| Host                        | IP           |
| --------------------------- | ------------ |
| Domain Controller           | 172.16.0.1   |
| Windows 11 Client           | 172.16.0.100 |
| Ubuntu Wazuh/OpenVAS Server | 172.16.0.101 |

---

## Greenbone/OpenVAS Access

Greenbone/OpenVAS was accessed through the Ubuntu Security Server.

```text
https://172.16.0.101:8443
```

### Evidence

#### Greenbone/OpenVAS login page

<img width="1728" height="1117" alt="Wazuh Login:Url" src="https://github.com/user-attachments/assets/3e5f8da0-d710-4c85-adb9-c2abfbaf9d1e" />

---

## Scan Tasks Completed

Four scan tasks were visible in Greenbone/OpenVAS.

### Evidence

#### Completed Greenbone scan tasks

<img width="1728" height="1117" alt="Wazuh Tasks all four complete Scans" src="https://github.com/user-attachments/assets/a30e2730-c92f-4826-98b1-2cbb92d21f06" />

---

## Domain Controller Scan

### Host

```text
172.16.0.1
```

### Findings

| Finding                                          | Severity |
| ------------------------------------------------ | -------- |
| DCE/RPC and MSRPC Services Enumeration Reporting | Medium   |
| TCP Timestamps Information Disclosure            | Low      |

### Evidence

#### Domain Controller OpenVAS results

<img width="1728" height="1117" alt="DC WinServer OpenVAS Results" src="https://github.com/user-attachments/assets/1123d134-f05b-4d50-baf5-3fc38b0c9848" />

---

## Windows 11 Client Scan

### Host

```text
172.16.0.100
```

### Findings

| Finding                                          | Severity |
| ------------------------------------------------ | -------- |
| DCE/RPC and MSRPC Services Enumeration Reporting | Medium   |
| TCP Timestamps Information Disclosure            | Low      |

### Evidence

#### Windows 11 OpenVAS results

<img width="1728" height="1117" alt="Windows 11 OpenVAS Results" src="https://github.com/user-attachments/assets/a980c1a1-f8cd-4ecb-8db2-102e0143cb07" />

---

## Ubuntu Wazuh/OpenVAS Server Scan

### Host

```text
172.16.0.101
```

### Findings

| Finding                                      | Severity |
| -------------------------------------------- | -------- |
| Unprotected OSSEC/Wazuh ossec-authd Protocol | High     |
| Missing Secure Cookie Attribute              | Medium   |
| TCP Timestamps Information Disclosure        | Low      |
| Weak SSH MAC Algorithms                      | Low      |
| ICMP Timestamp Reply Information Disclosure  | Low      |

### Evidence

#### Ubuntu Wazuh/OpenVAS server results

<img width="1728" height="1117" alt="Ubuntu Server OpenVAS Results" src="https://github.com/user-attachments/assets/c7587acc-fcc2-4b8b-85e7-c2017868ad64" />

### Analyst Notes

Greenbone identified exposed Windows services, timestamp disclosure issues, SSH configuration weaknesses, Wazuh enrollment exposure, and an application-level secure cookie finding. These findings were reviewed and used to guide Phase 14 hardening.

---

# Phase 3 - Security Hardening and Remediation

## Objective

Implement security controls and document remediation actions based on Greenbone/OpenVAS findings.

---

## 3.1 Group Policy Verification

The Domain Controller was configured with multiple SOC-focused Group Policy Objects.

### Applied GPOs

* SOC - Advanced Audit Policy
* SOC - Password and Lockout Policy
* SOC - Disable SMBv1
* SOC - Firewall Enabled
* SOC - PowerShell Logging
* SOC - Screen Lock Policy
* SOC - Windows Defender Baseline

### Evidence

#### Applied SOC Group Policy Objects

<img width="1728" height="1117" alt="Applied GPOs" src="https://github.com/user-attachments/assets/9277d7f2-6ce3-4aef-a8b5-b9e2a6e97f06" />

---

## 3.2 Audit Policy Validation

Advanced audit policy settings were verified.

### Command Used

```powershell
auditpol /get /category:* | findstr "Success and Failure"
```

### Evidence

#### Advanced audit policy validation

<img width="1728" height="1117" alt="Auditpol Results" src="https://github.com/user-attachments/assets/59c803ee-e84b-4c96-9c6d-4fff17fcbcd0" />

---

## 3.3 Firewall Enabled

Windows Defender Firewall was confirmed enabled for Domain, Private, and Public profiles.

### Evidence

#### Windows Defender Firewall enabled

<img width="1728" height="1117" alt="Authd Listening" src="https://github.com/user-attachments/assets/804afa2f-bb14-45a9-8f7e-7956b29d17a9" />

---

## 3.4 ICMP Timestamp Mitigation

ICMP timestamp requests were blocked through Windows Defender Firewall with Advanced Security.

### Control

```text
Block ICMP Timestamp Requests
Protocol: ICMPv4
Action: Block
Profiles: Domain, Private, Public
```

### Evidence

#### ICMP timestamp firewall block rule

<img width="1728" height="1117" alt="Icmp Timestamp Blocked by FW" src="https://github.com/user-attachments/assets/a738a7e9-4a19-4102-a45b-04e430fbd523" />

---

## 3.5 Windows TCP Timestamp Mitigation

TCP timestamps were disabled on Windows through the registry.

### Registry Path

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters
```

### Value

```text
Tcp1323Opts = 0
```

### Evidence

#### Windows TCP timestamps disabled in registry

<img width="1728" height="1117" alt="Windows Tcp Timestamp Disabled Hardening" src="https://github.com/user-attachments/assets/2306a306-0365-4b89-ba34-49234732e825" />

---

## 3.6 Linux TCP Timestamp Mitigation

TCP timestamps were disabled on the Ubuntu Wazuh/OpenVAS server.

### Command Used

```bash
sysctl net.ipv4.tcp_timestamps
```

### Expected Result

```bash
net.ipv4.tcp_timestamps = 0
```

### Evidence

#### Linux TCP timestamps disabled

<img width="1728" height="1117" alt="Linux TCP Timestamp Disabled Hardening" src="https://github.com/user-attachments/assets/95e1d521-cc1f-475a-969b-5accb4b713f9" />

---

## 3.7 SSH MAC Hardening

Weak SSH MAC algorithms were removed from the Ubuntu server configuration.

### Verification Command

```bash
sudo sshd -T | grep macs
```

### Expected Result

```bash
macs hmac-sha2-512,hmac-sha2-256
```

### Evidence

#### SSH MAC hardening verification

<img width="1728" height="1117" alt="Ubuntu Server Weak Mac Hardening" src="https://github.com/user-attachments/assets/48f79bd9-17df-4c12-88be-79df38556a00" />

---

## 3.8 Wazuh Enrollment Service Review

Greenbone identified Wazuh `ossec-authd` on TCP 1515 as a high-severity issue.

### Verification Command

```bash
sudo ss -tulpn | grep 1515
```

### Evidence

#### Wazuh authd listening on TCP 1515

<img width="1728" height="1117" alt="Authd Listening" src="https://github.com/user-attachments/assets/7d720f6b-bc50-43ab-96a9-1bbc60316e71" />

### Configuration Reviewed

The Wazuh configuration was reviewed to confirm enrollment authentication.

### Evidence

#### Wazuh authd password configuration

<img width="1728" height="1117" alt="authd Password Enabled" src="https://github.com/user-attachments/assets/f98953f4-3f2d-4017-8732-65404d0a81bb" />

### Risk Decision

The service was not fully disabled because future agent enrollment may still be needed in the lab. Instead, authentication was verified and the finding was documented as requiring future hardening.

### Residual Risk

Low to Medium, depending on whether enrollment remains exposed after lab completion.

### Future Remediation

When no more Wazuh agents need to be enrolled:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Change:

```xml
<disabled>no</disabled>
```

to:

```xml
<disabled>yes</disabled>
```

Then restart Wazuh and verify port 1515 is no longer listening.

---

## 3.9 Secure Cookie Finding

Greenbone identified:

```text
Missing 'Secure' Cookie Attribute
```

This finding originated from the Wazuh Dashboard web application.

### Risk Decision

This was documented as an accepted risk because modifying Wazuh dashboard cookie behavior could affect dashboard functionality and was outside the intended scope of this lab.

### Evidence

#### Secure cookie accepted risk note

<img width="1728" height="1117" alt="Secure Cookie Reasoning" src="https://github.com/user-attachments/assets/dc2b3d0a-e012-4e88-9d2b-5e823c3d5572" />

---

# Remediation Summary

| Finding                                     | Host            | Severity | Action                                               |
| ------------------------------------------- | --------------- | -------- | ---------------------------------------------------- |
| TCP Timestamp Information Disclosure        | Windows hosts   | Low      | Registry hardening applied                           |
| TCP Timestamp Information Disclosure        | Ubuntu server   | Low      | sysctl hardening applied                             |
| ICMP Timestamp Reply Information Disclosure | Windows hosts   | Low      | Firewall rule applied                                |
| Weak SSH MAC Algorithms                     | Ubuntu server   | Low      | SSH MACs restricted                                  |
| Missing Secure Cookie Attribute             | Wazuh Dashboard | Medium   | Accepted risk                                        |
| Wazuh ossec-authd Exposure                  | Ubuntu server   | High     | Authentication verified, future hardening documented |
| DCE/RPC Enumeration                         | Windows hosts   | Medium   | Accepted as expected AD lab exposure                 |

---

# Troubleshooting Notes

## TCP Timestamp Troubleshooting

TCP timestamps remained visible during early validation. Troubleshooting included:

```bash
sysctl net.ipv4.tcp_timestamps
sudo sysctl --system
sudo tcpdump -ni enp0s5 tcp
```

Observation:

* Ubuntu host showed `net.ipv4.tcp_timestamps = 0`
* Docker container traffic still showed timestamp-like TCP options in some captures
* Greenbone findings may vary depending on whether it scans host, container, or bridged traffic

## ICMP Timestamp Troubleshooting

Firewall rules were reviewed in Group Policy to ensure ICMP timestamp requests were blocked.

## Wazuh Authd Troubleshooting

The following was used to confirm exposure:

```bash
sudo ss -tulpn | grep 1515
```

The service was still listening through Docker proxy, so the finding was documented rather than hidden.

---

# Key Skills Demonstrated

* Active Directory administration
* Domain Controller management
* Windows endpoint monitoring
* Wazuh SIEM deployment
* Sysmon telemetry collection
* PowerShell logging
* Detection engineering
* Threat hunting
* MITRE ATT&CK mapping
* Nmap reconnaissance detection
* Failed logon detection
* PowerShell activity detection
* Windows service creation detection
* Vulnerability scanning with Greenbone/OpenVAS
* Vulnerability remediation
* Risk acceptance documentation
* Group Policy hardening
* Windows Firewall configuration
* Linux sysctl hardening
* SSH hardening

---

# Final Outcome

This lab demonstrates a complete SOC workflow:

```text
Generate Activity
Collect Logs
Detect Events
Investigate Alerts
Scan for Vulnerabilities
Prioritize Findings
Apply Hardening
Validate Remediation
Document Risk
```

This project shows practical experience with both blue-team detection and vulnerability management in a realistic Active Directory lab environment.
 
