# [Switch Configuration With Console/SSH Connection]

## Project Overview
The goal of this lab is to simulate a console connection to a switch and configure basic switch settings. I learned how to configure hostname, IP addresses, MOTD and passwords. 
I also used show commands to verify switch configurations.

* **Objective:** Simulate a console connection to a switch using Packet Tracer.
* **Core Technologies:** Secureshell (SSH)


## Network Topology
<img width="397" height="321" alt="Screenshot 2026-10-21 175144" src="https://github.com/user-attachments/assets/0a5ec97b-57d6-4ed0-9f62-6cd9b4d52dd9" />



---

## 🔢 Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| SW1 | FastEthernet 0/1 | 192.168.1.5 | 255.255.255.0 | 192.168.10.1 | Management |
| PC1 | FastEthernet0 | 192.168.1.3| 255.255.255.0 | 192.168.10.1 | End Host |

---

## ⚙️ Configuration Highlights
Explain the key technical implementations. Use syntax-highlighted code blocks for crucial Cisco IOS commands.

### 1. Inter-VLAN Routing & Sub-interfaces
```cisco
HQ-Router(config)# interface gigabitEthernet 0/0.10
HQ-Router(config-subif)# encapsulation dot1Q 10
HQ-Router(config-subif)# ip address 192.168.10.1 255.255.255.0
```

### 2. OSPF Routing Protocol
```cisco
HQ-Router(config)# router ospf 1
HQ-Router(config-router)# network 192.168.10.0 0.0.0.255 area 0
HQ-Router(config-router)# network 10.0.0.0 0.0.0.3 area 0
```

---

## 🧪 Verification & Testing
Prove that your network functions as designed. Show the commands used and paste or screenshot the output.

* **End-to-End Connectivity:** PC-01 successfully pings the Branch Server (`ping 172.16.1.100`).
* **Routing Table Verification:** `show ip route` demonstrates all expected OSPF routes (`O`).
* **Security Check:** `show ip access-lists` confirms unauthorized traffic is blocked.

```text
HQ-Router# show ip route ospf
     10.0.0.0/30 is subnetted, 1 subnets
O       10.0.0.4 [110/2] via 10.0.0.2, 00:14:22, GigabitEthernet0/1
O IA 172.16.1.0/24 [110/3] via 10.0.0.2, 00:12:05, GigabitEthernet0/1
```

---
