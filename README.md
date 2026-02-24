# 🔥 Enterprise Firewall Engineering Lab — FortiGate Next-Generation Firewall

![FortiGate](https://img.shields.io/badge/FortiGate-FortiOS%20v7.4.9-EE3124?style=for-the-badge&logo=fortinet&logoColor=white)
![Security](https://img.shields.io/badge/Security_Engineering-Network%20Security-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
![Tasks](https://img.shields.io/badge/Tasks_Completed-14%2F14-success?style=for-the-badge)

> **A hands-on, enterprise-grade firewall engineering project** demonstrating real-world configuration of a FortiGate Next-Generation Firewall (NGFW).
>
> Covers policy design, traffic segmentation, VPN, Virtual IP / Port Forwarding, Application Control, Web Filtering, IPS, Antivirus, and DNS Filtering — all built under a strict **Least Privilege** security model.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Executive Summary](#-executive-summary)
- [Network Architecture](#-network-architecture)
- [Technology Stack](#-technology-stack)
- [Implementation Tasks](#-implementation-tasks)
- [Skills Demonstrated](#-skills-demonstrated)
- [Screenshots](#-screenshots)
- [Lessons Learned](#-lessons-learned)
- [How to Reproduce This Lab](#-how-to-reproduce-this-lab)
- [Author](#-author)

---

## 📌 Project Overview

This repository documents an end-to-end **FortiGate firewall engineering lab** focused on:

- **Network segmentation** (SALES / IT / DMZ)
- **Least privilege inter-segment access**
- **Granular internet control**
- **Threat prevention** (AV / IPS / App Control / Web Filter / DNS Filter)
- **Virtual IP (VIP) / Port Forwarding**
- **Log validation for every rule**

### Lab Rules Followed

- ✅ No **any-any** rules
- ✅ Policies created **one-by-one**
- ✅ Every rule validated with **logs**
- ✅ Least privilege enforced throughout
- ✅ Deny rules tested, then disabled where required

---

## ✅ Executive Summary

### What I Built
A segmented enterprise firewall architecture on **FortiGate NGFW** with **SALES**, **IT**, and **DMZ** zones, secured using explicit policies and FortiGuard security profiles.

### Why I Built It
To simulate a realistic **Security Engineer / Firewall Administrator** workflow:
- design policies,
- enable inspection and threat prevention,
- validate traffic behavior,
- and document results with evidence.

### How I Validated It
No task was marked complete without **verification evidence**, including:
- FortiGate **Forward Traffic** logs
- Application Control / IPS / AV / DNS logs
- SSH / RDP sessions
- `curl`, `nslookup`, and `nmap` outputs
- Linux web server terminal logs

### Key Skills Demonstrated
- FortiGate policy engineering
- Network segmentation
- VIP / NAT / Port Forwarding
- Layer-7 controls (App Control / Web Filter)
- IPS / AV threat prevention
- DNS redirection and private DNS architecture
- Log analysis and troubleshooting

### Environment
- **FortiGate VM (FortiOS 7.4.x)**
- **Ubuntu Linux** (SALES + IT roles)
- **Windows Server** (DMZ IIS + DNS)
- **Windows Client** (RDP testing)
- **FortiClient SSL-VPN**

---

## 🗺️ Network Architecture

### Logical Topology

```mermaid
graph TD
    VPN[FortiClient SSL-VPN Clients] --> FGT[FortiGate NGFW]

    FGT --> SALES[SALES Network (port3)]
    FGT --> IT[IT Network (port5)]
    FGT --> DMZ[DMZ Network]

    SALES --> SLNX[Lnx-Sales2]
    SALES --> SWIN[Win-Sales]

    IT --> ILNX[Lnx-IT2 (Ubuntu 22.04)]
    IT --> IWIN[Win-IT]

    DMZ --> WSRV[WINSRV-DMZ]
    WSRV --> IIS[IIS Web Server]
    WSRV --> DNS[DNS Server (lab.local)]
```

### Segment Roles

| Segment | Role | Accessible Services |
|---|---|---|
| SALES Network | End-user workstations | SSH/ICMP to IT Linux, RDP to IT Windows, filtered internet |
| IT Network | Infrastructure systems | Web services (80/8080/9090), internet access (AWS blocked) |
| DMZ Network | Exposed/internal services | IIS web server, DNS server (`lab.local`) |
| SSL-VPN | Remote validation access | Controlled access into internal segments |

---

## 🛠 Technology Stack

| Technology | Version / Details |
|---|---|
| FortiGate NGFW | FortiOS 7.4.x (VM Lab) |
| VPN | FortiClient SSL-VPN |
| Linux | Ubuntu 22.04 LTS |
| Windows Server | DNS + IIS roles |
| Web Server (IT Linux) | `python3 -m http.server` |
| DNS Testing | `nslookup` |
| Web Testing | `curl` |
| Recon / IPS Test | `nmap` |
| Security Services | FortiGuard AV / IPS / App Control / Web Filter / DNS Filter |

---

## 🔧 Implementation Tasks

### 1. Linux SSH + ICMP Policy (SALES → IT)

**Goal:** Allow only **SSH** and **ICMP** from SALES Linux to IT Linux (inter-segment).

**Configuration**
- Created custom services:
  - `FiratCanBekar-SSH`
  - `FiratCanBekar-ICMP`
- Created policy:
  - Source: SALES Linux / SALES subnet
  - Destination: IT Linux host
  - Services: SSH + ICMP only
  - Action: **ACCEPT**
  - Logging: **Enabled**

**Verification**
- SSH session from SALES Linux/VPN client → IT Linux (**success**)
- `ping` to IT Linux (**success**)
- Traffic visible in FortiGate **Forward Traffic** logs

---

### 2. Windows RDP Policy (SALES → IT)

**Goal:** Allow only **RDP (TCP/3389)** from SALES Windows to IT Windows.

**Configuration**
- Created custom service:
  - `FiratCanBekar-RDP`
- Created policy:
  - Source: SALES Windows / SALES subnet
  - Destination: IT Windows host
  - Service: RDP only
  - Action: **ACCEPT**
  - Logging: **Enabled**
  - NAT: Enabled in lab policy (preserve source port)

**Verification**
- RDP session from SALES Windows / Remmina to IT Windows (**success**)
- FortiGate logs show policy hit with **RDP** traffic and **ACCEPT**

> *Note: NAT behavior in this task depends on the lab routing/VPN design. Policy enforcement was validated through FortiGate policy hits and traffic logs.*

---

### 3. Web Server Deployment & Access Policy

**Goal:** Publish a web service on IT Linux over **80, 8080, 9090** and allow access from SALES.

**Configuration**
- On IT Linux, started web services:
```bash
sudo python3 -m http.server 80
sudo python3 -m http.server 8080
sudo python3 -m http.server 9090
```
- Created custom services for TCP/8080 and TCP/9090
- Created policy:
  - Source: SALES subnet
  - Destination: IT Linux host
  - Services: HTTP, TCP/8080, TCP/9090
  - Action: **ACCEPT**
  - NAT: Disabled (inter-segment traffic)

**Verification**
- From SALES / VPN client:
```bash
curl http://[IT-Linux]:80
curl http://[IT-Linux]:8080
curl http://[IT-Linux]:9090
```
- Linux terminal showed incoming GET requests
- FortiGate logs recorded HTTP / 8080 / 9090 traffic as **ACCEPT**

---

### 4. Virtual IP & Port Forwarding

**Goal:** Use **Virtual IP (VIP)** and port forwarding to reach IT Linux web service through a mapped address.

**Configuration**
- VIP object: `VIP_SALES_TO_IT_LINUX_8080`
  - External IP: `[VIP Address]:8080`
  - Mapped IP: `[IT Linux]:8080`
  - Port Forwarding: **Enabled**
- Policy:
  - Source: SALES subnet
  - Destination: VIP object
  - Service: TCP/8080
  - Action: **ACCEPT**

**Verification**
- SALES client:
```bash
curl http://[VIP-Address]:8080
```
- Web page returned successfully
- FortiGate traffic logs showed VIP-based access

---

### 5. Internet Access Policies (SALES + IT)

**Goal:** Provide general internet access for both SALES and IT networks.

**Configuration**
- Created/verified internet allow policies:
  - SALES → WAN (allow)
  - IT → WAN (allow)
- Logging enabled on both rules

**Verification**
- Web browsing / DNS / HTTPS traffic generated from both segments
- FortiGate logs show outbound internet sessions as **ACCEPT**

---

### 6. URL / Destination Restrictions (SALES)

**Goal:** Block selected destinations for SALES while allowing other internet traffic.

**Configuration**
- Created explicit deny policies for SALES:
  - `example.com` (domain-based / DNS-filter related control path)
  - `1.1.1.1` (IP-based deny)
- Placed deny rules **above** general internet allow
- Logging enabled

**Verification**
- SALES client could not access blocked targets
- Other websites still worked
- FortiGate logs showed **DENY** hits for blocked targets

---

### 7. AWS Service Blocking (IT)

**Goal:** Block IT network access to AWS services while preserving other internet access.

**Configuration**
- Created deny policy for IT → WAN targeting AWS / `amazonaws` endpoints
- Placed deny rule above general IT internet allow policy
- Logging enabled

**Verification**
- IT Linux test:
```bash
curl https://ec2.amazonaws.com
```
- Request blocked
- FortiGate deny logs confirmed AWS traffic was denied
- Non-AWS internet traffic from IT remained allowed

---

### 8. Application Control (IT Windows)

**Goal:** Block specific applications on IT Windows using Layer-7 application signatures.

**Configuration**
- Created App Control sensor: `BLOCK_SOCIAL_API`
- Set actions to **Block** for:
  - Facebook
  - Instagram
  - Gmail
- Attached App Control profile to IT → WAN policy
- Deep inspection path enabled in lab policy context

**Verification**
- Attempts to access Facebook / Instagram / Gmail were blocked
- FortiGate Application Control logs showed application-based detections and blocks

---

### 9. Web Filtering (SALES)

**Goal:** Enforce category-based web restrictions for SALES users.

**Configuration**
- Created custom Web Filter profile
- Blocked categories (examples):
  - Social Networking
  - Gambling
  - Adult Content
  - Online Games
- Attached profile to SALES → WAN policy

**Verification**
- Browsing to blocked categories triggered FortiGuard block page
- FortiGate logs recorded Web Filter actions

---

### 10. Antivirus — EICAR Test

**Goal:** Enable AV inspection and verify blocking using the EICAR test file.

**Configuration**
- Enabled Antivirus profile on relevant outbound policy
- Logging enabled for AV events

**Verification**
- Download attempt from `www.eicar.org`
- FortiGate AV detected and blocked EICAR signature
- AV logs recorded the event

---

### 11. IPS — Intrusion Prevention System

**Goal:** Enable IPS and block malicious / reconnaissance traffic between subnets.

**Configuration**
- Created / enabled IPS sensor with FortiGuard signatures
- Applied IPS profile to inter-subnet policy
- Logging enabled for IPS events

**Verification**
- Triggered test traffic using:
```bash
nmap -A [target]
```
- IPS logs showed signatures and action = **Drop**
- Scan traffic was blocked / suppressed

---

### 12. DNS Filter & DMZ Redirect

**Goal:** Redirect `example.com` to a DMZ-hosted IIS page using **DNS Filter**.

**Configuration**
- Created DNS Filter profile (redirect action)
- Configured `example.com` → DMZ IIS server IP
- Attached DNS Filter profile to SALES internet policy

**Verification**
- SALES DNS requests for `example.com` resolved to DMZ IIS destination
- Browser displayed IIS page hosted in DMZ
- FortiGate DNS Filter logs confirmed redirect behavior

---

### 13. Private DNS Server (DMZ Windows Server)

**Goal:** Make the IT Linux web server reachable **only through** the DNS server hosted on DMZ Windows Server.

**Configuration**
- Installed DNS Server role on DMZ Windows Server
- Created internal zone: `lab.local`
- Added A record:
  - `web.lab.local` → IT Linux IP
- Ensured IT Linux web server was running (port 80)

**Verification**
- SALES client:
```bash
nslookup web.lab.local
curl http://web.lab.local
```
- `nslookup` resolved to IT Linux IP
- `curl` returned web page (HTTP 200 / page content)
- DNS + traffic logs validated path

---

### 14. Log Filtering per Policy

**Goal:** Demonstrate log filtering and per-rule validation for all implemented controls.

**Configuration**
- Used FortiGate **Log & Report → Forward Traffic**
- Filtered by:
  - Policy ID / Policy Name
  - Source / Destination
  - Service / Application
  - Action (Accept / Deny)
- Reviewed App Control / IPS / AV / DNS logs in relevant views

**Verification**
- Confirmed each rule produced matching evidence in logs
- Correlated policy intent with observed traffic behavior
- Used logs as final proof of enforcement

---

## 🧠 Skills Demonstrated

- **Firewall Engineering:** Policy design, segmentation, rule ordering, least privilege
- **FortiGate Administration:** Address/service objects, security profiles, logging, VIP/NAT
- **Threat Prevention:** Antivirus (EICAR), IPS (Nmap detection), DNS Filter, Web Filter, App Control
- **Traffic Validation:** SSH, RDP, `curl`, `nslookup`, `nmap`, log correlation
- **Systems Administration:** Ubuntu server setup, Python HTTP services, Windows Server DNS/IIS
- **Security Principles:** Zero Trust, Defense in Depth, explicit allow/deny strategy

---

## 📸 Screenshots

> All screenshots are included in the `/screenshots` folder of this repository.  
> Credentials, internal IP addresses, and sensitive identifiers have been redacted for security compliance.

> **Quick review:** Start with screenshots **01, 04, 07, 12, 15, 19, 21, 22, 27** for a fast technical walkthrough.

| # | Screenshot | Description |
|---|---|---|
| 01 | `01_policy_list_overview.png` | Full FortiGate firewall policy list |
| 02 | `02_linux_ssh_icmp_policy.png` | SALES Linux → IT Linux policy config |
| 03 | `03_ssh_session_verified.png` | SSH terminal session to IT Linux |
| 04 | `04_rdp_policy.png` | Windows RDP policy configuration |
| 05 | `05_rdp_session_verified.png` | Active RDP session to IT Windows |
| 06 | `06_web_server_ports.png` | Python3 HTTP server on ports 80/8080/9090 |
| 07 | `07_curl_web_server.png` | `curl` test output from SALES client |
| 08 | `08_vip_portforward.png` | Virtual IP & Port Forwarding config |
| 09 | `09_vip_curl_verified.png` | `curl` via VIP address confirmed |
| 10 | `10_internet_policy_sales_it.png` | Internet access policies for SALES & IT |
| 11 | `11_deny_example_1.1.1.1.png` | Deny policies for `example.com` and `1.1.1.1` |
| 12 | `12_deny_logs.png` | FortiGate deny log entries |
| 13 | `13_aws_block_policy.png` | IT → AWS blocking policy |
| 14 | `14_app_control_sensor.png` | App Control sensor (Facebook/Instagram/Gmail) |
| 15 | `15_app_control_logs.png` | Application blocked in logs |
| 16 | `16_web_filter_profile.png` | Web Filter profile with blocked categories |
| 17 | `17_web_filter_blocked.png` | FortiGuard block page in browser |
| 18 | `18_antivirus_profile.png` | Antivirus profile configuration |
| 19 | `19_eicar_blocked.png` | EICAR test blocked (AV logs) |
| 20 | `20_ips_sensor.png` | IPS sensor configuration |
| 21 | `21_nmap_ips_blocked.png` | Nmap scan dropped by IPS |
| 22 | `22_dns_filter_redirect.png` | DNS Filter redirect for `example.com` |
| 23 | `23_iis_redirect_verified.png` | IIS page served after redirect |
| 24 | `24_dns_server_zone.png` | Windows DNS Manager (`lab.local`) |
| 25 | `25_nslookup_weblablocal.png` | `nslookup web.lab.local` → IT Linux IP |
| 26 | `26_curl_weblablocal.png` | `curl http://web.lab.local` → web response |
| 27 | `27_log_filtering_examples.png` | FortiGate log filtering per policy |

---

## 💡 Lessons Learned

### 1. Least Privilege
Granular service-based rules (SSH, ICMP, RDP, specific ports) are far safer than broad allow rules.

### 2. No Any-Any Rules
Avoiding `any-any` forces proper object design and makes troubleshooting easier.

### 3. Defense in Depth
Layering App Control, Web Filter, AV, IPS, and DNS Filter creates stronger controls than port rules alone.

### 4. Deny-Before-Allow (Rule Ordering)
FortiGate policy order is critical. Specific deny rules must be placed above general internet allow rules.

### 5. Logs Are Ground Truth
Firewall logs were the most reliable validation source throughout the lab and helped detect misconfiguration quickly.

### 6. SSL Inspection Constraints
Full HTTPS visibility may require endpoint CA deployment. In lab environments, partial visibility is common.

### 7. DNS as a Security Control Point
DNS filtering and controlled internal resolution are powerful techniques for redirection and policy enforcement.

---

## 🚀 How to Reproduce This Lab

> This lab was built on a virtual FortiGate appliance. To replicate it, you will need:

1. **FortiGate VM** (FortiOS 7.x evaluation)
2. **Virtual network** with at least 3 segments (SALES / IT / DMZ)
3. **Linux VM** in IT segment (Ubuntu 22.04 recommended)
4. **Windows Server VM** in DMZ (DNS + IIS roles)
5. **Windows/Linux client(s)** in SALES segment
6. **FortiClient** for SSL-VPN validation (optional but recommended)

### Build Steps (High-Level)
1. Configure FortiGate interfaces and zones
2. Create address objects (hosts/subnets)
3. Create custom service objects (SSH, ICMP, RDP, 8080, 9090)
4. Build inter-segment policies first
5. Build internet policies and explicit deny rules
6. Attach security profiles (AV / IPS / App Control / Web Filter / DNS Filter)
7. Configure VIP and DMZ services
8. Test each rule and verify via logs
9. Capture screenshots and redact sensitive data

> *Note: FortiGuard subscription/signature availability may vary in evaluation lab environments and can affect App Control, IPS, AV, and Web Filter behavior.*

---

## 👨‍💼 Author

**Fırat Can Bekar**  
| Cyber Security Engineer | Network Security |


[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=flat-square&logo=linkedin)](https://linkedin.com)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat-square&logo=github)](https://github.com/NaphyxHub)

---

*This project was completed as part of a hands-on cybersecurity firewall engineering lab in an isolated environment. No production systems were used.*
