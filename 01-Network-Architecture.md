# 01. Architecture & Network Setup

## 🗺️ Lab Topology
The lab is built using VMware Workstation to simulate a segmented enterprise environment. The network is divided into three distinct security zones:

*   **WAN (External / Red Zone):** `192.168.80.0/24` - Where the attacker (Kali) resides. 
*   **LAN (Protected / Green Zone):** `192.168.1.0/24` - The internal network containing the AD DC, Windows 10 endpoint, and Splunk Server.
*   **Isolated Sandbox (Grey Zone):** An air-gapped network for malware analysis using FLARE-VM.

<p align="center">
  <img width="1424" height="1944" alt="Network Topology Diagram" src="https://github.com/user-attachments/assets/9ee38b73-c7e4-4dec-9002-9b0acacaad87" />
  <br>
  <i>Figure 1: Complete SOC Lab Network Topology</i>
</p>

---

## 💻 Machine Inventory & Specifications

To ensure the lab could run efficiently on my host machine, static IPs were carefully configured.

| Machine Name | Role | IP Address | Zone |
| :--- | :--- | :--- | :--- |
| **pfSense** | Perimeter Firewall / Router | WAN: `192.168.80.133`<br>LAN: `192.168.1.1` | WAN / LAN |
| **Kali-Linux** | Attacker Node | `192.168.80.128` | WAN |
| **DC-01** | Active Directory Domain Controller | `192.168.1.10` | LAN |
| **WIN10-CLIENT** | Target Endpoint | `192.168.1.30` | LAN |
| **SIEM-SPLUNK** | Splunk Enterprise SIEM | `192.168.1.50` | LAN |
| **FLARE-VM** | Malware Analysis Sandbox | Isolated Segment | Sandbox |

---

## ⚙️ Network Configuration & Screenshots

Here is the breakdown of the IP configurations for the core devices in the lab.

### 1. pfSense Firewall (on Perimeter)
The pfSense firewall acts as the default gateway for the LAN. The WAN interface receives an IP via DHCP from the VMware NAT network, while the LAN interface is statically assigned.
*   **WAN:** `192.168.80.133` (Faces the attacker/Kali).
*   **LAN:** `192.168.1.1` (Faces the internal enterprise network).

<p align="center">
  <img width="473" height="76" alt="pfsense firewall network config" src="https://github.com/user-attachments/assets/b1fb4fa4-5293-45f1-a835-d545f61a6c65" />
  <br>
  <i>Figure 2: pfSense WAN and LAN Interface Assignments</i>
</p>

---

### 2. Domain Controller
The Domain Controller requires a static IP so clients can reliably find it for DNS and Active Directory authentication. 
*   **IP:** `192.168.1.10`
*   **Gateway:** `192.168.1.1` (pfSense)
*   **DNS:** Configured to query public DNS (`8.8.8.8`) for internet resolution, while serving local `soc.lab` queries internally.

<p align="center">
  <img width="395" height="452" alt="AD Network config" src="https://github.com/user-attachments/assets/bed89521-ca49-4283-a5d9-04530d5e54fa" />
  <br>
  <i>Figure 3: Domain Controller (DC-01) IPv4 Configuration</i>
</p>

---

### 3. Windows 10 Client 
This is the primary victim machine. It is configured to use pfSense as its DNS server so it can resolve internet domains even if the Domain Controller is powered off. 
*   **IP:** `192.168.1.30`
*   **DNS:** `192.168.1.1` (pfSense acts as the resolver and routes `soc.lab` traffic to the DC).

<p align="center">
  <img width="400" height="456" alt="Win 10 Network config" src="https://github.com/user-attachments/assets/978276a4-156c-4436-b84f-5c3405689c75" />
  <br>
  <i>Figure 4: Windows 10 Client (WIN10-CLIENT) IPv4 Configuration</i>
</p>

---

### 4. Splunk Enterprise SIEM Server
The SIEM server is statically assigned to ensure Universal Forwarders always know where to send telemetry (port 9997).
*   **IP:** `192.168.1.50`
*   **DNS:** `192.168.1.1` (pfSense)

<p align="center">
  <img width="400" height="452" alt="Win Splunk serv Network config" src="https://github.com/user-attachments/assets/cbdcb812-e1d5-4885-9b92-0e9454466469" />
  <br>
  <i>Figure 5: Splunk Enterprise Server (SIEM-SPLUNK) IPv4 Configuration</i>
</p>

---

### 5. Kali Linux (Attacker)
The attacker machine sits on the WAN network (VMnet8). It receives its IP via DHCP from VMware, which places it outside the internal LAN.
*   **IP:** `192.168.80.128`
*   **Gateway:** `192.168.80.2` (VMware NAT)

<p align="center">
  <img width="606" height="537" alt="Kali Network config" src="https://github.com/user-attachments/assets/2cd2e093-1d67-4722-949f-9a175820f31c" />
  <br>
  <i>Figure 6: Kali Linux Attacker Network Configuration</i>
</p>

---

## 🌐 Domains & DNS Configuration

*   **Active Directory Domain:** `soc.lab`
*   **Domain Controller Hostname:** `DC-01`
*   **DNS Architecture:** pfSense handles DNS query forwarding to public servers (like `8.8.8.8`). A **Domain Override** is configured in pfSense so that any request for `soc.lab` is automatically forwarded to the DC (`192.168.1.10`). This ensures endpoints have independent internet access even if the DC is offline.
