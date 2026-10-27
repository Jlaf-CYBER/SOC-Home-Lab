# 07. Attack Simulations Overview

With the network architecture, SIEM, and Endpoint IPS fully operational, the lab is ready for live attack simulations. The goal of these simulations is to practice SOC operations and validate the entire defense pipeline: from the initial perimeter breach attempt to the endpoint's automated remediation and the final SIEM alerting.

The following scenarios will be executed from the **Kali Linux** attacker machine located on the WAN network.

## Scenario 1: External RDP Brute Force
*   **Attacker:** Kali Linux
*   **Target:** Windows 10 Endpoint (Port 3389 exposed via pfSense NAT)
*   **Tool:** Hydra (Password Cracking)
*   **Defense Focus:** Testing CrowdSec's ability to detect repeated authentication failures (Event ID 4625) and trigger the Windows Firewall Bouncer to drop the attacker's IP.

[**Go to Attack 1 Details ->**](08-Attack-1-RDP-Brute-Force.md)

---

## Scenario 2: Endpoint Persistence (Scheduled Task)
*   **Attacker:** Kali Linux (Simulated remote code execution) or Manual Execution
*   **Target:** Windows 10 Endpoint
*   **Tool:** `schtasks.exe` (Windows Native Utility)
*   **Defense Focus:** Testing Sysmon and Splunk's ability to detect suspicious process creation and Windows Event ID 4698 (A scheduled task was created) for persistence mechanisms.

[**Go to Attack 2 Details ->**](09-Attack-2-Persistence.md)
