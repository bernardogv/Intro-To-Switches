# Chapter 4: Basic Switch Configuration

## Overview: Getting Started with Switch Management

This chapter covers the essential skills needed to configure and manage network switches. While we'll focus primarily on Cisco IOS (the most common CLI), the concepts apply broadly across vendors with syntax variations.

**You'll learn:**
- Physical console access methods
- CLI navigation and command modes
- Initial setup procedures
- Basic security configuration
- Port configuration fundamentals
- Configuration persistence

**Prerequisites:**
- Access to a managed switch (physical or virtual)
- Console cable (USB-to-RJ45 or USB-to-Serial)
- Terminal emulator software (PuTTY, SecureCRT, screen, etc.)

## Console Access and Initial Setup

### Physical Console Connections

Most switches provide a **console port** for out-of-band management—a dedicated port for initial configuration when network access isn't available.

#### Console Port Types

**1. RJ45 Console Port (Most Common)**
```
Switch                          Computer
┌──────────┐                  ┌──────────┐
│          │                  │          │
│   [RJ45] │◄─Rollover Cable─►│ [USB]    │
│  Console │                  │  (USB-to-│
│   Port   │                  │  Serial) │
└──────────┘                  └──────────┘

Cable pinout (RJ45 Rollover):
Pin 1 ──────────────────► Pin 8
Pin 2 ──────────────────► Pin 7
Pin 3 ──────────────────► Pin 6
Pin 4 ──────────────────► Pin 5
```

**Modern approach**: USB-to-RJ45 console cable
- Drivers: FTDI or Prolific chipset
- Single cable solution (USB directly to switch RJ45 console)
- Example: Cisco USB Console Cable

**Traditional approach**: Rollover cable + USB-to-Serial adapter
- Rollover (RJ45 to DB9)
- DB9-to-USB adapter
- Two-piece solution (common for older equipment)

**2. Mini-USB Console Port (Newer Switches)**
```
Switch                    Computer
┌──────────┐            ┌──────────┐
│          │            │          │
│ [Mini-USB│◄─USB Cable►│ [USB]    │
│  Console]│            │          │
└──────────┘            └──────────┘
```
- Example: Cisco Catalyst 9000 series
- Standard Mini-USB to USB-A cable
- Drivers often built into modern OSes

**3. Micro-USB Console Port (Some SMB Switches)**
- Example: Some Netgear, TP-Link models
- Standard Micro-USB cable

#### Serial Port Settings (Critical!)

Regardless of cable type, these settings must match:

```
┌──────────────────────────────────┐
│ Console Port Settings            │
├──────────────────────────────────┤
│ Baud Rate:     9600 (default)    │
│ Data Bits:     8                 │
│ Parity:        None              │
│ Stop Bits:     1                 │
│ Flow Control:  None              │
├──────────────────────────────────┤
│ Shorthand: 9600 8-N-1            │
└──────────────────────────────────┘
```

**Some switches use 115200 baud** (check documentation)—if you see garbled text, try this baud rate.

### Terminal Emulator Software

#### Windows: PuTTY (Free)

**Setup steps:**
1. Download PuTTY from https://www.putty.org
2. Run PuTTY
3. Select "Serial" connection type
4. Set serial line (e.g., COM3—check Device Manager)
5. Set speed to 9600
6. Click "Open"

**Finding COM port:**
```
1. Open Device Manager (devmgmt.msc)
2. Expand "Ports (COM & LPT)"
3. Look for "USB Serial Port (COM3)" or similar
4. Note the COM number
```

**PuTTY configuration:**
```
Connection Type: [Serial]
Serial line: COM3
Speed: 9600

Category → Serial:
- Speed (baud): 9600
- Data bits: 8
- Stop bits: 1
- Parity: None
- Flow control: None
```

#### macOS/Linux: screen or minicom

**Using screen (built-in):**
```bash
# Find the device
ls /dev/tty.*
# Output: /dev/tty.usbserial-1234

# Connect
screen /dev/tty.usbserial-1234 9600

# Disconnect
# Press: Ctrl+A, then K, then Y
```

**Using minicom:**
```bash
# Install
brew install minicom  # macOS
sudo apt install minicom  # Ubuntu/Debian

# Configure
minicom -s
# Set Serial Device: /dev/tty.usbserial-1234
# Set Bps/Par/Bits: 9600 8N1
# Save setup as default

# Connect
minicom
```

#### Troubleshooting Console Connections

**No output when connected:**
1. Press Enter several times (wake up switch)
2. Verify baud rate (try 115200 if 9600 fails)
3. Check cable (try different USB port)
4. Verify driver installation
5. Check COM port in Device Manager
6. Try different terminal emulator

**Garbled text:**
- Wrong baud rate (switch between 9600/115200)
- Incorrect data bits/parity/stop bits
- Bad cable or loose connection

**Connection drops:**
- USB power management (disable in Windows)
- Bad USB cable
- Insufficient power from USB port (try powered hub)

### Web GUI Access (Alternative)

Many managed switches offer web interfaces for initial setup:

**Access steps:**
1. Connect switch to your computer via Ethernet
2. Set your computer's IP to same subnet as switch default IP
   - Example: Switch default 192.168.1.1
   - Set computer to 192.168.1.100/24
3. Open browser: http://192.168.1.1
4. Default credentials (check manual):
   - Cisco: admin / cisco
   - Netgear: admin / password
   - TP-Link: admin / admin
   - **CHANGE IMMEDIATELY AFTER LOGIN**

**Web GUI limitations:**
- Slower than CLI for bulk configuration
- Not all features exposed
- Automation difficult
- Can't be used if IP config is wrong

**Best practice**: Use web GUI for initial IP setup, then switch to CLI for ongoing management.

## CLI Basics: Cisco IOS Command Structure

### Command Modes and Navigation

Cisco IOS has hierarchical command modes, each with different privileges and available commands:

```
┌────────────────────────────────────────────────────┐
│                User EXEC Mode                      │
│  Prompt: Switch>                                   │
│  Purpose: Basic monitoring, limited show commands  │
│  Access: Default (no password)                     │
├────────────────────────────────────────────────────┤
│  Command to enter: [Automatic on login]            │
└────────────────────────────────────────────────────┘
                      │
                enable │ (privileged mode)
                      ▼
┌────────────────────────────────────────────────────┐
│           Privileged EXEC Mode                     │
│  Prompt: Switch#                                   │
│  Purpose: View full config, debugging, admin tasks │
│  Access: Requires enable password                  │
├────────────────────────────────────────────────────┤
│  Command to enter: Switch> enable                  │
│  Command to exit: Switch# disable                  │
└────────────────────────────────────────────────────┘
                      │
        configure terminal │ (global config)
                      ▼
┌────────────────────────────────────────────────────┐
│          Global Configuration Mode                 │
│  Prompt: Switch(config)#                          │
│  Purpose: System-wide configuration               │
│  Access: From privileged EXEC                     │
├────────────────────────────────────────────────────┤
│  Command to enter: Switch# configure terminal      │
│  Command to exit: Switch(config)# exit            │
└────────────────────────────────────────────────────┘
                      │
        interface <name> │ (interface config)
                      ▼
┌────────────────────────────────────────────────────┐
│         Interface Configuration Mode               │
│  Prompt: Switch(config-if)#                       │
│  Purpose: Configure specific interface            │
│  Access: From global config                       │
├────────────────────────────────────────────────────┤
│  Command to enter:                                │
│    Switch(config)# interface gigabitethernet 0/1  │
│  Command to exit: Switch(config-if)# exit         │
└────────────────────────────────────────────────────┘
```

**Mode summary:**
```
Switch>                         User EXEC
Switch> enable
Switch#                         Privileged EXEC
Switch# configure terminal
Switch(config)#                 Global Configuration
Switch(config)# interface gi0/1
Switch(config-if)#             Interface Configuration
Switch(config-if)# exit
Switch(config)# line vty 0 15
Switch(config-line)#           Line Configuration
Switch(config-line)# exit
Switch(config)# exit
Switch#
```

### Essential CLI Commands

#### Help System

**Context-sensitive help:**
```
Switch# ?
  clear      Reset functions
  clock      Manage system clock
  configure  Enter configuration mode
  copy       Copy files
  debug      Debugging functions
  ...

Switch# show ?
  arp             ARP table
  interfaces      Interface status
  ip              IP information
  mac             MAC address table
  running-config  Current configuration
  version         System hardware/software
  ...

Switch# show ip ?
  interface  IP interface status
  route      IP routing table
  ...
```

**Command completion:**
```
Switch# conf?
configure

Switch# sh int?
interface

Switch# show int gi?
gigabitethernet
```

**Partial commands** (if unambiguous):
```
Switch# show run       (show running-config)
Switch# conf t         (configure terminal)
Switch# int gi0/1      (interface gigabitethernet 0/1)
```

#### Navigation Shortcuts

**Keyboard shortcuts:**
```
Ctrl+A          Move cursor to beginning of line
Ctrl+E          Move cursor to end of line
Ctrl+W          Erase word to left
Ctrl+U          Erase line
Ctrl+Z          Exit to privileged EXEC from any config mode
Tab             Complete partial command
?               Context-sensitive help
Up Arrow        Previous command (command history)
Down Arrow      Next command (command history)
```

**Exit methods:**
```
exit            Exit one level
end             Exit to privileged EXEC (same as Ctrl+Z)
Ctrl+C          Abort command entry
Ctrl+Shift+6    Interrupt running process (ping, traceroute)
```

#### Viewing Configurations

**Two configurations exist:**

**1. Running Configuration (RAM)**
```
Switch# show running-config
! Active configuration (in RAM)
! Changes here take effect immediately
! Lost on reboot unless saved

version 15.2
hostname MySwitch
!
interface GigabitEthernet0/1
 description Server Room Port
 switchport mode access
 switchport access vlan 10
!
end
```

**2. Startup Configuration (NVRAM)**
```
Switch# show startup-config
! Saved configuration (in NVRAM)
! Loaded on boot
! Must be manually saved from running-config

! If you see:
! startup-config is not present
! → Configuration has never been saved
```

**Configuration relationship:**
```
                  Power On
                      │
                      ▼
          Load startup-config (NVRAM)
                      │
                      ▼
            running-config (RAM)
                      │
            User makes changes
                      │
         (changes in running-config only)
                      │
                      ▼
         User issues: copy run start
                      │
                      ▼
          Save to startup-config
```

## Hostname, Passwords, and Basic Security

### Setting a Hostname

**Purpose**: Identify switch in prompts and network diagrams

```
Switch# configure terminal
Switch(config)# hostname Core-Switch-1
Core-Switch-1(config)#
```

**Naming conventions:**
```
Good examples:
- Core-Switch-1 (clear role and number)
- Building-A-IDF-2 (location-based)
- SW-DATACENTER-01 (prefix + location + number)

Bad examples:
- Switch (too generic)
- Bob (unclear purpose)
- asdf1234 (meaningless)
```

**Best practices:**
- Use descriptive names (role, location, or both)
- Include numbering for multiple switches
- Avoid special characters (stick to letters, numbers, hyphens)
- Keep under 20 characters for readability

### Securing Access: Passwords and Encryption

#### Enable Password (Privileged EXEC Access)

**Enable secret (encrypted):**
```
Switch(config)# enable secret Str0ngP@ssw0rd
! Uses MD5 hashing (more secure than "enable password")
```

**Enable password (plaintext - deprecated):**
```
Switch(config)# enable password WeakPassword
! Stored in plaintext - DO NOT USE
! Only for legacy compatibility
```

**Best practice**: Always use `enable secret`, never `enable password`.

#### Console Password (Console Port Access)

```
Switch(config)# line console 0
Switch(config-line)# password Cons0leP@ss
Switch(config-line)# login
Switch(config-line)# logging synchronous
Switch(config-line)# exec-timeout 15 0
Switch(config-line)# exit
```

**Explanation:**
- `line console 0`: Enter console line configuration
- `password Cons0leP@ss`: Set password
- `login`: Require password for console access
- `logging synchronous`: Prevent log messages from interrupting typing
- `exec-timeout 15 0`: Logout after 15 minutes idle (0 = no seconds)
- `exec-timeout 0 0`: Never timeout (not recommended—security risk)

#### VTY Password (Telnet/SSH Access)

**VTY lines** handle remote access (Telnet and SSH):

```
Switch(config)# line vty 0 15
Switch(config-line)# password VtyP@ssw0rd
Switch(config-line)# login
Switch(config-line)# transport input ssh
Switch(config-line)# exec-timeout 10 0
Switch(config-line)# exit
```

**Explanation:**
- `line vty 0 15`: Configure all 16 VTY lines (0-15)
- `transport input ssh`: Only allow SSH (disable Telnet for security)
- `transport input telnet ssh`: Allow both (not recommended)
- `transport input none`: Disable remote access

#### Password Encryption

**Problem**: Passwords stored in plaintext in running-config:

```
Switch# show running-config
!
line vty 0 15
 password VtyP@ssw0rd
!
! ← Password visible to anyone with "show run" access
```

**Solution**: Enable service password-encryption

```
Switch(config)# service password-encryption
```

**Result:**
```
Switch# show running-config
!
line vty 0 15
 password 7 082A4D5E0A1C1B
!
! ← Password now encrypted (Type 7 - weak, but better than plaintext)
```

**Encryption types in Cisco IOS:**
- **Type 0**: Plaintext (no encryption)
- **Type 7**: Cisco proprietary reversible encryption (weak—can be decrypted online)
- **Type 5**: MD5 hash (enable secret—not reversible)
- **Type 8**: PBKDF2-SHA256 (newer, stronger)
- **Type 9**: Scrypt (strongest, modern IOS)

**Best practice**:
```
! Always enable password encryption
Switch(config)# service password-encryption

! Use enable secret (Type 5/8/9)
Switch(config)# enable algorithm-type scrypt secret MySecureP@ss

! For VTY, use local user database with Type 8/9
Switch(config)# username admin algorithm-type scrypt secret AdminP@ss
Switch(config)# line vty 0 15
Switch(config-line)# login local
```

### SSH Configuration (Recommended over Telnet)

**Why SSH?**
- Encrypted communication (Telnet is plaintext)
- Authentication
- Secure remote management

**Requirements:**
- IOS image with cryptographic features (k9)
- Hostname and domain name configured
- RSA keys generated

**SSH setup:**
```
! 1. Set hostname and domain
Switch(config)# hostname MySwitch
MySwitch(config)# ip domain-name example.com

! 2. Generate RSA keys
MySwitch(config)# crypto key generate rsa
How many bits in the modulus [512]: 2048
! (2048-bit keys recommended for security)

! 3. Configure SSH version 2 (more secure)
MySwitch(config)# ip ssh version 2

! 4. Create user account
MySwitch(config)# username admin privilege 15 secret AdminP@ss

! 5. Configure VTY lines for SSH with local authentication
MySwitch(config)# line vty 0 15
MySwitch(config-line)# transport input ssh
MySwitch(config-line)# login local
MySwitch(config-line)# exit

! 6. (Optional) Set SSH timeout and authentication retries
MySwitch(config)# ip ssh time-out 60
MySwitch(config)# ip ssh authentication-retries 3
```

**Verification:**
```
MySwitch# show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 60 secs; Authentication retries: 3

MySwitch# show ssh
Connection  Version  Mode  Encryption  Hmac         State
0           2.0      IN    aes128-cbc  hmac-sha1    Session started
```

**Connecting via SSH:**
```bash
# From another device:
ssh admin@192.168.1.1

# Specify port if non-standard:
ssh -p 2222 admin@192.168.1.1
```

## IP Addressing for Management

### Management VLAN Concept

Switches need IP addresses for remote management (SSH, SNMP, web GUI). This is configured on a **management VLAN** interface (typically VLAN 1 by default, but best practice is to use a dedicated VLAN).

```
Physical Topology:
PC (192.168.10.50) ──► Port 1 ──┐
                                 ├──► Switch
Remote Admin ─────► SSH ─────────┘

Logical Configuration:
VLAN 1 (Management): 192.168.10.1/24
PC communicates with switch via VLAN 1 interface
```

### Configuring Management IP Address

**Default VLAN (VLAN 1):**
```
Switch# configure terminal
Switch(config)# interface vlan 1
Switch(config-if)# ip address 192.168.10.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit
Switch(config)# ip default-gateway 192.168.10.254
```

**Explanation:**
- `interface vlan 1`: Enter VLAN 1 interface (Layer 3 virtual interface)
- `ip address 192.168.10.1 255.255.255.0`: Assign IP and subnet mask
- `no shutdown`: Enable the interface (interfaces are shutdown by default)
- `ip default-gateway`: Set gateway for management traffic (reaching other subnets)

**Dedicated Management VLAN (Best Practice):**
```
! 1. Create management VLAN
Switch(config)# vlan 99
Switch(config-vlan)# name Management
Switch(config-vlan)# exit

! 2. Assign IP to VLAN 99 interface
Switch(config)# interface vlan 99
Switch(config-if)# ip address 192.168.99.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

! 3. Set default gateway
Switch(config)# ip default-gateway 192.168.99.254

! 4. Assign management port to VLAN 99
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 99
Switch(config-if)# exit
```

**Why separate management VLAN?**
- **Security**: Isolate management from user traffic
- **Performance**: Management traffic doesn't compete with data
- **Best practice**: VLAN 1 is often left as default; dedicated VLAN shows intentional design

### Verification

```
Switch# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
Vlan1                  unassigned      YES unset  administratively down down
Vlan99                 192.168.99.1    YES manual up                    up
GigabitEthernet0/1     unassigned      YES unset  up                    up
...

Switch# show interfaces vlan 99
Vlan99 is up, line protocol is up
  Hardware is Ethernet SVI, address is 001a.2b3c.4d5e
  Internet address is 192.168.99.1/24
  MTU 1500 bytes, BW 1000000 Kbit/sec
  ...
```

**Ping test:**
```
Switch# ping 192.168.99.254
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.99.254, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
```

## Port Configuration Fundamentals

### Interface Naming Conventions

**Cisco naming:**
```
FastEthernet0/1      → FE 0/1    (100 Mbps)
GigabitEthernet0/1   → Gi 0/1    (1 Gbps)
TenGigabitEthernet0/1→ Te 0/1    (10 Gbps)
TwentyFiveGigE0/1    → Twe 0/1   (25 Gbps)

Format: <Type><Module>/<Port>
- Module: 0 for fixed switches, 1-N for chassis/stacks
- Port: 1-48 (or more)

Examples:
- GigabitEthernet0/1  (Module 0, Port 1)
- GigabitEthernet1/5  (Module 1, Port 5 - stacked/chassis)
```

**Range configuration:**
```
! Configure multiple ports at once
Switch(config)# interface range gigabitethernet 0/1-24
Switch(config-if-range)# description Access Ports
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# exit
```

### Basic Port Commands

#### Port Description

**Purpose**: Document port usage for troubleshooting and auditing

```
Switch(config)# interface gigabitethernet 0/5
Switch(config-if)# description File Server - Primary NIC
Switch(config-if)# exit

Switch(config)# interface gigabitethernet 0/12
Switch(config-if)# description 3rd Floor - Room 301 - Workstation
Switch(config-if)# exit
```

**Viewing descriptions:**
```
Switch# show interfaces description
Interface      Status         Protocol  Description
Gi0/1          up             up        Uplink to Distribution Switch
Gi0/5          up             up        File Server - Primary NIC
Gi0/12         up             up        3rd Floor - Room 301 - Workstation
Gi0/24         down           down      UNUSED
```

**Best practices:**
- Always add descriptions
- Include: device type, location, purpose
- Use consistent format across organization
- Update when devices change

#### Port Speed and Duplex

**Auto-negotiation (default and recommended):**
```
Switch(config-if)# speed auto
Switch(config-if)# duplex auto
```

**Manual configuration (when auto-negotiation fails):**
```
Switch(config-if)# speed 100
Switch(config-if)# duplex full
```

**Speed options:**
- `auto`: Auto-negotiate (recommended)
- `10`: 10 Mbps
- `100`: 100 Mbps (Fast Ethernet)
- `1000`: 1 Gbps (Gigabit)
- `10000`: 10 Gbps

**Duplex options:**
- `auto`: Auto-negotiate
- `full`: Full-duplex (send and receive simultaneously)
- `half`: Half-duplex (legacy, collision-prone)

**Warning**: Speed and duplex must match on both ends!
```
Mismatch scenario:
Switch Port: auto/auto
Server NIC:  1000/full (manual)

Result: Duplex mismatch → poor performance, errors

Solution: Set both to auto, or both to manual with matching settings
```

#### Enable/Disable Ports

**Shutdown (disable port):**
```
Switch(config)# interface gigabitethernet 0/24
Switch(config-if)# shutdown
Switch(config-if)# exit
! Port LED goes off, interface status = down/down
```

**No shutdown (enable port):**
```
Switch(config)# interface gigabitethernet 0/24
Switch(config-if)# no shutdown
Switch(config-if)# exit
! Port LED goes on, interface status = up/up (if cable connected)
```

**Use case for shutdown:**
- Unused ports (security best practice)
- Troubleshooting (isolate problems)
- Maintenance windows

**Bulk shutdown unused ports:**
```
Switch(config)# interface range gigabitethernet 0/25-48
Switch(config-if-range)# shutdown
Switch(config-if-range)# description UNUSED - Shutdown for security
Switch(config-if-range)# exit
```

#### Viewing Port Status

**Summary view:**
```
Switch# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/1     unassigned      YES unset  up                    up
GigabitEthernet0/2     unassigned      YES unset  down                  down
...

Status column:
- up: Interface enabled, cable connected
- down: Interface enabled, no cable connected
- administratively down: Interface shutdown
```

**Detailed view:**
```
Switch# show interface gigabitethernet 0/1
GigabitEthernet0/1 is up, line protocol is up (connected)
  Hardware is Gigabit Ethernet, address is 001a.2b3c.4d5e (bia 001a.2b3c.4d5e)
  Description: Uplink to Distribution Switch
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  Full-duplex, 1000Mb/s, media type is 10/100/1000BaseTX
  ...
  5 minute input rate 12000 bits/sec, 20 packets/sec
  5 minute output rate 15000 bits/sec, 25 packets/sec
     1024567 packets input, 123456789 bytes, 0 no buffer
     ...
     987654 packets output, 98765432 bytes, 0 underruns
```

**Key fields:**
- **is up / line protocol is up**: Ideal state (physical + data link)
- **is administratively down**: Manually shutdown
- **is up / line protocol is down**: Cable issue or remote end problem
- **Full-duplex, 1000Mb/s**: Speed and duplex (verify matches expectations)
- **Input/output rates**: Traffic statistics
- **Errors**: CRC, collisions, runts (should be near zero)

**Port statistics:**
```
Switch# show interface gigabitethernet 0/1 | include error
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 output errors, 0 collisions, 0 interface resets
```

**Errors to watch:**
- **CRC errors**: Bad cable, EMI, duplex mismatch
- **Collisions**: Half-duplex or cable problem
- **Runts/Giants**: Malformed frames
- **Input errors**: Physical layer issues

## Saving Configurations

### The Critical Importance of Saving

**Reality check:**
```
Scenario: You spend 2 hours configuring a switch
         Power outage occurs
         Switch reboots

If you didn't save:
  ← All configuration LOST! ←
  Startup-config loads (old config or empty)
  You start over from scratch

If you saved:
  ← Configuration preserved ←
  Startup-config loads your work
  Switch operates normally
```

### Save Commands

**Primary method (copy running to startup):**
```
Switch# copy running-config startup-config
Destination filename [startup-config]? [Enter]
Building configuration...
[OK]
```

**Shortcut (write memory):**
```
Switch# write memory
Building configuration...
[OK]

! or even shorter:
Switch# wr
```

**Equivalent commands:**
```
Switch# copy run start         (most common)
Switch# write                  (short form)
Switch# write memory           (explicit)
Switch# wr                     (shortest)
```

**All four commands do the same thing**: Copy running-config (RAM) to startup-config (NVRAM).

### Backup to External Server (TFTP)

**Why backup externally?**
- Disaster recovery
- Configuration auditing
- Rolling back changes
- Documenting network state

**TFTP backup:**
```
! Copy running-config to TFTP server
Switch# copy running-config tftp:
Address or name of remote host []? 192.168.10.100
Destination filename [switch-confg]? backup-2025-01-15.cfg
!!
12345 bytes copied in 2.5 secs (4938 bytes/sec)

! Restore from TFTP
Switch# copy tftp: running-config
Address or name of remote host []? 192.168.10.100
Source filename []? backup-2025-01-15.cfg
Destination filename [running-config]? [Enter]
Accessing tftp://192.168.10.100/backup-2025-01-15.cfg...
Loading backup-2025-01-15.cfg from 192.168.10.100 (via Vlan1): !
[OK - 12345 bytes]
```

**Setup TFTP server:**
- **Windows**: Tftpd64, SolarWinds TFTP
- **Linux**: `sudo apt install tftpd-hpa`
- **macOS**: Built-in TFTP (`launchctl load -w /System/Library/LaunchDaemons/tftp.plist`)

**Best practice**: Schedule regular backups (weekly/monthly) and store off-site.

### Configuration File Management

**View startup vs. running config:**
```
Switch# show startup-config | include hostname
hostname OldName

Switch# show running-config | include hostname
hostname NewName

! If different, you haven't saved recent changes!
```

**Erase startup-config (reset to factory defaults):**
```
Switch# write erase
Erasing the nvram filesystem will remove all configuration files! Continue? [confirm] [Enter]
[OK]
Erase of nvram: complete

! On next reboot, switch will be blank (factory state)
! Use for decommissioning or starting fresh
```

**Reload switch:**
```
Switch# reload
System configuration has been modified. Save? [yes/no]: yes
Proceed with reload? [confirm] [Enter]

! Switch reboots, loads startup-config
```

## Complete Initial Configuration Example

Here's a step-by-step first-time setup:

```
! Connect via console cable

Switch> enable
Switch# configure terminal

! 1. Set hostname
Switch(config)# hostname Office-SW1

! 2. Secure privileged EXEC mode
Office-SW1(config)# enable secret Pr1v1l3g3dP@ss

! 3. Configure console access
Office-SW1(config)# line console 0
Office-SW1(config-line)# password C0ns0l3P@ss
Office-SW1(config-line)# login
Office-SW1(config-line)# logging synchronous
Office-SW1(config-line)# exec-timeout 15 0
Office-SW1(config-line)# exit

! 4. Set up SSH access
Office-SW1(config)# ip domain-name company.local
Office-SW1(config)# crypto key generate rsa
How many bits in the modulus [512]: 2048
Office-SW1(config)# ip ssh version 2
Office-SW1(config)# username admin privilege 15 secret AdminP@ss2025

! 5. Configure VTY lines
Office-SW1(config)# line vty 0 15
Office-SW1(config-line)# transport input ssh
Office-SW1(config-line)# login local
Office-SW1(config-line)# exec-timeout 10 0
Office-SW1(config-line)# exit

! 6. Encrypt passwords
Office-SW1(config)# service password-encryption

! 7. Set management IP (VLAN 1)
Office-SW1(config)# interface vlan 1
Office-SW1(config-if)# ip address 192.168.1.10 255.255.255.0
Office-SW1(config-if)# no shutdown
Office-SW1(config-if)# exit

! 8. Set default gateway
Office-SW1(config)# ip default-gateway 192.168.1.1

! 9. Configure banner (legal warning)
Office-SW1(config)# banner motd #
******************************************
* Unauthorized access is prohibited!    *
* All activity is monitored and logged. *
******************************************
#

! 10. Set clock (for accurate logging)
Office-SW1(config)# exit
Office-SW1# clock set 14:30:00 15 Jan 2025

! 11. SAVE CONFIGURATION
Office-SW1# copy running-config startup-config
Destination filename [startup-config]? [Enter]
Building configuration...
[OK]

! 12. Verify
Office-SW1# show running-config
! (Review configuration)

Office-SW1# show ip interface brief
! (Verify VLAN 1 is up with IP 192.168.1.10)

! 13. Test SSH access from another device
! ssh admin@192.168.1.10
```

## Key Takeaways

1. **Console access is primary**: Always configure console port first—it's your out-of-band access when network fails.

2. **Know your command modes**: User EXEC (`>`) → Privileged EXEC (`#`) → Global Config (`(config)#`) → Interface Config (`(config-if)#`).

3. **Security fundamentals**:
   - Enable secret (not enable password)
   - Console password
   - SSH-only VTY access (no Telnet)
   - Service password-encryption
   - Strong passwords (min 12 chars, mixed case, numbers, symbols)

4. **Management IP is essential**: Without it, no remote access. Configure VLAN interface + default gateway.

5. **ALWAYS SAVE YOUR WORK**: `copy run start` after every configuration session. Unsaved changes are lost on reboot.

6. **Documentation matters**: Use descriptions on every interface. Your future self will thank you.

7. **CLI shortcuts save time**: `conf t`, `int gi0/1`, `do show run`, Tab completion, `?` for help.

8. **Verification is crucial**: Use `show` commands to confirm changes took effect as expected.

## What's Next?

In Chapter 5, we'll build on these basics to explore VLANs in depth—creating, configuring, and troubleshooting virtual LANs. You'll learn how to segment networks, configure trunk ports, implement inter-VLAN routing, and leverage VLANs for security and performance.

**Practice exercises for Chapter 4:**
1. Connect to a switch via console and access all command modes
2. Configure hostname, passwords, and SSH access
3. Set up management IP and test SSH connectivity
4. Configure port descriptions and shutdown unused ports
5. Save configuration and verify with `show` commands
6. Backup configuration to TFTP server

**Common mistakes to avoid:**
- Forgetting to save configuration (causing loss on reboot)
- Setting manual speed/duplex without matching both ends
- Using Telnet instead of SSH (security risk)
- Leaving default passwords unchanged
- Not documenting configurations (descriptions, diagrams)

---

**Chapter 4 Complete** | Next: Chapter 5 - VLANs and Trunking

**Congratulations!** You now have the foundational skills to access, configure, and secure network switches. Chapters 5-10 will build advanced features on this solid base.
