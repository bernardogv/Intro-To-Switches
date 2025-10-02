# Chapter 9: Switch Security

## Introduction

Network security begins at Layer 2. While firewalls and intrusion detection systems protect at higher layers, switches form the first line of defense against many common attacks. This chapter explores essential switch security features that protect against MAC flooding, ARP poisoning, rogue DHCP servers, and unauthorized access.

Modern security threats have evolved beyond simple packet interception. Today's attackers exploit fundamental weaknesses in switched networks: the trusted nature of Layer 2, the stateless operation of many protocols, and the implicit trust between network devices. Understanding these vulnerabilities—and the countermeasures available—is crucial for building resilient networks.

---

## 9.1 Port Security: Controlling MAC Access

### The Problem: MAC Address Flooding

Switch MAC address tables have finite capacity—typically 8,000 to 128,000 entries depending on the model. Attackers exploit this limitation through MAC flooding: sending packets with thousands of random source MAC addresses, filling the CAM table until it overflows.

When the CAM table is full, switches revert to fail-open mode, flooding all frames to all ports—essentially becoming a hub. This allows attackers to capture traffic destined for other devices.

**Attack Scenario:**
```
Attacker generates packets:
  Source MAC: 00:00:00:00:00:01
  Source MAC: 00:00:00:00:00:02
  Source MAC: 00:00:00:00:00:03
  ... (continues for 10,000+ addresses)

Switch CAM table fills → Overflow → Flooding mode
Attacker receives all network traffic
```

### Port Security Fundamentals

Port security prevents MAC flooding by limiting the number of MAC addresses learned on each port. When enabled, the switch monitors source MAC addresses and takes action when violations occur.

**Three Key Parameters:**

1. **Maximum MAC addresses** - How many MACs can be learned (default: 1)
2. **Learning method** - How MACs are added (dynamic, static, sticky)
3. **Violation action** - What happens on violation (shutdown, restrict, protect)

### Configuration: Basic Port Security

**Cisco IOS Example:**
```cisco
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 2
Switch(config-if)# switchport port-security violation shutdown
Switch(config-if)# switchport port-security mac-address sticky
```

**Step-by-Step Explanation:**

1. **`switchport mode access`** - Port security requires access or trunk mode
2. **`switchport port-security`** - Enables port security
3. **`maximum 2`** - Allows up to 2 MAC addresses
4. **`violation shutdown`** - Disables port on violation
5. **`mac-address sticky`** - Dynamically learned MACs are saved to running-config

### MAC Address Learning Methods

**1. Static Configuration**
```cisco
Switch(config-if)# switchport port-security mac-address 0050.56be.0001
```
- Manually specify allowed MAC addresses
- Survives switch reboot
- Best for critical infrastructure (printers, servers)
- Labor-intensive for large deployments

**2. Dynamic Learning**
```cisco
Switch(config-if)# switchport port-security maximum 3
```
- Switch learns MACs automatically
- Entries lost on switch reload
- Suitable for temporary connections
- Flexible but less secure

**3. Sticky Learning** (Recommended)
```cisco
Switch(config-if)# switchport port-security mac-address sticky
Switch(config-if)# switchport port-security maximum 2
```
- Combines automatic learning with persistence
- Learned MACs saved to running-config (use `copy run start` to persist)
- Balance between security and manageability
- Industry best practice for most deployments

### Violation Actions: Responding to Security Events

When an unauthorized MAC address is detected, three actions are possible:

**1. Shutdown (Default - Most Secure)**
```cisco
Switch(config-if)# switchport port-security violation shutdown
```
- Port enters `err-disabled` state
- All traffic stops immediately
- LED turns amber
- Requires manual intervention: `shutdown` then `no shutdown`
- Best for high-security environments
- Generates SNMP trap and syslog message

**Recovery from err-disabled:**
```cisco
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# shutdown
Switch(config-if)# no shutdown
```

Or enable automatic recovery:
```cisco
Switch(config)# errdisable recovery cause psecure-violation
Switch(config)# errdisable recovery interval 300
```

**2. Restrict (Moderate Security)**
```cisco
Switch(config-if)# switchport port-security violation restrict
```
- Drops packets from unauthorized MACs
- Port remains operational for authorized MACs
- Increments violation counter
- Generates SNMP trap and syslog
- Suitable for user access ports with monitoring

**3. Protect (Minimal Impact)**
```cisco
Switch(config-if)# switchport port-security violation protect
```
- Silently drops packets from unauthorized MACs
- No logging or alerting
- Lowest security (violations go unnoticed)
- Use only for non-critical ports

### Real-World Implementation Scenarios

**Scenario 1: User Access Port**
```cisco
! Single device per port (typical office workstation)
interface range gigabitethernet0/1-24
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 spanning-tree bpduguard enable
```

**Scenario 2: IP Phone + PC**
```cisco
! Port connects IP phone with PC daisy-chained
interface gigabitethernet0/5
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
```

**Scenario 3: Conference Room**
```cisco
! Multiple devices (projector, BYOD, video conferencing)
interface gigabitethernet0/15
 switchport mode access
 switchport access vlan 30
 switchport port-security
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 60
 switchport port-security aging type inactivity
```

### Advanced Features: Aging and Verification

**MAC Address Aging:**
```cisco
! Remove inactive MACs after 2 hours
Switch(config-if)# switchport port-security aging time 120

! Absolute aging (removes after timeout regardless of activity)
Switch(config-if)# switchport port-security aging type absolute

! Inactivity aging (removes only if no traffic seen)
Switch(config-if)# switchport port-security aging type inactivity
```

**Verification Commands:**
```cisco
! Show port security status
Switch# show port-security

! Detailed info for specific port
Switch# show port-security interface gigabitethernet0/1

! Show all secure MAC addresses
Switch# show port-security address

! Check violation counters
Switch# show port-security interface gi0/1 | include Security Violation
```

**Sample Output:**
```
Switch# show port-security interface gi0/1
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 2
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 1
Last Source Address:Vlan   : 0050.56be.0001:10
Security Violation Count   : 0
```

### Common Pitfalls and Best Practices

**❌ Common Mistakes:**

1. **Enabling on trunk ports:** Port security is incompatible with trunk ports
   ```cisco
   ! This will fail
   interface gigabitethernet0/1
    switchport mode trunk
    switchport port-security  ← Error
   ```

2. **Maximum set too low:** IP phones require 2 MAC addresses minimum
3. **Forgetting sticky MACs persist:** Can lock out devices after replacements
4. **No aging configuration:** MAC table grows indefinitely in dynamic environments

**✅ Best Practices:**

1. **Start with restrict mode** during deployment, switch to shutdown after stabilization
2. **Document MAC addresses** for critical infrastructure before going static
3. **Implement aging** for conference rooms and guest networks
4. **Use SNMP traps** to alert on violations
5. **Test recovery procedures** before production deployment
6. **Combine with 802.1X** for identity-based security (see next section)

### Troubleshooting Port Security

**Problem: Port repeatedly enters err-disabled**
```cisco
! Check violation history
show port-security interface gi0/1

! Look for patterns in logs
show logging | include PSECURE

! Verify connected device MAC
show mac address-table interface gi0/1

! Check for MAC flapping
show mac address-table dynamic | include Gi0/1
```

**Problem: Sticky MACs not saving**
```cisco
! Verify sticky is enabled
show run interface gi0/1 | include sticky

! Check running vs startup config
show mac address-table secure

! Save configuration
copy running-config startup-config
```

---

## 9.2 802.1X: Identity-Based Network Access

### Beyond MAC Addresses: The Need for Authentication

Port security prevents MAC flooding but suffers from a fundamental weakness: MAC addresses can be spoofed. An attacker can observe authorized MAC addresses (via packet capture) and clone them onto their device. Port security alone cannot verify the identity of the device or user.

**802.1X** solves this by requiring authentication before granting network access. Based on the Extensible Authentication Protocol (EAP), it provides identity verification using usernames/passwords, digital certificates, or hardware tokens.

### 802.1X Architecture: Three Key Components

**1. Supplicant (Client)**
- Software on the end device
- Built into Windows, macOS, Linux
- Available for IoT devices (varies by vendor)
- Initiates authentication process

**2. Authenticator (Switch)**
- Enforces access control at the port level
- Relays EAP messages between supplicant and authentication server
- Controls port state (unauthorized/authorized)
- Does NOT perform authentication itself

**3. Authentication Server (RADIUS/TACACS+)**
- Validates credentials
- Checks against user database (Active Directory, LDAP, local)
- Returns authorization information (VLAN, ACL, QoS)
- Centralized policy management

**Authentication Flow:**
```
[Supplicant] ←→ [Authenticator/Switch] ←→ [RADIUS Server]
   (PC)           EAP over LAN (EAPOL)      EAP over RADIUS

1. Supplicant → Authenticator: EAPOL-Start
2. Authenticator → Supplicant: EAP-Request/Identity
3. Supplicant → Authenticator → RADIUS: EAP-Response/Identity
4. RADIUS → Authenticator → Supplicant: EAP-Request (challenge)
5. Supplicant → Authenticator → RADIUS: EAP-Response (credentials)
6. RADIUS validates → Sends Access-Accept or Access-Reject
7. Authenticator: Opens port OR keeps blocked
```

### Configuration: Basic 802.1X Setup

**Step 1: Enable AAA (Authentication, Authorization, Accounting)**
```cisco
Switch(config)# aaa new-model
Switch(config)# aaa authentication dot1x default group radius
Switch(config)# aaa authorization network default group radius
```

**Step 2: Configure RADIUS Server**
```cisco
Switch(config)# radius server RADIUS-01
Switch(config-radius-server)# address ipv4 192.168.100.10 auth-port 1812 acct-port 1813
Switch(config-radius-server)# key SecureKey123!
```

**Step 3: Enable 802.1X Globally**
```cisco
Switch(config)# dot1x system-auth-control
```

**Step 4: Configure Ports**
```cisco
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# authentication port-control auto
Switch(config-if)# dot1x pae authenticator
```

### Authentication Port Control Modes

**1. Auto (Standard 802.1X)**
```cisco
Switch(config-if)# authentication port-control auto
```
- Port blocked until authentication succeeds
- Requires 802.1X-capable supplicant
- Most secure mode
- Breaks non-802.1X devices (printers, cameras)

**2. Force-Authorized (Open)**
```cisco
Switch(config-if)# authentication port-control force-authorized
```
- Port always open (no authentication)
- Use for trusted devices or troubleshooting
- Effectively disables 802.1X

**3. Force-Unauthorized (Blocked)**
```cisco
Switch(config-if)# authentication port-control force-unauthorized
```
- Port always blocked
- Use for decommissioned ports
- Better than leaving port shut down (can remotely re-enable)

### Authentication Methods: EAP Variants

**EAP-MD5** (Weak - Not Recommended)
- One-way authentication (client to server only)
- No mutual authentication
- Vulnerable to man-in-the-middle attacks
- Passwords sent hashed but not encrypted
- Legacy compatibility only

**EAP-TLS** (Strongest Security)
```cisco
Switch(config-if)# authentication eap methods eap-tls
```
- Mutual authentication via digital certificates
- Requires PKI infrastructure
- Most secure but complex deployment
- Common in high-security environments (government, finance)

**PEAP (Protected EAP)** - Industry Standard
```cisco
Switch(config-if)# authentication eap methods peap
```
- Server-side certificate, client username/password
- Encrypted TLS tunnel protects credentials
- Balance of security and ease of deployment
- Native Windows/macOS support
- Most common enterprise deployment

**EAP-FAST** (Flexible Authentication via Secure Tunneling)
- Cisco proprietary (now open standard)
- No certificates required
- Uses Protected Access Credentials (PAC)
- Simpler deployment than PEAP

### Advanced Features: MAB and Multi-Auth

**MAC Authentication Bypass (MAB)**

Problem: Non-802.1X devices (printers, IP cameras, VoIP phones) cannot authenticate.

Solution: Fallback to MAC-based authentication.

```cisco
Switch(config)# interface gigabitethernet0/5
Switch(config-if)# authentication port-control auto
Switch(config-if)# mab
Switch(config-if)# authentication order dot1x mab
Switch(config-if)# authentication priority dot1x mab
```

**How it works:**
1. Switch waits for EAPOL-Start (802.1X)
2. If no response after timeout, tries MAB
3. Sends MAC address to RADIUS as username/password
4. RADIUS checks MAC against authorized list
5. Grants or denies access

**Multi-Auth Mode: Multiple Devices Per Port**

```cisco
Switch(config-if)# authentication host-mode multi-auth
Switch(config-if)# authentication port-control auto
```

Allows multiple authenticated devices per port:
- IP phone + PC (most common use case)
- Conference room with BYOD
- Each device authenticates independently
- VLAN assignment per device supported

**Comparison of Host Modes:**

| Mode | Devices | Use Case |
|------|---------|----------|
| **single-host** | 1 authenticated device | High-security environments |
| **multi-host** | Multiple devices, 1 authentication | Guest networks (first device authenticates all) |
| **multi-domain** | 2 devices (voice + data) | IP phone + PC |
| **multi-auth** | Multiple, each authenticated | BYOD, flexible environments |

### Dynamic VLAN Assignment

RADIUS server can assign VLAN based on user identity:

**RADIUS Configuration (Simplified):**
```
User: employee@company.com
  Tunnel-Type = VLAN
  Tunnel-Medium-Type = 802
  Tunnel-Private-Group-ID = 10  ← Assigns VLAN 10

User: guest@company.com
  Tunnel-Type = VLAN
  Tunnel-Medium-Type = 802
  Tunnel-Private-Group-ID = 99  ← Assigns VLAN 99 (guest network)
```

**Switch Configuration:**
```cisco
Switch(config-if)# authentication port-control auto
Switch(config-if)# dot1x pae authenticator
! No static VLAN assignment - RADIUS provides it
```

**Benefits:**
- Single physical network, logical segmentation
- User follows device (laptop moves from office to conference room)
- Consistent policy enforcement
- Centralized management

### Guest VLAN and Auth-Fail VLAN

**Guest VLAN** - For non-802.1X devices:
```cisco
Switch(config-if)# authentication guest-vlan 99
Switch(config-if)# authentication guest-vlan timeout 30
```
- If no 802.1X response, place in guest VLAN
- Limited access (Internet only, no corporate resources)
- Timeout prevents permanent assignment

**Auth-Fail VLAN** - For failed authentication:
```cisco
Switch(config-if)# authentication fail vlan 98
```
- Device attempted 802.1X but failed
- Quarantine VLAN (remediation portal)
- Different from guest VLAN (device has supplicant but invalid credentials)

### Troubleshooting 802.1X

**Common Issues and Solutions:**

**1. Port stays unauthorized**
```cisco
! Check 802.1X is enabled globally
show dot1x all

! Verify RADIUS server connectivity
show radius server-group all

! Test RADIUS communication
test aaa group radius server 192.168.100.10 username testuser password testpass

! Check port configuration
show authentication sessions interface gi0/1
```

**2. RADIUS timeouts**
```cisco
! Verify IP connectivity to RADIUS server
ping 192.168.100.10

! Check RADIUS shared secret matches
show radius server-group all

! Enable debug (caution in production)
debug dot1x all
debug radius authentication
```

**3. Wrong VLAN assignment**
```cisco
! Verify RADIUS attributes
show authentication sessions interface gi0/1 details

! Check RADIUS server logs
! Ensure Tunnel-Private-Group-ID matches VLAN ID
```

**Verification Commands:**
```cisco
! Show all authenticated sessions
show authentication sessions

! Detailed view of specific port
show authentication sessions interface gi0/1 details

! 802.1X statistics
show dot1x statistics interface gi0/1

! View supplicant information
show dot1x interface gi0/1
```

---

## 9.3 DHCP Snooping: Protecting Address Assignment

### The Threat: Rogue DHCP Servers

DHCP operates without authentication—any device can respond to DHCP discover messages. Attackers exploit this by running rogue DHCP servers that:

1. **Provide malicious default gateway** → Man-in-the-middle attacks (all traffic routed through attacker)
2. **Supply attacker's DNS server** → Phishing via DNS spoofing
3. **Assign invalid IP addresses** → Denial of service
4. **Steal credentials** → Captive portal mimics legitimate login

**Attack Scenario:**
```
1. Legitimate DHCP server: 192.168.1.1
2. Attacker runs rogue DHCP on 192.168.1.100
3. Client broadcasts DHCP Discover
4. Both servers respond with DHCP Offer
5. Client accepts first response (often the attacker's)
6. Attacker-provided default gateway: 192.168.1.100
7. All client traffic flows through attacker
```

### DHCP Snooping Fundamentals

DHCP Snooping creates a firewall between untrusted ports (user devices) and trusted ports (legitimate DHCP servers). It:

1. **Validates DHCP messages** from untrusted ports
2. **Builds a binding table** (IP-MAC-Port-VLAN mapping)
3. **Rate-limits DHCP traffic** to prevent DoS
4. **Enables downstream security features** (DAI, IP Source Guard)

**Port Classification:**
- **Trusted ports:** Uplinks to DHCP servers, inter-switch links
- **Untrusted ports:** Access ports (user devices)

### Configuration: Enabling DHCP Snooping

**Step 1: Enable DHCP Snooping Globally**
```cisco
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10,20,30
```

**Step 2: Configure Trusted Ports**
```cisco
Switch(config)# interface gigabitethernet0/24
Switch(config-if)# ip dhcp snooping trust
```

**Step 3: Rate Limiting (Optional but Recommended)**
```cisco
Switch(config)# interface range gigabitethernet0/1-23
Switch(config-if-range)# ip dhcp snooping limit rate 10
```
- Limits DHCP packets to 10 per second
- Prevents DHCP starvation attacks (flooding DISCOVER messages)

**Step 4: Verify Configuration**
```cisco
Switch# show ip dhcp snooping
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
10,20,30
Insertion of option 82 is enabled
Interface                  Trusted    Rate limit (pps)
-----------------------    -------    ----------------
GigabitEthernet0/1         no         10
GigabitEthernet0/24        yes        unlimited
```

### The DHCP Snooping Binding Table

The binding table is a database of legitimate DHCP assignments:

```cisco
Switch# show ip dhcp snooping binding
MacAddress          IpAddress    Lease(sec)  Type           VLAN  Interface
------------------  -----------  ----------  -------------  ----  -----------------
00:50:56:BE:00:01   10.1.1.100   86400       dhcp-snooping  10    GigabitEthernet0/5
00:50:56:BE:00:02   10.1.1.101   86400       dhcp-snooping  10    GigabitEthernet0/6
```

**Table entries include:**
- MAC address of client
- Assigned IP address
- Lease duration
- VLAN ID
- Switch port

**Critical for downstream features:**
- **Dynamic ARP Inspection (DAI):** Validates ARP packets against binding table
- **IP Source Guard:** Filters traffic not matching binding table

### DHCP Option 82: Relay Agent Information

By default, DHCP snooping inserts **Option 82** (relay agent information) into DHCP packets:
- Circuit ID (switch port identifier)
- Remote ID (switch MAC address)

**Potential Problem:** Some DHCP servers reject packets with Option 82 from clients.

**Solution 1: Disable Option 82 globally**
```cisco
Switch(config)# no ip dhcp snooping information option
```

**Solution 2: Trust port and allow Option 82 passthrough**
```cisco
Switch(config-if)# ip dhcp snooping trust
```

**When to use Option 82:**
- Enables per-port tracking by DHCP server
- Required for some ISP environments
- Useful for detailed logging and forensics

### Advanced: Binding Table Persistence

By default, the binding table is lost on reboot. For high-availability environments:

```cisco
! Store bindings to flash
Switch(config)# ip dhcp snooping database flash:dhcp-snooping.db

! Or store to TFTP server
Switch(config)# ip dhcp snooping database tftp://192.168.1.100/dhcp-snooping.db
```

### Troubleshooting DHCP Snooping

**Problem: Clients can't get DHCP addresses**

```cisco
! Verify snooping is enabled
show ip dhcp snooping

! Check trusted ports
show ip dhcp snooping | include Trusted

! Verify uplink to DHCP server is trusted
show run interface gi0/24 | include trust

! Check rate limiting isn't blocking legitimate traffic
show ip dhcp snooping statistics
```

**Problem: Binding table is empty**

```cisco
! Ensure snooping is enabled on correct VLANs
show ip dhcp snooping

! Trigger new DHCP request from client
! Client: ipconfig /release && ipconfig /renew

! Watch snooping in action
debug ip dhcp snooping event
debug ip dhcp snooping packet
```

---

## 9.4 Dynamic ARP Inspection (DAI): Preventing ARP Poisoning

### The ARP Vulnerability

ARP (Address Resolution Protocol) maps IP addresses to MAC addresses but operates without authentication. Any device can send gratuitous ARP messages claiming to own an IP address:

**ARP Poisoning Attack:**
```
1. Normal operation:
   Gateway (192.168.1.1) is at MAC AA:AA:AA:AA:AA:AA
   Client stores in ARP cache

2. Attacker sends malicious ARP:
   "192.168.1.1 is at MAC BB:BB:BB:BB:BB:BB" ← Attacker's MAC

3. Client updates ARP cache with false information
4. Client sends all Internet-bound traffic to attacker
5. Attacker intercepts, reads, and forwards traffic (MITM attack)
```

### Dynamic ARP Inspection (DAI) Solution

DAI validates ARP packets against the **DHCP snooping binding table**. Packets from untrusted ports are checked:
- Does the source IP match the binding table?
- Does the source MAC match the binding table?
- Is the ARP sender's IP/MAC valid?

**Invalid ARP packets are dropped** and violations are logged.

### Configuration: Enabling DAI

**Prerequisites:**
1. DHCP snooping must be enabled (builds binding table)
2. DHCP snooping binding table must be populated

**Step 1: Enable DAI on VLANs**
```cisco
Switch(config)# ip arp inspection vlan 10,20,30
```

**Step 2: Configure Trusted Ports**
```cisco
! Trust uplinks and server connections
Switch(config)# interface gigabitethernet0/24
Switch(config-if)# ip arp inspection trust

! Trust inter-switch links
Switch(config)# interface gigabitethernet0/23
Switch(config-if)# ip arp inspection trust
```

**Step 3: Rate Limiting (Automatic)**
```cisco
! Default: 15 ARP packets per second (pps)
! Customize if needed:
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# ip arp inspection limit rate 20
```

**Step 4: Verify Configuration**
```cisco
Switch# show ip arp inspection

Source Mac Validation      : Disabled
Destination Mac Validation : Disabled
IP Address Validation      : Disabled

 Vlan     Configuration    Operation   ACL Match          Static ACL
 ----     -------------    ---------   ---------          ----------
   10        Enabled          Active
   20        Enabled          Active

 Vlan     ACL Logging      DHCP Logging      Probe Logging
 ----     -----------      ------------      -------------
   10        Deny             Deny              Off
   20        Deny             Deny              Off
```

### Additional Validation: MAC and IP Address Checks

By default, DAI only validates ARP packets against the binding table. Enable additional checks:

```cisco
! Validate source MAC in Ethernet header matches sender MAC in ARP payload
Switch(config)# ip arp inspection validate src-mac

! Validate destination MAC
Switch(config)# ip arp inspection validate dst-mac

! Validate IP addresses
Switch(config)# ip arp inspection validate ip

! Enable all validations (recommended)
Switch(config)# ip arp inspection validate src-mac dst-mac ip
```

**Why additional validation matters:**
- Catches malformed ARP packets
- Detects spoofed Ethernet headers
- Identifies attacks using valid IP/MAC pairs from binding table but wrong packet structure

### Handling Static IP Devices

Problem: Devices with static IP addresses aren't in the DHCP snooping binding table.

**Solution 1: ARP Access Control Lists (ACLs)**
```cisco
! Define permitted static IP/MAC mappings
Switch(config)# arp access-list STATIC-DEVICES
Switch(config-arp-nacl)# permit ip host 10.1.1.50 mac host 0050.56BE.1234
Switch(config-arp-nacl)# permit ip host 10.1.1.51 mac host 0050.56BE.5678

! Apply to VLAN
Switch(config)# ip arp inspection filter STATIC-DEVICES vlan 10
```

**Solution 2: Trust the port (less secure)**
```cisco
Switch(config)# interface gigabitethernet0/10
Switch(config-if)# ip arp inspection trust
```

### Logging and Monitoring

**View DAI Statistics:**
```cisco
Switch# show ip arp inspection statistics vlan 10

 Vlan      Forwarded        Dropped     DHCP Drops      ACL Drops
 ----      ---------        -------     ----------      ---------
   10           1450             23              15              8

 Vlan   DHCP Permits    ACL Permits  Probe Permits   Source MAC Failures
 ----   ------------    -----------  -------------   -------------------
   10           1400             50              0                    12

 Vlan   Dest MAC Failures   IP Validation Failures   Invalid Protocol Data
 ----   -----------------   ----------------------   ---------------------
   10                   5                        6                       0
```

**Syslog Messages:**
```
%SW_DAI-4-DHCP_SNOOPING_DENY: 1 Invalid ARPs (Req) on Gi0/5, vlan 10
  ([0050.56be.9999/10.1.1.200/0000.0000.0000/10.1.1.1/09:30:15 UTC])
```

**Log Buffer:**
```cisco
Switch(config)# ip arp inspection log-buffer entries 1024
Switch(config)# ip arp inspection log-buffer logs 512 interval 30
```

### Troubleshooting DAI

**Problem: Legitimate traffic being dropped**

```cisco
! Check DAI statistics for drop reasons
show ip arp inspection statistics vlan 10

! Verify DHCP snooping binding table has correct entries
show ip dhcp snooping binding

! Ensure uplinks are trusted
show ip arp inspection interfaces

! Check for static IP devices without ACL entries
show ip arp inspection
```

**Problem: ARP spoofing still occurring**

```cisco
! Verify DAI is enabled on correct VLANs
show ip arp inspection

! Ensure additional validations are enabled
show run | include arp inspection validate

! Check for trusted ports that shouldn't be trusted
show ip arp inspection interfaces | include Trust

! Monitor for violations
show ip arp inspection log
```

---

## 9.5 Private VLANs: Isolation Within a VLAN

### The Use Case: Shared Infrastructure Isolation

Traditional VLANs provide isolation between different groups but not within a group. Consider a data center hosting multiple customers:

**Problem:**
- Customer A: VLANs 10-15
- Customer B: VLANs 20-25
- Customer C: VLANs 30-35

If customers A, B, and C each need 100 VLANs, you exhaust the 4096 VLAN limit quickly. Additionally, you may want isolation between customer devices while allowing all to reach a common gateway.

**Private VLANs (PVLANs)** solve this by subdividing a single VLAN into isolated sub-domains.

### Private VLAN Types

**1. Primary VLAN**
- The main VLAN that contains all sub-VLANs
- All traffic appears to be in this VLAN from outside the switch

**2. Isolated VLAN (Complete Isolation)**
- Hosts cannot communicate with each other
- Hosts can only communicate with promiscuous ports
- Use case: Web hosting (servers shouldn't talk to each other)

**3. Community VLAN (Group Isolation)**
- Hosts can communicate within their community
- Cannot communicate with other communities or isolated ports
- Can communicate with promiscuous ports
- Use case: Department grouping within an organization

**4. Promiscuous Port**
- Can communicate with all ports (primary, isolated, community)
- Typically connects to routers, firewalls, or shared resources

**Visual Example:**
```
Primary VLAN 100
├── Promiscuous Port (Gi0/24) → Router
├── Isolated VLAN 101
│   ├── Port Gi0/1 (Server A) ← Cannot talk to Server B
│   └── Port Gi0/2 (Server B) ← Cannot talk to Server A
├── Community VLAN 102 (Engineering)
│   ├── Port Gi0/3 (Eng PC 1) ← Can talk to Eng PC 2
│   └── Port Gi0/4 (Eng PC 2) ← Can talk to Eng PC 1
└── Community VLAN 103 (Sales)
    ├── Port Gi0/5 (Sales PC 1) ← Can talk to Sales PC 2
    └── Port Gi0/6 (Sales PC 2) ← Cannot talk to Engineering
```

### Configuration: Private VLANs

**Step 1: Enable VTP Transparent Mode**
```cisco
! Private VLANs require VTP transparent or off mode
Switch(config)# vtp mode transparent
```

**Step 2: Create Primary and Secondary VLANs**
```cisco
! Create primary VLAN
Switch(config)# vlan 100
Switch(config-vlan)# private-vlan primary
Switch(config-vlan)# exit

! Create isolated VLAN
Switch(config)# vlan 101
Switch(config-vlan)# private-vlan isolated
Switch(config-vlan)# exit

! Create community VLANs
Switch(config)# vlan 102
Switch(config-vlan)# private-vlan community
Switch(config-vlan)# exit

Switch(config)# vlan 103
Switch(config-vlan)# private-vlan community
Switch(config-vlan)# exit
```

**Step 3: Associate Secondary VLANs with Primary**
```cisco
Switch(config)# vlan 100
Switch(config-vlan)# private-vlan association 101,102,103
Switch(config-vlan)# exit
```

**Step 4: Configure Promiscuous Port**
```cisco
Switch(config)# interface gigabitethernet0/24
Switch(config-if)# switchport mode private-vlan promiscuous
Switch(config-if)# switchport private-vlan mapping 100 101,102,103
```

**Step 5: Configure Host Ports**
```cisco
! Isolated port
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# switchport mode private-vlan host
Switch(config-if)# switchport private-vlan host-association 100 101

! Community port (Engineering)
Switch(config)# interface gigabitethernet0/3
Switch(config-if)# switchport mode private-vlan host
Switch(config-if)# switchport private-vlan host-association 100 102

! Community port (Sales)
Switch(config)# interface gigabitethernet0/5
Switch(config-if)# switchport mode private-vlan host
Switch(config-if)# switchport private-vlan host-association 100 103
```

### Verification and Troubleshooting

```cisco
! Show private VLAN configuration
Switch# show vlan private-vlan

Primary Secondary Type              Ports
------- --------- ----------------- ---------------------------
100     101       isolated          Gi0/1, Gi0/2
100     102       community         Gi0/3, Gi0/4
100     103       community         Gi0/5, Gi0/6

! Show private VLAN port mapping
Switch# show interface gigabitethernet0/24 switchport
Name: Gi0/24
Switchport: Enabled
Administrative Mode: private-vlan promiscuous
Operational Mode: private-vlan promiscuous
Administrative Private VLAN Mapping: 100 (primary) 101,102,103 (secondary)
```

### Real-World Scenario: Multi-Tenant Data Center

**Requirements:**
- 3 customers sharing infrastructure
- Each customer has 5 servers
- Servers must not communicate with each other (security)
- All servers need access to shared storage and Internet gateway

**Configuration:**
```cisco
! Primary VLAN for all customers
vlan 200
 private-vlan primary
 private-vlan association 201,202,203

! Isolated VLANs (one per customer)
vlan 201
 private-vlan isolated
vlan 202
 private-vlan isolated
vlan 203
 private-vlan isolated

! Promiscuous port to gateway/storage
interface gigabitethernet0/48
 switchport mode private-vlan promiscuous
 switchport private-vlan mapping 200 201,202,203

! Customer A servers (VLAN 201)
interface range gigabitethernet0/1-5
 switchport mode private-vlan host
 switchport private-vlan host-association 200 201

! Customer B servers (VLAN 202)
interface range gigabitethernet0/6-10
 switchport mode private-vlan host
 switchport private-vlan host-association 200 202

! Customer C servers (VLAN 203)
interface range gigabitethernet0/11-15
 switchport mode private-vlan host
 switchport private-vlan host-association 200 203
```

---

## 9.6 Access Control Lists (ACLs): Filtering at Layer 2/3

### ACL Fundamentals

While routers commonly use ACLs for Layer 3 filtering, switches support:
- **Router ACLs (RACL):** Applied to SVIs (Switched Virtual Interfaces)
- **VLAN ACLs (VACL):** Applied to all traffic within a VLAN
- **Port ACLs (PACL):** Applied to specific switch ports

**Precedence:** PACL > RACL > VACL

### VLAN Access Control Lists (VACLs)

VACLs filter traffic within a VLAN or between VLANs on the same switch—catching intra-VLAN attacks that bypass routing.

**Use Cases:**
- Block specific protocols within a VLAN
- Prevent IP spoofing from specific hosts
- Log suspicious traffic patterns

**Configuration Example: Block TFTP within VLAN 10**
```cisco
! Define ACL
Switch(config)# ip access-list extended BLOCK-TFTP
Switch(config-ext-nacl)# deny udp any any eq 69
Switch(config-ext-nacl)# permit ip any any
Switch(config-ext-nacl)# exit

! Create VLAN access map
Switch(config)# vlan access-map VLAN10-FILTER 10
Switch(config-access-map)# match ip address BLOCK-TFTP
Switch(config-access-map)# action forward
Switch(config-access-map)# exit

Switch(config)# vlan access-map VLAN10-FILTER 20
Switch(config-access-map)# action forward
Switch(config-access-map)# exit

! Apply to VLAN
Switch(config)# vlan filter VLAN10-FILTER vlan-list 10
```

### Port ACLs (PACLs)

Applied to Layer 2 interfaces, filtering traffic entering the port.

**Configuration Example: Restrict HTTP traffic on access port**
```cisco
! Define ACL
Switch(config)# ip access-list extended RESTRICT-HTTP
Switch(config-ext-nacl)# deny tcp any any eq 80
Switch(config-ext-nacl)# permit ip any any

! Apply to interface
Switch(config)# interface gigabitethernet0/5
Switch(config-if)# ip access-group RESTRICT-HTTP in
```

### MAC ACLs: Layer 2 Filtering

Filter based on source/destination MAC addresses and EtherType.

**Configuration Example: Allow only specific MAC addresses**
```cisco
Switch(config)# mac access-list extended MAC-FILTER
Switch(config-ext-macl)# permit host 0050.56be.0001 any
Switch(config-ext-macl)# permit host 0050.56be.0002 any
Switch(config-ext-macl)# deny any any

Switch(config)# interface gigabitethernet0/10
Switch(config-if)# mac access-group MAC-FILTER in
```

---

## 9.7 Security Best Practices Summary

### Layered Security Approach

Combine multiple features for defense in depth:

```cisco
! Example: Secure user access port
interface gigabitethernet0/1
 ! Layer 2 security
 switchport mode access
 switchport access vlan 10

 ! Port security
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky

 ! 802.1X authentication
 authentication port-control auto
 dot1x pae authenticator
 mab
 authentication order dot1x mab

 ! DHCP snooping (untrusted by default)
 ip dhcp snooping limit rate 10

 ! DAI (untrusted by default)
 ip arp inspection limit rate 20

 ! Spanning tree protection
 spanning-tree portfast
 spanning-tree bpduguard enable

 ! Storm control
 storm-control broadcast level 10.00
 storm-control multicast level 10.00
```

### Global Security Baseline

```cisco
! Enable essential security features
ip dhcp snooping
ip dhcp snooping vlan 1-4094
ip arp inspection vlan 1-4094
ip arp inspection validate src-mac dst-mac ip

! Disable unused services
no ip http server
no ip http secure-server
no cdp run  ! Or selectively disable per port
no lldp run

! Secure management access
enable secret <strong-password>
service password-encryption
login block-for 120 attempts 3 within 60

! Configure AAA
aaa new-model
aaa authentication login default group tacacs+ local
aaa authorization exec default group tacacs+ local

! Secure VTY lines
line vty 0 15
 transport input ssh
 logging synchronous
 exec-timeout 10 0
```

### Monitoring and Incident Response

**Enable logging:**
```cisco
logging buffered 51200 informational
logging console warnings
logging trap informational
logging host 192.168.100.50
```

**SNMP traps for security events:**
```cisco
snmp-server enable traps port-security
snmp-server enable traps dot1x
snmp-server host 192.168.100.60 version 2c <community>
```

**Regular audits:**
```cisco
! Review port security violations
show port-security

! Check 802.1X authentication failures
show authentication sessions

! Analyze DHCP snooping bindings
show ip dhcp snooping binding

! Review DAI statistics
show ip arp inspection statistics
```

---

## Key Takeaways

1. **Port security** prevents MAC flooding but can be bypassed via spoofing—combine with 802.1X
2. **802.1X** provides identity-based access control; use MAB for non-supplicant devices
3. **DHCP snooping** builds the foundation for DAI and IP Source Guard
4. **DAI** prevents ARP poisoning attacks using the DHCP snooping binding table
5. **Private VLANs** enable isolation within VLANs—critical for multi-tenant environments
6. **ACLs** on switches filter traffic at Layer 2/3; use VACLs for intra-VLAN filtering
7. **Defense in depth**: Layer multiple security features for comprehensive protection
8. **Monitoring**: Security features are only effective with proper logging and alerting

Security is not a one-time configuration but an ongoing process requiring monitoring, updating, and adaptation to emerging threats.
