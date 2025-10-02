# Chapter 12: Advanced Switching Features

## Introduction

Beyond basic switching, VLANs, and routing, modern switches offer advanced features that optimize network performance, enhance security, and enable sophisticated traffic management. This chapter explores multicast switching, jumbo frames, storm control, port mirroring, and other advanced capabilities that separate enterprise-grade switches from basic models.

Understanding these features allows network engineers to design high-performance, resilient networks that meet the demands of modern applications—from IP video surveillance and VoIP to data center storage and cloud computing.

---

## 12.1 Multicast Switching

### The Multicast Problem

**Unicast:** One sender → One receiver (efficient for point-to-point)
**Broadcast:** One sender → All receivers (wasteful, floods entire VLAN)
**Multicast:** One sender → Multiple interested receivers (efficient for group communication)

**Use Cases:**
- IP video conferencing (Zoom, Teams)
- IPTV and digital signage
- Stock ticker updates
- Software distribution
- Live streaming events
- Multicast DNS (mDNS) / Bonjour

**Without Multicast Optimization:**

```
Video Stream ──► Switch ──┬──► Interested Client 1
                          ├──► Interested Client 2
                          ├──► Uninterested Client 3 ← Wastes bandwidth!
                          └──► Uninterested Client 4 ← Wastes bandwidth!

Switch floods multicast to all ports (like broadcast)
```

### IGMP Snooping: Intelligent Multicast Forwarding

**IGMP (Internet Group Management Protocol)** allows hosts to join/leave multicast groups.
**IGMP Snooping** makes switches listen to IGMP messages and forward multicast traffic only to interested ports.

**How IGMP Snooping Works:**

```
1. Client sends IGMP Membership Report: "I want group 239.1.1.1"
2. Switch learns: Port Gi0/5 wants group 239.1.1.1
3. Multicast traffic for 239.1.1.1 sent ONLY to Gi0/5
4. Client sends IGMP Leave: "I'm leaving 239.1.1.1"
5. Switch removes Gi0/5 from group (after query timeout)
```

**Configuration:**

```cisco
! Enable IGMP snooping globally (usually enabled by default)
Switch(config)# ip igmp snooping

! Enable on specific VLAN
Switch(config)# ip igmp snooping vlan 10

! Configure querier (switch acts as IGMP query source if no router present)
Switch(config)# ip igmp snooping vlan 10 querier
Switch(config)# ip igmp snooping vlan 10 querier address 10.1.10.254
```

**Verification:**

```cisco
Switch# show ip igmp snooping
Global IGMP Snooping configuration:
-----------------------------------
IGMP snooping                 : Enabled
IGMPv3 snooping (minimal)     : Enabled
Report suppression            : Enabled
TCN solicit query             : Disabled
TCN flood query count         : 2

Switch# show ip igmp snooping groups
Vlan      Group                    Type        Version     Port List
---------------------------------------------------------------------
10        239.1.1.1                igmp        v2          Gi0/5, Gi0/8
10        239.1.1.5                igmp        v3          Gi0/12
```

### Multicast VLAN Registration (MVR)

**Use Case:** Shared multicast stream across VLANs (IPTV to multiple subscriber VLANs)

```cisco
! Create multicast VLAN
Switch(config)# vlan 100
Switch(config-vlan)# name Multicast-VLAN

! Enable MVR
Switch(config)# mvr
Switch(config)# mvr vlan 100
Switch(config)# mvr mode compatible

! Configure source port (where multicast originates)
Switch(config)# interface gigabitethernet0/24
Switch(config-if)# mvr type source

! Configure receiver ports
Switch(config)# interface range gigabitethernet0/1-10
Switch(config-if-range)# mvr type receiver
Switch(config-if-range)# mvr vlan 100 group 239.1.1.1
```

**Benefit:** Single multicast stream shared across VLANs without needing Layer 3 routing.

---

## 12.2 Jumbo Frames: Increasing MTU

### Understanding MTU

**MTU (Maximum Transmission Unit):** Largest packet size that can be transmitted without fragmentation.

- **Standard Ethernet:** 1500 bytes
- **Jumbo Frames:** 9000 bytes (or 9216 bytes on some switches)

**Why Jumbo Frames?**
- Reduce CPU overhead (fewer packets to process)
- Lower latency (less packet processing time)
- Higher throughput for large data transfers
- Essential for iSCSI, NFS, and backup traffic

**Performance Gain Example:**

```
Transferring 90,000 bytes:

Standard MTU (1500):
  90,000 / 1500 = 60 packets
  60 × packet overhead = High CPU usage

Jumbo Frames (9000):
  90,000 / 9000 = 10 packets
  10 × packet overhead = Low CPU usage (6× fewer packets)
```

### Configuration

**Enable Jumbo Frames:**

```cisco
! Set system MTU (requires reboot on some platforms)
Switch(config)# system mtu 9000
Switch(config)# exit
Switch# reload

! Or configure per-interface (on supported platforms)
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# mtu 9000

! For routed interfaces (Layer 3)
Switch(config)# interface gigabitethernet0/24
Switch(config-if)# no switchport
Switch(config-if)# ip mtu 9000
```

**Verification:**

```cisco
Switch# show system mtu
System MTU size is 9000 bytes

Switch# show interfaces gigabitethernet0/1 | include MTU
  MTU 9000 bytes, BW 1000000 Kbit/sec
```

**⚠️ Important Considerations:**

1. **End-to-end support required:** All devices in the path must support jumbo frames
2. **MTU mismatch = fragmentation:** Fragmentation negates performance benefits
3. **Testing required:** Use ping with large packets:
   ```
   ping 10.1.1.1 size 9000 df-bit
   ```
4. **Not for general traffic:** Use only for backend storage, backups, and server-to-server

---

## 12.3 Storm Control: Preventing Traffic Floods

### The Problem: Broadcast/Multicast Storms

**Causes:**
- Network loops (STP misconfiguration)
- Faulty NIC (floods broadcast frames)
- Malware or DoS attack
- Misconfigured applications

**Impact:**
- 100% link utilization
- Switch CPU overload
- Network outage

### Storm Control Configuration

**Limit Broadcast, Multicast, and Unknown Unicast:**

```cisco
! Configure on access ports
Switch(config)# interface range gigabitethernet0/1-24
Switch(config-if-range)# storm-control broadcast level 10.00
Switch(config-if-range)# storm-control multicast level 10.00
Switch(config-if-range)# storm-control unicast level 10.00

! Action when threshold exceeded
Switch(config-if-range)# storm-control action shutdown
```

**Parameters:**

- **Level:** Percentage of bandwidth (10.00 = 10%)
- **Actions:**
  - `shutdown` - Err-disable the port (most secure)
  - `trap` - Send SNMP trap (alert only)

**Rising/Falling Thresholds:**

```cisco
! Trigger at 15%, stop action at 10%
Switch(config-if)# storm-control broadcast level 10.00 15.00
```

**Verification:**

```cisco
Switch# show storm-control
Interface  Filter State   Upper       Lower      Current
---------  -------------  ----------  ---------  ---------
Gi0/1      Forwarding     10.00%      10.00%     0.50%
Gi0/2      Forwarding     10.00%      10.00%     1.20%
```

---

## 12.4 UDLD: Detecting Unidirectional Links

### The Problem

**Unidirectional Link:** Traffic flows one direction but not the other.

**Causes:**
- Fiber patch cable with one strand broken
- Failed transceiver (TX works, RX fails)
- Port buffer issues

**Danger:**
- Spanning Tree Protocol (STP) relies on bidirectional communication
- Unidirectional link can cause loops

### UDLD Operation

**UDLD (UniDirectional Link Detection)** sends periodic messages and expects to receive its own Device ID back.

```
Switch A ──────────────► Switch B (TX working)
         ◄X─────────────           (RX broken)

Switch A sends UDLD message with Device ID "Switch-A"
Switch B should echo back message showing it heard from "Switch-A"
Switch A never sees its Device ID returned → Detects unidirectional link
```

**Configuration:**

```cisco
! Enable UDLD globally for fiber ports
Switch(config)# udld enable

! Enable aggressive mode (faster detection, less forgiving)
Switch(config)# udld aggressive

! Enable on specific interface
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# udld port aggressive
```

**UDLD Modes:**

- **Normal:** Logs warning, marks port as undetermined
- **Aggressive:** Err-disables port after 8 failed retries (recommended)

**Verification:**

```cisco
Switch# show udld gigabitethernet0/1
Interface Gi0/1
---
Port enable administrative configuration setting: Enabled / in aggressive mode
Port enable operational state: Enabled / in aggressive mode
Current bidirectional state: Bidirectional
Message interval: 15 seconds
Time out interval: 5 seconds
```

---

## 12.5 Port Mirroring Advanced Features

### ERSPAN (Encapsulated Remote SPAN)

**Problem with RSPAN:** Requires dedicated VLAN across network

**ERSPAN Solution:** Encapsulates mirrored traffic in GRE tunnel

```cisco
! Source switch
Switch(config)# monitor session 1 type erspan-source
Switch(config-mon-erspan-src)# source interface gigabitethernet0/5
Switch(config-mon-erspan-src)# destination
Switch(config-mon-erspan-src-dst)# erspan-id 100
Switch(config-mon-erspan-src-dst)# ip address 192.168.100.50
Switch(config-mon-erspan-src-dst)# origin ip address 192.168.100.1

! Destination switch (or packet analyzer with IP)
Switch(config)# monitor session 2 type erspan-destination
Switch(config-mon-erspan-dst)# destination interface gigabitethernet0/24
Switch(config-mon-erspan-dst)# source
Switch(config-mon-erspan-dst-src)# erspan-id 100
Switch(config-mon-erspan-dst-src)# ip address 192.168.100.1
```

**Benefits:**
- No VLAN required
- Can span across Layer 3 networks
- Multiple sources to one destination

### SPAN Filtering

**Mirror only specific traffic:**

```cisco
! Create ACL
Switch(config)# ip access-list extended SPAN-FILTER
Switch(config-ext-nacl)# permit tcp any host 10.1.1.100 eq 443

! Apply to SPAN session
Switch(config)# monitor session 1 source interface gi0/5
Switch(config)# monitor session 1 filter access-group SPAN-FILTER
Switch(config)# monitor session 1 destination interface gi0/24
```

---

## 12.6 FlexLink: Simple Redundancy

### Use Case

**Problem:** Need link redundancy without Spanning Tree Protocol complexity

**FlexLink:** Provides active/standby link pairing

```
                    ┌──────────────┐
                    │   Switch A   │
                    └──────────────┘
                         │    │
                  Active │    │ Standby
                         │    │
                    ┌────▼────▼────┐
                    │  Access SW   │
                    └──────────────┘
```

**Configuration:**

```cisco
! On access switch
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# switchport backup interface gigabitethernet0/2

! Optional: Preemption (return to primary when it comes back up)
Switch(config-if)# switchport backup interface gigabitethernet0/2 preemption mode forced
```

**Verification:**

```cisco
Switch# show interfaces switchport backup
Switch Backup Interface Pairs:

Active Interface        Backup Interface        State
------------------------------------------------------------------------
GigabitEthernet0/1      GigabitEthernet0/2      Active Up/Backup Standby
```

---

## 12.7 Auto-Configuration and Zero-Touch Provisioning

### Smart Install

**Automatically configure switches without manual intervention**

**Use Case:** Deploy hundreds of access switches with consistent configuration

**Configuration (Director Switch):**

```cisco
Switch(config)# vstack
Switch(config-vstack)# director
```

**Zero-Touch Provisioning (ZTP):**

```
1. New switch boots with no config
2. DHCP provides IP address + TFTP server info
3. Switch downloads config file from TFTP
4. Applies configuration and reboots
5. Switch is production-ready
```

### Auto-MDIX

**Automatically detects and corrects cable type** (straight-through vs. crossover)

```cisco
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# mdix auto
```

Eliminates need for crossover cables in switch-to-switch or PC-to-PC connections.

---

## 12.8 Stacking and Modular Chassis

### Switch Stacking

**Combine multiple switches into single logical unit**

```
┌────────────┐
│  Switch 1  │ ← Master (manages stack)
│  (Master)  │
└─────┬──────┘
      │ Stack Cable (80 Gbps)
┌─────▼──────┐
│  Switch 2  │ ← Member
│  (Member)  │
└─────┬──────┘
      │
┌─────▼──────┐
│  Switch 3  │ ← Member
│  (Member)  │
└────────────┘
```

**Benefits:**
- Single management IP
- Unified configuration
- Cross-switch EtherChannel
- High-speed backplane
- Automatic failover

**Configuration (Cisco StackWise):**

```cisco
! Set switch priority (highest becomes master)
Switch(config)# switch 1 priority 15
Switch(config)# switch 2 priority 10
Switch(config)# switch 3 priority 5

! Renumber switch (change stack member ID)
Switch(config)# switch 2 renumber 3

! View stack status
Switch# show switch
Switch/Stack Mac Address : 0050.56be.0001
                                           H/W   Current
Switch#  Role   Mac Address     Priority Version  State
--------------------------------------------------------------
*1       Master 0050.56be.0001     15     V01     Ready
 2       Member 0050.56be.0002     10     V01     Ready
 3       Member 0050.56be.0003      5     V01     Ready
```

### Virtual Switching System (VSS) / StackWise Virtual

**Combine two physical switches into one logical switch**

```
┌──────────────┐       Virtual        ┌──────────────┐
│  Switch 1    │◄───Switch Link (VSL)─►│  Switch 2    │
│  (Active)    │                       │  (Standby)   │
└──────┬───────┘                       └──────┬───────┘
       │                                      │
       └──────────── EtherChannel ────────────┘
                         ▼
                   ┌────────────┐
                   │ Core Switch│
                   └────────────┘
```

**Benefits:**
- No Spanning Tree blocking (both uplinks active)
- Unified control plane
- Hitless failover

---

## Key Takeaways

1. **IGMP Snooping** prevents multicast flooding by forwarding only to interested hosts
2. **Jumbo frames** boost performance for storage and backup traffic but require end-to-end support
3. **Storm control** protects against broadcast/multicast floods that can disable networks
4. **UDLD** detects unidirectional links that can cause STP loops on fiber connections
5. **ERSPAN** enables remote packet capture across Layer 3 networks
6. **FlexLink** provides simple active/standby redundancy without STP overhead
7. **Stacking** combines switches for simplified management and higher availability

Advanced features transform basic switches into sophisticated network appliances capable of optimizing performance, enhancing security, and improving operational efficiency.

---

**Chapter 12 Complete** | Next: Chapter 13 - High Availability and Redundancy
