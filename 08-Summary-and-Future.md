# 08. Summary & Future Enhancements

## 📝 Project Summary
Over the course of this project, a fully functional, isolated Security Operations Center (SOC) home lab was designed, built, and tested from scratch. 

The lab successfully simulated an enterprise network with:
*   **Segmented Architecture:** A perimeter firewall (pfSense) separating an external attacker (Kali Linux) from a protected internal LAN (Windows AD & Windows 10).
*   **Centralized Telemetry:** Splunk Enterprise successfully ingesting Windows Event Logs, Sysmon telemetry, and CrowdSec engine logs via Universal Forwarders.
*   **Active Defense:** CrowdSec IDS dynamically detecting brute-force attacks and instructing the Windows Firewall Bouncer to block malicious IPs in real-time.
*   **Threat Hunting:** Writing custom SPL queries and using regular expressions (`rex`) to parse raw XML data, uncover IoCs, and reconstruct the timeline of an attack.

By executing both an external RDP Brute Force and a remote Post-Exploitation/Persistence attack, the lab proved that the entire pipeline from perimeter defense to endpoint AV, SIEM ingestion, and threat hunting is operational. 

---

## 🚀 Future Enhancements
If this lab were to be expanded into a larger environment (or moved to a cloud provider with more resources), the following enhancements would be implemented:

1.  **SIEM Migration (Wazuh or ELK Stack):** While Splunk is an industry standard, the Free license limits automated alerting. Migrating the SIEM to Wazuh (an open-source XDR/SIEM) would allow for fully automated, real-time email alerts and native field extraction without needing `rex` commands.
2.  **SOAR Integration (Shuffle / TheHive):** Implement a Security Orchestration, Automation, and Response (SOAR) platform. When CrowdSec or Splunk triggers an alert, Shuffle could automatically create a case in TheHive and query VirusTotal for the attacker's IP address.
3.  **Threat Intelligence (MISP):** Integrate the Malware Information Sharing Platform (MISP) so the SIEM can automatically correlate ingested logs with known threat actor IPs and file hashes.
4.  **Linux Endpoint Telemetry:** Add an Ubuntu Server as a target endpoint to forward Linux authentication logs and bash history to Splunk, simulating a hybrid Windows/Linux enterprise environment.
5.  **VPN Access:** Remove the exposed RDP and SMB ports on the pfSense WAN. Instead, configure OpenVPN or WireGuard on pfSense so administrators and attackers must authenticate to the VPN before accessing the internal LAN.
6.  **Vulnerability Management:** Deploy a vulnerability scanner (like OpenVAS or Nessus) on the LAN to scan the Windows 10 and Server 2022 VMs, identifying missing patches and misconfigurations for the SOC team to remediate.

---

## 🎓 Final Thoughts
Building this lab required extensive troubleshooting across multiple domains: networking (pfSense NAT and DNS overrides), virtualization (VMware network segments), Windows administration (GPOs, Services, Registry), and SIEM engineering (Splunk index case-sensitivity and Forwarder permissions).

This project provided hands-on, practical experience that directly translates to the day-to-day responsibilities of a **SOC Analyst**. It reinforced the importance of **Defense in Depth** and proved that even if one control fails (e.g., a firewall rule), subsequent layers (endpoint AV, Sysmon, and SIEM) provide the visibility needed to detect and respond to the intrusion.
