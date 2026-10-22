# 03. SIEM (Splunk Enterprise)

Splunk Enterprise acts as the central Security Information and Event Management (SIEM) system for the lab. It is responsible for receiving, indexing, and allowing the analysis of telemetry from all endpoints in the network.

## 1. Server Setup & Web UI
Splunk is installed on a Windows Server 2022 VM inside the protected LAN (`192.168.1.50`). To keep the SIEM secure, it is placed behind the pfSense firewall. Access to the Web UI is achieved by hitting the pfSense WAN IP on port 8000, which uses a NAT rule to forward the traffic internally.

<p align="center">
<img width="1323" height="582" alt="Splunk Web UI" src="https://github.com/user-attachments/assets/28730d0c-68ee-43b6-b669-2c415c81d045" />
  <br>
  <i>Figure 1: Splunk Enterprise Search Interface</i>
</p>

---

## 2. Index Management
To keep search performance high and organize the data logically, two separate indexes were created.

<p align="center">
<img width="1302" height="377" alt="Indexes" src="https://github.com/user-attachments/assets/473ea3ba-c947-47cd-8a31-d973e4efff0a" />
  <br>
  <i>Figure 2: Custom Indexes created for Windows and CrowdSec telemetry</i>
</p>

*   **`winlog_clients`**: Ingests Windows Security logs, System logs, and Sysmon telemetry from the endpoint and Domain Controller.
*   **`crowdsec`**: Ingests the operational logs and ban alerts from the CrowdSec engine.

---

## 3. Data Ingestion (Receiving Port)
For Splunk to accept logs from the Universal Forwarders, it must be configured to listen on a specific network port. 

<p align="center">
<img width="1324" height="291" alt="Receiving Port" src="https://github.com/user-attachments/assets/5aa198f1-6b3f-4edd-bfd8-bf1666936d2a" />
  <br>
  <i>Figure 3: TCP Port 9997 enabled to receive forwarded telemetry</i>
</p>

**TCP Port 9997** was enabled under `Settings > Forwarding and receiving`. This allows the Universal Forwarders to securely stream logs from the endpoints to the SIEM.

---

## 4. Universal Forwarder Configuration (Sender)
A Splunk Universal Forwarder (UF) is installed on the Windows 10 Client to monitor and ship logs. The Forwarder background service must be running for telemetry to be sent.

<p align="center">
<img width="401" height="463" alt="SplunkForwarder Service" src="https://github.com/user-attachments/assets/b6e3ea7b-db6e-489a-ac21-b6ca4da985d6" />
  <br>
  <i>Figure 4: SplunkForwarder Service running on the Windows 10 Endpoint</i>
</p>

### `inputs.conf`
This file tells the Forwarder *what* logs to collect and *where* to send them (which index). Located at `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`.


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
```
### `outputs.conf`
This file tells the Forwarder the IP address of the Splunk Server. Located at C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf.



```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.1.50:9997

[tcpout-server://192.168.1.50:9997]
  ```
After saving these files, the SplunkForwarder service on the Windows 10 machine was restarted to apply the changes.
