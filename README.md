# Enterprise SOC Home Lab

## Overview

This project expands upon my Active Directory Home Lab by introducing enterprise security monitoring, vulnerability management, system hardening, and detection engineering.

The environment simulates a small Security Operations Center (SOC) where Windows and Linux systems are monitored through Wazuh SIEM, audited through Sysmon and Windows Event Logging, scanned using OpenVAS, and hardened through Group Policy and host-based security controls.

---

## Objectives

- Build and manage an Active Directory environment
- Centralize log collection using Wazuh
- Deploy Sysmon for advanced telemetry
- Detect persistence techniques
- Monitor authentication activity
- Perform vulnerability assessments
- Remediate discovered vulnerabilities
- Implement security hardening controls
- Document accepted risk decisions

---

## Lab Architecture

### Infrastructure
<img width="1536" height="1024" alt="SOC HomeLab" src="https://github.com/user-attachments/assets/aa3236d2-65d4-4a4e-93cd-9e29dcd2f8b8" />

| System | Role |
|----------|----------|
| Windows Server 2025 | Domain Controller |
| Windows 11 | Domain Joined Client |
| Ubuntu Server | Wazuh SIEM + OpenVAS |
| Kali Linux | Attack / Assessment Platform |

### Security Stack

- Active Directory
- Group Policy
- Sysmon
- Wazuh
- OpenVAS
- Windows Defender Firewall
- Advanced Audit Policy
- SSH Hardening

---

# Phase 1 – Persistence Detection with Wazuh

## Objective

Demonstrate Wazuh's ability to detect suspicious Windows service creation activity commonly associated with persistence techniques.

---

### Create Suspicious Service

The following command was executed on the Windows client:

```cmd
sc create LabTestService binPath= "cmd.exe /c whoami > C:\Windows\Temp\labtest.txt"
```

### Screenshot

**Insert Screenshot:**
<img width="1728" height="1117" alt="Service Creation Commands" src="https://github.com/user-attachments/assets/53ccfb71-c375-400f-8181-e15d239c8386" />

Command prompt showing service creation

---

### Windows Event Viewer Detection

Event ID 7045 was generated.

This event indicates:

- New service installed
- Service account used
- Service executable path
- Service startup type

### Screenshot

**Insert Screenshot:**
<img width="1728" height="1117" alt="Service Creation Event Viewer" src="https://github.com/user-attachments/assets/71eb6280-c547-485a-83de-cc943af664f7" />

Event Viewer showing Event ID 7045

---

### Wazuh Detection

Wazuh successfully ingested and parsed the event.

Alert fields included:

- Service Name
- Service Path
- Service Account
- Host Information

### Screenshot

**Insert Screenshot:**
<img width="1728" height="1117" alt="Wazuh Service Creation Alert" src="https://github.com/user-attachments/assets/5838b59d-345d-4f4e-bf23-6c87bb968c6b" />

Wazuh Discover view showing LabTestService alert

<img width="1728" height="1117" alt="Wazuh Service Creation Detailed" src="https://github.com/user-attachments/assets/5f8cf660-e49b-492a-abd8-0b59a84cf837" />

Detailed view of alert

---

### Skills Demonstrated

- Persistence Detection
- Event ID 7045 Analysis
- Windows Event Logging
- Wazuh Event Correlation
- Threat Hunting

---

# Phase 13 – Vulnerability Assessment with OpenVAS

## Objective

Perform authenticated vulnerability scans against all lab assets.

---

### OpenVAS Deployment

OpenVAS was deployed on the Ubuntu Security Server.

### Screenshot

**Insert Screenshot:**
`phase13-openvas-dashboard.png`

(OpenVAS login page)

---

## Windows 11 Vulnerability Scan

### Findings

- DCE/RPC Enumeration
- TCP Timestamp Disclosure

### Screenshot

**Insert Screenshot:**
`phase13-windows11-scan.png`

(OpenVAS report for Windows 11)

---

## Domain Controller Scan

### Findings

- DCE/RPC Enumeration
- TCP Timestamp Disclosure

### Screenshot

**Insert Screenshot:**
`phase13-dc-scan.png`

(OpenVAS report for Domain Controller)

---

## Ubuntu Security Server Scan

### Findings

- Wazuh Authd Exposure
- Weak SSH MAC Algorithms
- Missing Secure Cookie Attribute
- TCP Timestamp Disclosure
- ICMP Timestamp Disclosure

### Screenshot

**Insert Screenshot:**
`phase13-wazuh-server-scan.png`

(OpenVAS report showing High Severity findings)

---

### Skills Demonstrated

- Vulnerability Assessment
- OpenVAS Administration
- Network Enumeration
- Security Analysis
- Risk Prioritization

---

# Phase 14 – Vulnerability Remediation and Hardening

## Objective

Remediate identified vulnerabilities and validate security controls.

---

## Security Baseline Verification

### Applied Security Policies

- Advanced Audit Policy
- Password Lockout Policy
- SMBv1 Disabled
- Windows Defender Baseline
- PowerShell Logging
- Firewall Enforcement

### Screenshot

**Insert Screenshot:**
`phase14-gpo-verification.png`

(Applied GPOs)

---

## Audit Policy Validation

Advanced auditing was confirmed through auditpol.

### Screenshot

**Insert Screenshot:**
`phase14-audit-policy.png`

(Auditpol Success and Failure auditing)

---

## Wazuh Authd Security Review

OpenVAS identified the authd service.

Port verification confirmed:

```bash
sudo ss -tulpn | grep 1515
```

### Screenshot

**Insert Screenshot:**
`phase14-authd-port.png`

(Authd Listening)

---

Password authentication was verified.

### Screenshot

**Insert Screenshot:**
`phase14-authd-password.png`

(authd Password Enabled)

---

## ICMP Timestamp Mitigation

Firewall rules were deployed through Group Policy.

### Screenshot

**Insert Screenshot:**
`phase14-firewall-rule.png`

(ICMP Timestamp Blocked by Firewall)

---

## Windows TCP Timestamp Mitigation

Registry hardening disabled TCP timestamps.

### Screenshot

**Insert Screenshot:**
`phase14-windows-timestamps.png`

(Registry showing Tcp1323Opts = 0)

---

## Linux TCP Timestamp Mitigation

Linux timestamps were disabled.

### Screenshot

**Insert Screenshot:**
`phase14-linux-timestamps.png`

(sysctl net.ipv4.tcp_timestamps = 0)

---

## SSH Hardening

Weak MAC algorithms were removed.

### Screenshot

**Insert Screenshot:**
`phase14-ssh-macs.png`

(SSH MAC verification)

---

## Accepted Risk Documentation

The Missing Secure Cookie Attribute finding was reviewed and accepted as a documented risk.

### Screenshot

**Insert Screenshot:**
`phase14-secure-cookie-risk.png`

(Risk Acceptance Documentation)

---

## Firewall Validation

Firewall enforcement was verified through Group Policy.

### Screenshot

**Insert Screenshot:**
`phase14-firewall-enabled.png`

(Windows Defender Firewall Enabled)

---

# Remediation Summary

| Finding | Severity | Status |
|----------|----------|----------|
| Wazuh Authd Exposure | High | Mitigated |
| ICMP Timestamp Disclosure | Low | Remediated |
| TCP Timestamp Disclosure (Windows) | Low | Remediated |
| TCP Timestamp Disclosure (Linux) | Low | Remediated |
| Weak SSH MAC Algorithms | Low | Remediated |
| Missing Secure Cookie Attribute | Medium | Accepted Risk |

---

# Skills Demonstrated

- Active Directory Administration
- Group Policy Management
- Wazuh SIEM
- Sysmon
- Threat Detection
- Event Analysis
- OpenVAS
- Vulnerability Management
- Vulnerability Remediation
- Windows Hardening
- Linux Hardening
- Firewall Management
- Audit Policy Configuration
- SSH Hardening
- Risk Management
- Security Operations
- Threat Hunting

---

# Outcome

This lab demonstrates the complete defensive security lifecycle:

Build → Monitor → Detect → Assess → Remediate → Validate

The project simulates enterprise security operations by combining Active Directory administration, SIEM monitoring, vulnerability management, and system hardening within a unified SOC environment.
