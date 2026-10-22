# [Switch Configuration With Console/SSH Connection]

## Project Overview
The goal of this Packet Tracer lab is to simulate a console connection to a switch and configure basic switch settings. I learned how to configure hostname, IP addresses, MOTD and passwords. 
I also used show commands to verify switch configurations.

* **Objective:** Simulate a console connection to a switch using Packet Tracer.
* **Core Technologies:** Secureshell (SSH)


## Network Topology
<img width="397" height="321" alt="Screenshot 2026-10-21 175144" src="https://github.com/user-attachments/assets/0a5ec97b-57d6-4ed0-9f62-6cd9b4d52dd9" />




##  Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| SW1 | FastEthernet 0/1 | 192.168.1.5 | 255.255.255.0 | 192.168.10.1 | Management |
| PC1 | FastEthernet0 | 192.168.1.3| 255.255.255.0 | 192.168.10.1 | End Host |

### Credentials used in this lab

| Purpose | Username | Password | Notes |
|---------|----------|----------|-------|
| Privileged EXEC | n/a | `homelab` | `enable secret` (encrypted) |
| Console line | n/a | `cisco` | Set on `line con 0` |
| SSH login | `admin` | `cisco` | Local user, privilege 15 |



### Terminal settings

| Setting | Value |
|---------|-------|
| Baud rate | 9600 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |

---

## Procedure

### Build the topology and connect to the console

1. **Create the topology.**
2. **Connect the PC to the switch.**
3. **Open the switch CLI.**
 


### 4.2 Inspect the default configuration

Examine the default switch settings: current configuration, IOS information, interface properties, VLAN information, and flash memory.

Enter Privileged EXEC mode first. By default the switch starts in User EXEC mode with limited access.

```
Switch> enable
```

| Step | Purpose | Command |
|------|---------|---------|
| 1 | Examine the current running configuration | `Switch# show running-config` |
| 2 | Examine the startup configuration in NVRAM | `Switch# show startup-config` |
| 3 | Examine the Cisco IOS version information | `Switch# show version` |
| 4 | Examine the default VLAN settings | `Switch# show vlan` |
| 5 | Examine flash memory | `Switch# show flash` |

### 4.3 Configure basic device settings

Configure the hostname, local passwords, and MOTD banner.

| Step | Task | Command | Notes |
|------|------|---------|-------|
| 1 | Set hostname | `Switch(config)# hostname Switch01` | |
| 2 | Encrypt plaintext passwords | `Switch01(config)# service password-encryption` | |
| 3 | Set privileged EXEC password | `Switch01(config)# enable secret class` | Stored encrypted; adds a layer of security |
| 4 | Disable DNS lookups | `Switch01(config)# no ip domain-lookup` | Prevents unwanted DNS lookups on mistyped commands |

#### Configure the MOTD banner

```
Switch01(config)# banner motd #
Enter TEXT message.  End with the character '#'.
Unauthorized access is strictly prohibited. #
```

#### Secure the console port

By default, the console allows all connections with no password. Require a password, and use `logging synchronous` so console messages do not interrupt commands you are typing.

```
Switch01(config)# line con 0
Switch01(config-line)# password cisco
Switch01(config-line)# login
Switch01(config-line)# logging synchronous
Switch01(config-line)# exit
```

### 4.4 Configure management addressing

#### Assign an IP address to VLAN 1

VLAN 1 is the default VLAN on most switches. Assigning it an IP address allows remote management of the switch. `no shutdown` activates the interface.

```
Switch01(config)# interface vlan 1
Switch01(config-if)# ip address 192.168.1.5 255.255.255.0
Switch01(config-if)# no shutdown
```

#### Configure the default gateway

The default gateway lets the switch communicate with devices outside its local network, enabling remote management from other subnets.

```
Switch01(config)# ip default-gateway 192.168.1.1
```

### 4.5 Configure SSH remote access

SSH is the more secure option for remote management.

**Step 1. Create a local user**

```
Switch01(config)# username admin privilege 15 secret cisco
```

**Step 2. Set a domain name**

```
Switch01(config)# ip domain-name eastcharmer.com
```

**Step 3. Generate RSA keys**

```
Switch01(config)# crypto key generate rsa
The name for the keys will be: Switch01.eastcharmer.com
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
```

**Step 4. Enable SSH on the VTY lines**

```
Switch01(config)# ip ssh version 2
Switch01(config)# line vty 0 15
Switch01(config-line)# transport input ssh
Switch01(config-line)# login local
```

---

## 5. Verification

### Verify the switch configuration

```
Switch# show ip interface brief
Switch# show run
```

### Test SSH access

From the PC's Command Prompt:

```
PC> ssh -l admin 192.168.1.5
```

Enter the password when prompted. A successful login confirms secure remote access to the switch.

---

## 6. Complete Configuration

All commands from this lab in one place, in the order they are applied (from global configuration mode).

```
hostname Switch01
service password-encryption
enable secret class
no ip domain-lookup

banner motd #
Unauthorized access is strictly prohibited. #

line con 0
 password cisco
 login
 logging synchronous
 exit

interface vlan 1
 ip address 192.168.1.5 255.255.255.0
 no shutdown
 exit

ip default-gateway 192.168.1.1

username admin privilege 15 secret cisco
ip domain-name eastcharmer.com
crypto key generate rsa
 ! modulus: 1024
ip ssh version 2

line vty 0 15
 transport input ssh
 login local
