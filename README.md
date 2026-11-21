# Network Segmentation & Stateful Firewall Policy Enforcement with pfSense

## 📌 Project Overview
This lab demonstrates enterprise-style network segmentation and stateful firewall rule enforcement using **pfSense** in a virtualized **VMware Workstation** environment. The architecture isolates untrusted guest traffic (`GUEST_NET`) from sensitive internal corporate infrastructure (`EMPLOYEE_NET`) while permitting controlled HTTP access to an internal web server and general internet connectivity.

---

## 🏗️ Network Topology & IP Addressing

| Interface / Segment | VMware Network | Subnet / CIDR | Gateway IP | Purpose & Allocation |
| :--- | :--- | :--- | :--- | :--- |
| **WAN (`em0`)** | NAT (`VMnet8`) | DHCP (NAT) | Upstream NAT | External internet uplink |
| **LAN (`em1`)** | Host-Only (`VMnet2`) | `192.168.10.0/24` | `192.168.10.1` | Internal Employee LAN & Apache Web Server (`192.168.10.10`) |
| **GUEST (`opt1 / em2`)** | Host-Only (`VMnet3`) | `192.168.20.0/24` | `192.168.20.1` | Isolated Guest Wi-Fi/LAN segment (DHCP Pool: `.100` – `.200`) |

---

## 🛠️ Lab Environment & Tools
* **Firewall / Router:** pfSense Community Edition (2.9.0-RELEASE)
* **Internal Web Server:** Ubuntu Server 24.04 LTS running Apache2 (`192.168.10.10`)
* **Guest Test Client:** Kali Linux (`192.168.20.101`)
* **Hypervisor:** VMware Workstation Pro

---

## ⚙️ Key Implementation Steps

### 1. Virtual Network Segmentation
Configured isolated Host-Only virtual switches (`VMnet2` and `VMnet3`) in VMware Virtual Network Editor with internal VMware DHCP disabled so pfSense manages all routing and IP leases.
* ![VMware Virtual Networks]<img width="598" height="550" alt="01_VMware_VMnet2_VMnet3" src="https://github.com/user-attachments/assets/cbf270da-e3d7-43ce-9504-37784d494d4e" />


### 2. pfSense Interface & DHCP Configuration
Assigned the third NIC (`em2`) as `GUEST` (`192.168.20.1/24`) and enabled the pfSense DHCP server (`192.168.20.100` – `192.168.20.200`).
* ![pfSense Interfaces]<img width="1313" height="765" alt="PfScense Assignment - Interface" src="https://github.com/user-attachments/assets/2131c0ec-ac78-489a-9281-c952a7548773" />

* ![Guest DHCP Lease]<img width="1237" height="725" alt="03_Guest_DHCP_Lease" src="https://github.com/user-attachments/assets/53d5196b-b88c-4c10-a702-0ed2bcd2f9a6" />


### 3. Internal Web Server Deployment
Assigned a static IP (`192.168.10.10/24`) on Ubuntu Server and deployed Apache2 to simulate an internal corporate portal.
* ![Ubuntu Static IP]<img width="866" height="291" alt="ubuntu ip addr" src="https://github.com/user-attachments/assets/73da7f2a-6039-48c5-a7d2-6e51ad1c3a8f" />

* ![Apache Web Access]<img width="1209" height="756" alt="05_Employee_Web_Access" src="https://github.com/user-attachments/assets/681a6ded-8134-468f-83dd-9ac1bf4ad21e" />


### 4. Firewall Aliases & Top-Down Rule Logic
Created reusable IP Aliases (`EMPLOYEE_NET`, `GUEST_NET`, `WEB_SERVER`) and configured strict top-down rules on the `GUEST` interface:
1. **Pass TCP Port 80 (HTTP):** `GUEST_NET` $\rightarrow$ `WEB_SERVER` *(Permits web portal access)*
2. **Block Any Protocol (Logged):** `GUEST_NET` $\rightarrow$ `EMPLOYEE_NET` *(Blocks lateral movement, ICMP, SSH, and internal workstation access)*
3. **Pass Any Protocol:** `GUEST_NET` $\rightarrow$ `Any` *(Allows outbound internet access)*

* ![Firewall Aliases]<img width="1210" height="726" alt="06_Firewall_Aliases" src="https://github.com/user-attachments/assets/3b92c5ba-5154-4dfc-9e85-a5bdd9e938b5" />

* ![Guest Firewall Rules]<img width="1211" height="686" alt="07_Guest_Firewall_Rules" src="https://github.com/user-attachments/assets/55dfbff4-cd19-4a3d-9890-3f9253ab6839" />


---

## ✅ Policy Verification & Security Log Analysis
Tested traffic enforcement from the Kali Linux Guest client (`192.168.20.101`):
* **Gateway Reachability (`192.168.20.1`):** Passed (`0% packet loss`)
* **Internet Access (`google.com`):** Passed
* **Allowed HTTP (`http://192.168.10.10:80`):** Passed (Apache2 page loaded)
* **Unauthorized ICMP (`ping 192.168.10.10`):** Blocked (`100% packet loss`) and logged under **Status $\rightarrow$ System Logs $\rightarrow$ Firewall**.

* ![Firewall Block Logs]<img width="1228" height="193" alt="09_Firewall_Block_Log" src="https://github.com/user-attachments/assets/61c32a68-38ff-402c-81a5-0281fd1b9458" />


---

## 🧠 Key Security Takeaway
pfSense evaluates firewall rules strictly from **top to bottom** using a *first-match-wins* model. Placing the granular HTTP (`Port 80`) Pass rule above the broad `EMPLOYEE_NET` Block rule ensures legitimate web requests succeed while all other unauthorized cross-subnet traffic is dropped and logged.
