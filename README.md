# Basic Switch Configuration

A hands-on Packet Tracer lab for configuring a Cisco 2960 switch from a fresh default state. This guide walks through the essential setup tasks required to secure a switch, assign management addressing, and enable SSH remote access.

## Table of Contents

1. [Overview](#1-overview)
2. [Lab Requirements](#2-lab-requirements)
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

This lab simulates a console session to a Cisco switch and walks through the initial configuration needed to bring the device into a usable and secure state. By the end of the exercise, you will have configured a hostname, secure passwords, a login banner, a management IP address, a default gateway, and SSH access.

### Objectives

- Connect to a switch using the console port in Packet Tracer.
- Inspect the default switch configuration.
- Configure the hostname, passwords, and banner.
- Assign an IP address to VLAN 1 for management.
- Enable SSH for remote administration.
- Verify that the switch is correctly configured.

---

## 2. Lab Requirements

| Requirement | Details |
|-------------|---------|
| Devices | 1 x Cisco 2960 switch, 1 x PC |
| Cabling | 1 x console cable, 1 x straight-through Ethernet cable |
| Software | Cisco Packet Tracer |
| Knowledge | Basic familiarity with Cisco CLI navigation |

### CLI modes used in this lab

| Mode | Prompt | How to enter |
|------|--------|--------------|
| User EXEC | `Switch>` | Default login state |
| Privileged EXEC | `Switch#` | `enable` |
| Global configuration | `Switch(config)#` | `configure terminal` |
| Interface configuration | `Switch(config-if)#` | `interface <name>` |
| Line configuration | `Switch(config-line)#` | `line <type> <number>` |

> Note: The switch starts in User EXEC mode. This mode is limited and does not allow configuration changes until you enter Privileged EXEC mode with `enable`.

---

## 3. Lab Reference

### Topology

<img width="536" height="362" alt="Switch configuration topology" src="https://github.com/user-attachments/assets/26964705-13a1-4621-9b40-76ba5d413cbe" />

### Addressing

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| Switch01 | VLAN 1 (SVI) | 192.168.1.5 | 255.255.255.0 | 192.168.1.1 |

### Credentials used in this lab

| Purpose | Username | Password | Notes |
|---------|----------|----------|-------|
| Privileged EXEC | — | `class` | `enable secret` (encrypted) |
| Console line | — | `cisco` | Configured on `line con 0` |
| SSH login | `admin` | `cisco` | Local user with privilege level 15 |

### Terminal settings

| Setting | Value |
|---------|-------|
| Baud rate | 9600 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |

---

## 4. Procedure

### 4.1 Build the topology and connect to the console

Follow these steps to physically connect the devices and access the switch CLI.

1. Create the topology with the Cisco switch and a PC.
2. Connect the PC to the switch using the console cable.
   1. Select the **Console Cable** from the cable options.
   2. Click the PC and connect the cable to the **RS-232** port.
   3. Click the switch and connect the other end to the **Console** port.
3. Open the switch CLI.
   1. Click the PC and open the **Desktop** tab.
   2. Launch the **Terminal** application.
   3. Keep the default terminal settings shown in [Terminal settings](#terminal-settings).

Expected result: You are connected to the switch and can begin entering Cisco IOS commands.

---

### 4.2 Inspect the default configuration

Before making any changes, review the switch’s current state to understand what the default configuration contains.

Enter Privileged EXEC mode first:

```text
Switch> enable
```

| Step | Purpose | Command |
|------|---------|---------|
| 1 | View the active configuration | `Switch# show running-config` |
| 2 | View the saved startup configuration | `Switch# show startup-config` |
| 3 | Review the IOS version and system details | `Switch# show version` |
| 4 | View VLAN information | `Switch# show vlan` |
| 5 | Inspect flash memory | `Switch# show flash` |

These commands help confirm the device is in a default state and provide useful context before you change settings.

---

### 4.3 Configure basic device settings

In this section, you will configure the hostname, secure the console, disable DNS lookup, encrypt saved passwords, and add a login banner.

| Step | Task | Command | Notes |
|------|------|---------|-------|
| 1 | Set the hostname | `Switch(config)# hostname Switch01` | Device name becomes `Switch01` |
| 2 | Encrypt plaintext passwords | `Switch01(config)# service password-encryption` | Protects saved passwords |
| 3 | Set the privileged EXEC password | `Switch01(config)# enable secret class` | Stores the password in encrypted form |
| 4 | Disable DNS lookups | `Switch01(config)# no ip domain-lookup` | Prevents slow or unwanted DNS queries |

#### Configure the MOTD banner

The message of the day (MOTD) banner appears when users access the device and helps warn unauthorized users.

```text
Switch01(config)# banner motd #
Enter TEXT message.  End with the character '#'.
Unauthorized access is strictly prohibited. #
```

#### Secure the console port

By default, the console line allows access without a password. To secure it, configure a password and enable synchronous logging so console messages do not interrupt your typed commands.

```text
Switch01(config)# line con 0
Switch01(config-line)# password cisco
Switch01(config-line)# login
Switch01(config-line)# logging synchronous
Switch01(config-line)# exit
```

---

### 4.4 Configure management addressing

To manage the switch over the network, assign it a management IP address and default gateway.

#### Assign an IP address to VLAN 1

VLAN 1 is the default VLAN on most Cisco switches. Assigning it a management IP address allows access from the local network.

```text
Switch01(config)# interface vlan 1
Switch01(config-if)# ip address 192.168.1.5 255.255.255.0
Switch01(config-if)# no shutdown
```

#### Configure the default gateway

The default gateway allows the switch to send traffic outside its local subnet, which is necessary for remote management.

```text
Switch01(config)# ip default-gateway 192.168.1.1
```

---

### 4.5 Configure SSH remote access

SSH is the preferred method for remote access because it provides encrypted communication.

#### Step 1: Create a local user

```text
Switch01(config)# username admin privilege 15 secret cisco
```

#### Step 2: Set a domain name

```text
Switch01(config)# ip domain-name rockinprotein.com
```

#### Step 3: Generate RSA keys

```text
Switch01(config)# crypto key generate rsa
The name for the keys will be: Switch01.rockinprotein.com
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
```

#### Step 4: Enable SSH on the VTY lines

```text
Switch01(config)# ip ssh version 2
Switch01(config)# line vty 0 15
Switch01(config-line)# transport input ssh
Switch01(config-line)# login local
```

This configures the switch to accept SSH connections only and authenticate users against the local database.

---

## 5. Verification

After completing the configuration, verify that the switch is reachable and that SSH is working correctly.

### Verify the switch configuration

```text
Switch# show ip interface brief
Switch# show run
```

Review the output to confirm:

- VLAN 1 has an assigned IP address.
- The default gateway is configured.
- The hostname is correct.
- The console and SSH settings are present.

### Test SSH access

From the PC command prompt:

```text
PC> ssh -l admin 192.168.1.5
```

When prompted, enter the password `cisco`.

A successful login confirms that SSH is enabled and operating correctly.

---

## 6. Complete Configuration

The commands below are the full configuration in the order they are typically applied from Global Configuration mode.

```text
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
| `show startup-config` | Privileged EXEC | Display the saved startup configuration |
| `show version` | Privileged EXEC | Display IOS version information |
| `show vlan` | Privileged EXEC | Display VLAN settings |
| `show flash` | Privileged EXEC | Display flash memory contents |
| `show ip interface brief` | Privileged EXEC | Summarize current interface IP status |
| `hostname <name>` | Global config | Set the device hostname |
| `service password-encryption` | Global config | Encrypt plaintext passwords in the saved configuration |
| `enable secret <pw>` | Global config | Configure the encrypted privileged EXEC password |
| `no ip domain-lookup` | Global config | Disable DNS lookups |
| `banner motd #<text>#` | Global config | Set the message of the day banner |
| `line con 0` | Global config | Enter the console line configuration |
| `logging synchronous` | Line config | Prevent console messages from interrupting command input |
| `interface vlan 1` | Global config | Enter the VLAN 1 interface configuration |
| `ip address <ip> <mask>` | Interface config | Assign an IP address to the interface |
| `no shutdown` | Interface config | Activate the interface |
| `ip default-gateway <ip>` | Global config | Configure the default gateway |
| `username <u> privilege 15 secret <pw>` | Global config | Create a local admin user |
| `ip domain-name <name>` | Global config | Set the domain name |
| `crypto key generate rsa` | Global config | Generate RSA keys for SSH |
| `ip ssh version 2` | Global config | Enable SSH version 2 only |
| `line vty 0 15` | Global config | Enter the VTY line configuration |
| `transport input ssh` | Line config | Allow only SSH access on VTY lines |
| `login local` | Line config | Authenticate users against the local database |

---

This lab provides a practical introduction to basic switch hardening and remote management setup. Once complete, the switch will be secured, reachable on the network, and ready for SSH administration.
