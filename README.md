# Basic Switch Configuration


## Table of Contents

1. [Overview](#1-overview)
2. [Devices/Hardware](#2-Devices/Hardware)
3. [Lab Reference](#3-lab-reference)
4. [Procedure](#4-procedure)
   - [4.1 Build the topology and connect to the console](#41-build-the-topology-and-connect-to-the-console)
   - [4.2 Inspect the default configuration](#42-inspect-the-default-configuration)
   - [4.3 Configure basic device settings](#43-configure-basic-device-settings)
   - [4.4 Configure management addressing](#44-configure-management-addressing)
   - [4.5 Configure SSH remote access](#45-configure-ssh-remote-access)
5. [Verification](#5-verification)
6. [Complete Configuration](#6-complete-configuration)
7. [Command Quick Reference](#7-command-quick-reference)

---

## 1. Overview

This Packet Tracer Lab simulates a console connection to a switch and walks through its initial configuration: hostname, passwords, login banner, management IP address, default gateway, and secure remote access over SSH.

### Objectives

- Simulate a console connection to a switch using Packet Tracer.
- Understand and configure basic switch settings.
- Configure hostname, IP addresses, MOTD and passwords.
- Use `show` commands to verify switch configurations.

---

## 2. Devices/Hardware

| Requirement | Detail |
|-------------|--------|
| Devices | 1 × Cisco 2960 switch, 1 × PC |
| Cabling | 1 × console cable/ 1 x straight through cable |
| Knowledge | Basic navigation of the Cisco CLI (see [CLI modes](#cli-modes-used-in-this-lab)) |

### CLI modes used in this lab

| Mode | Prompt | How to enter |
|------|--------|--------------|
| User EXEC | `Switch>` | Default on login (limited access) |
| Privileged EXEC | `Switch#` | `enable` |
| Global configuration | `Switch(config)#` | `configure terminal` |
| Interface configuration | `Switch(config-if)#` | `interface <name>` |
| Line configuration | `Switch(config-line)#` | `line <type> <number>` |

---

## 3. Lab Reference

### Topology

<img width="536" height="362" alt="Switch Configuration" src="https://github.com/user-attachments/assets/26964705-13a1-4621-9b40-76ba5d413cbe" />


### Addressing

| Device | Interface | IP address | Subnet mask | Default gateway |
|--------|-----------|------------|-------------|-----------------|
| Switch01 | VLAN 1 (SVI) | 192.168.1.5 | 255.255.255.0 | 192.168.1.1 |

### Credentials used in this lab

| Purpose | Username | Password | Notes |
|---------|----------|----------|-------|
| Privileged EXEC | n/a | `class` | `enable secret` (encrypted) |
| Console line | n/a | `cisco` | Set on `line con 0` |
| SSH login | `admin` | `cisco` | Local user, privilege 15 |



### Terminal settings

| Setting | Value |
|---------|-------|
| Baud rate | 9600 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |



## 4. Procedure

### Build the topology and connect to the console

1. **Create the topology.**
2. **Connect the PC to the switch.**
   1. Select the **Console Cable** from the cable options.
   2. Click the PC and connect the cable to the **RS-232** port (the console port).
   3. Click the switch and connect the other end to its **Console** port.
3. **Open the switch CLI.**
   1. Click the PC and open the **Desktop** tab.
   2. Launch the **Terminal** application.
   3. Keep the default terminal settings (see [Terminal settings](#terminal-settings)).

**Expected result:** You have access to the switch's CLI.

### Inspect the default configuration

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
Switch01(config)# ip domain-name rockinprotein.com
```

**Step 3. Generate RSA keys**

```
Switch01(config)# crypto key generate rsa
The name for the keys will be: Switch01.rockinprotein.com
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
ip domain-name rockinprotein.com
crypto key generate rsa
 ! modulus: 1024
ip ssh version 2

line vty 0 15
 transport input ssh
 login local
```

---

## 7. Command Quick Reference

| Command | Mode | Description |
|---------|------|-------------|
| `enable` | User EXEC | Enter Privileged EXEC mode |
| `show running-config` | Privileged EXEC | Display the active configuration |
| `show startup-config` | Privileged EXEC | Display the configuration stored in NVRAM |
| `show version` | Privileged EXEC | Display IOS version information |
| `show vlan` | Privileged EXEC | Display VLAN settings |
| `show flash` | Privileged EXEC | Display flash memory contents |
| `show ip interface brief` | Privileged EXEC | Summarize interface IP status |
| `hostname <name>` | Global config | Set the device hostname |
| `service password-encryption` | Global config | Encrypt plaintext passwords in the config |
| `enable secret <pw>` | Global config | Set the encrypted Privileged EXEC password |
| `no ip domain-lookup` | Global config | Disable DNS lookups |
| `banner motd #<text>#` | Global config | Set the message of the day banner |
| `line con 0` | Global config | Enter console line configuration |
| `logging synchronous` | Line config | Keep console messages from interrupting input |
| `interface vlan 1` | Global config | Enter VLAN 1 interface configuration |
| `ip address <ip> <mask>` | Interface config | Assign an IP address |
| `no shutdown` | Interface config | Activate the interface |
| `ip default-gateway <ip>` | Global config | Set the default gateway |
| `username <u> privilege 15 secret <pw>` | Global config | Create a local admin user |
| `ip domain-name <name>` | Global config | Set the domain name |
| `crypto key generate rsa` | Global config | Generate RSA keys for SSH |
| `ip ssh version 2` | Global config | Require SSH version 2 |
| `line vty 0 15` | Global config | Enter VTY line configuration |
| `transport input ssh` | Line config | Allow only SSH on the VTY lines |
| `login local` | Line config | Authenticate against the local user database |
