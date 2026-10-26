# 05. Endpoint Security & CrowdSec 

The Windows 10 Client (`192.168.1.30`) serves as the primary target endpoint. To ensure deep visibility and active defense, it is equipped with Microsoft Sysmon for high-fidelity telemetry and CrowdSec for automated Intrusion Prevention (IPS).

## 1. Microsoft Sysmon
While Windows Event Logs provide basic security auditing, Sysmon provides incredibly detailed telemetry (process creation, network connections, file creation time changes). 

Sysmon is installed using the community-standard **SwiftOnSecurity** configuration file. This configuration filters out noise and focuses on high-value events, such as:
*   **Event ID 1:** Process Creation
*   **Event ID 3:** Network Connection
*   **Event ID 10:** Process Access (Crucial for detecting credential dumping tools like Mimikatz)
*   **Event ID 11:** FileCreate

<p align="center">
<img width="402" height="464" alt="sysmon service" src="https://github.com/user-attachments/assets/1ecff246-78c0-432e-9406-f2fc7a8d5756" />
  <br>
  <i>Figure 1: Sysmon Service running on the Windows 10 Endpoint</i>
</p>

---

## 2. CrowdSec Security Engine (IDS)
CrowdSec is a modern, open-source IDS/IPS that analyzes log streams to detect attacks. 

### Collections
The CrowdSec engine is configured with specific collections tailored for this lab. Using the `cscli` tool, we can verify the installed parsers and scenarios.

<p align="center">
<img width="914" height="221" alt="CrowdSec Collections" src="https://github.com/user-attachments/assets/2bec8959-2f84-4ebc-a029-205ae2c6a314" />
  <br>
  <i>Figure 2: CrowdSec Collections verifying Windows event parsing and brute-force detection</i>
</p>

*   `crowdsecurity/windows`: Parses standard Windows Event Logs.
*   `crowdsecurity/windows-firewall`: Detects port scans and network anomalies in the firewall logs.
*   `crowdsecurity/windows-bf`: Specifically detects brute-force attacks (Event ID 4625).

---

## 3. CrowdSec Windows Firewall Bouncer (IPS)
By default, CrowdSec acts only as an IDS (detecting but not blocking). To make it an **Active IPS**, the Windows Firewall Bouncer is installed. 

The Bouncer listens to the local CrowdSec API. When the engine generates a ban decision, the Bouncer immediately injects a dynamic DROP rule into Windows Defender Firewall.

<p align="center">
<img width="918" height="92" alt="CrowdSec Bouncer" src="https://github.com/user-attachments/assets/f82c7242-fb57-4034-add5-d9c8f3df5c10" />
  <br>
  <i>Figure 3: Verifying the Windows Firewall Bouncer is active and connected to the engine</i>
</p>

---

## 4. Splunk Integration & Engine Logs
To ensure the SOC team is notified of these bans, the Splunk Universal Forwarder is configured to monitor the CrowdSec log file and the Windows Event Logs. 

### `inputs.conf` Configuration
The following configuration was appended to the Universal Forwarder's `inputs.conf` on the Windows 10 machine to ship the telemetry to the SIEM:

```ini
[WinEventLog://Application]
disabled = 0
index = winlog_clients

[WinEventLog://Security]
disabled = 0
index = winlog_clients

[WinEventLog://System]
disabled = 0
index = winlog_clients

[Monitor://C:\ProgramData\CrowdSec\log\crowdsec.log]
disabled = false
sourcetype = crowdsec:log
index = crowdsec
