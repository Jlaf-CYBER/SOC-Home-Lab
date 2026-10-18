# 🛡️ SOC Home Lab

## 📖 Project Overview
Welcome to my Security Operations Center (SOC) Home Lab. This project was built to simulate a realistic enterprise network environment, focusing on network segmentation, centralized log ingestion, and automated threat mitigation. 

The primary goal of this lab is to demonstrate and learn about some cyberattacks, and practicing SOC operations so i builed a fully functional SOC Environment.

By integrating **CrowdSec** as an active Intrusion Prevention System (IPS), this lab also demonstrates automated defense mechanisms that detect and block malicious traffic in real-time.

---

## 🛠️ Systems & Tools Deployed

The lab leverages a mix of open-source and enterprise-grade tools across multiple operating systems:

*   **Virtualization:** VMware Workstation
*   **Perimeter Security:** pfSense (Firewall, NAT, DNS Resolver)
*   **SIEM & Log Management:** Splunk Enterprise & Splunk Universal Forwarders
*   **Endpoint Security:** Microsoft Sysmon, CrowdSec Engine, CrowdSec Windows Firewall Bouncer
*   **Malware Analysis Sandbox:** FLARE-VM (Isolated Environment)

---

## 🔄 How the Systems Interact

This lab simulates a full attack lifecycle: 

An external attacker on Kali Linux launches attacks against the interior network. The firewall routes the traffic to the targets. The endpoint's security agents CrowdSec detect the malicious activity and trigger the Windows Firewall Bouncer to actively block the attacker. Simultaneously, Splunk Universal Forwarders stream all telemetry (Windows Event Logs, Sysmon, and CrowdSec alerts) to the central Splunk Enterprise server for monitoring, detection, and analysis.

---

## 📂 Repository Navigation

To explore the configurations and attack simulations in detail, navigate through the following pages:

*   **[01. Architecture & Network Setup](01-Network-Architecture.md)** - Network topology, IP addressing, and machine inventory.
*   **[02. Perimeter Firewall (pfSense)](02-Perimeter-Firewall.md)** - NAT rules, firewall policies, and DNS routing.
*   **[03. SIEM (Splunk Enterprise)](03-SIEM-Splunk.md)** - Index setup, port forwarding, and Universal Forwarder configs.
*   **[04. Active Directory](04-Active-Directory.md)** - Domain controller setup and environment population.
*   **[05. Endpoint Security & CrowdSec](05-Endpoint-Defense.md)** - Sysmon configs and Active IPS deployment.
*   **[06. Isolated Malware Sandbox](06-Malware-Sandbox.md)** - Air-gapped FLARE-VM setup.
*   **[07. Attack Simulations](07-Attack-and-Hunting.md)** - Brute-force execution, CrowdSec bans, and Splunk threat hunting.
