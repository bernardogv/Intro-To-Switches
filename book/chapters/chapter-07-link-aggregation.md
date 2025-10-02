# Chapter 7: Link Aggregation (EtherChannel)

## 7.1 Introduction to Link Aggregation

Link aggregation, known as **EtherChannel** on Cisco switches and **Port Channel** on other vendors, combines multiple physical links into a single logical link. This provides increased bandwidth, load balancing, and redundancy without triggering Spanning Tree Protocol (STP) to block ports.

### 7.1.1 Why Link Aggregation?

**Without EtherChannel:**
```
    [Switch A]
      |  |  |  ← Three 1 Gbps links
      |  |  |
    [Switch B]
```
- STP blocks 2 links to prevent loops
- Effective bandwidth: 1 Gbps
- Wasted capacity: 2 Gbps

**With EtherChannel:**
```
    [Switch A]
      ║  ║  ║  ← Bundled into single logical channel
      ║  ║  ║
    [Switch B]
```
- All links active
- Effective bandwidth: 3 Gbps
- STP sees one logical link (no blocking)

### 7.1.2 Benefits of Link Aggregation

**Increased Bandwidth:**
- Aggregate bandwidth of all member links
- 2×10 Gbps = 20 Gbps logical link
- Linear scaling up to 8 links (typically)

**Redundancy:**
- Automatic failover if link fails
- Sub-second convergence
- No STP reconvergence needed

**Load Balancing:**
- Traffic distributed across member links
- Various algorithms (src/dst MAC, IP, port)
- Optimizes link utilization

**Simplified STP:**
- Single logical link for STP
- Reduces STP topology complexity
- Faster convergence

**Cost Effective:**
- Cheaper than single high-speed link
- 2×10G often less expensive than 1×40G
- Easier upgrades (add links incrementally)

## 7.2 EtherChannel Protocols

### 7.2.1 Protocol Overview

| Protocol | Standard | Type | Description |
|----------|----------|------|-------------|
| **PAgP** | Cisco Proprietary | Dynamic | Port Aggregation Protocol |
| **LACP** | IEEE 802.3ad | Dynamic | Link Aggregation Control Protocol |
| **Static/On** | None | Manual | No negotiation |

### 7.2.2 PAgP (Port Aggregation Protocol)

**Cisco proprietary** - only works between Cisco switches.

**PAgP Modes:**
- **Auto**: Passive, waits for PAgP packets (default)
- **Desirable**: Active, initiates PAgP negotiation
- **On**: Forces channel without PAgP (static)

**Mode Combinations:**

| Local | Remote | Result |
|-------|--------|--------|
| Desirable | Desirable | Channel forms |
| Desirable | Auto | Channel forms |
| Auto | Auto | No channel |
| On | On | Channel forms (no negotiation) |
| On | Desirable/Auto | No channel |

**Configuration:**
```
Switch(config)# interface range gigabitethernet 0/1-2
Switch(config-if-range)# channel-group 1 mode desirable
Switch(config-if-range)# exit
```

### 7.2.3 LACP (Link Aggregation Control Protocol)

**Industry standard (IEEE 802.3ad)** - works with all vendors.

**LACP Modes:**
- **Active**: Actively sends LACP packets
- **Passive**: Waits for LACP packets
- **On**: Forces channel without LACP (static)

**Mode Combinations:**

| Local | Remote | Result |
|-------|--------|--------|
| Active | Active | Channel forms |
| Active | Passive | Channel forms |
| Passive | Passive | No channel |
| On | On | Channel forms (no negotiation) |
| On | Active/Passive | No channel |

**Configuration:**
```
Switch(config)# interface range gigabitethernet 0/1-2
Switch(config-if-range)# channel-group 1 mode active
Switch(config-if-range)# exit
```

**Best Practice**: **Use LACP (mode active)** for multi-vendor compatibility and better error detection.

### 7.2.4 Static EtherChannel (Mode On)

**No negotiation protocol** - forces all links into channel.

**Characteristics:**
- No dynamic detection of misconfigurations
- No graceful degradation if links fail
- Both sides must be configured identically
- Faster initialization (no negotiation delay)

**Configuration:**
```
Switch(config)# interface range gigabitethernet 0/1-2
Switch(config-if-range)# channel-group 1 mode on
Switch(config-if-range)# exit
```

**Use Case**: Point-to-point links where both ends are under your control and protocol overhead is concern.

## 7.3 EtherChannel Configuration Requirements

### 7.3.1 Configuration Consistency Rules

All ports in an EtherChannel **must match** on these parameters:

**Physical Layer:**
- ✓ Same speed (all 1 Gbps or all 10 Gbps)
- ✓ Same duplex (all full-duplex)
- ✓ Same media type (copper or fiber)

**Layer 2 Configuration:**
- ✓ Same switchport mode (all access or all trunk)
- ✓ Same access VLAN (if access ports)
- ✓ Same native VLAN (if trunk ports)
- ✓ Same allowed VLANs (if trunk ports)
- ✓ Same STP settings
- ✓ Same VLAN trunking encapsulation

**Common Mistakes:**
```
! ❌ WRONG - Different speeds
interface gigabitethernet 0/1
 speed 1000
interface gigabitethernet 0/2
 speed 100
 channel-group 1 mode active  ← Will fail!

! ❌ WRONG - Different VLANs
interface gigabitethernet 0/1
 switchport access vlan 10
interface gigabitethernet 0/2
 switchport access vlan 20
 channel-group 1 mode active  ← Will fail!
```

### 7.3.2 EtherChannel Misconfig Detection

Cisco switches automatically detect misconfigurations:

```
Switch# show etherchannel summary
...
Po1     LACP      Gi0/1(D)    Gi0/2(D)
                  ↑ D = Down (misconfigured)

%EC-5-CANNOT_BUNDLE2: Gi0/2 is not compatible with Gi0/1 and will be suspended (vlan numbers do not match)
```

**EtherChannel Guard:**
```
Switch(config)# spanning-tree etherchannel guard misconfig
```
Automatically places misconfigured ports in err-disabled state.

## 7.4 Basic EtherChannel Configuration

### 7.4.1 Layer 2 EtherChannel (Switch-to-Switch)

**Scenario**: Trunk between two switches using 4×1 Gbps links

**Switch A Configuration:**
```
SwitchA(config)# interface range gigabitethernet 0/1-4
SwitchA(config-if-range)# description Link to SwitchB
SwitchA(config-if-range)# switchport mode trunk
SwitchA(config-if-range)# switchport trunk allowed vlan 10,20,30,99
SwitchA(config-if-range)# switchport trunk native vlan 99
SwitchA(config-if-range)# channel-group 1 mode active
SwitchA(config-if-range)# exit

Creating a port-channel interface Port-channel 1

! Configure the port-channel interface (optional but recommended)
SwitchA(config)# interface port-channel 1
SwitchA(config-if)# description Trunk to SwitchB
SwitchA(config-if)# switchport mode trunk
SwitchA(config-if)# switchport trunk allowed vlan 10,20,30,99
SwitchA(config-if)# switchport trunk native vlan 99
SwitchA(config-if)# exit
```

**Switch B Configuration:**
```
SwitchB(config)# interface range gigabitethernet 0/1-4
SwitchB(config-if-range)# description Link to SwitchA
SwitchB(config-if-range)# switchport mode trunk
SwitchB(config-if-range)# switchport trunk allowed vlan 10,20,30,99
SwitchB(config-if-range)# switchport trunk native vlan 99
SwitchB(config-if-range)# channel-group 1 mode active
SwitchB(config-if-range)# exit

SwitchB(config)# interface port-channel 1
SwitchB(config-if)# description Trunk to SwitchA
SwitchB(config-if)# switchport mode trunk
SwitchB(config-if)# switchport trunk allowed vlan 10,20,30,99
SwitchB(config-if)# switchport trunk native vlan 99
SwitchB(config-if)# exit
```

**Verification:**
```
SwitchA# show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)       LACP        Gi0/1(P)    Gi0/2(P)    Gi0/3(P)    Gi0/4(P)
                                  ↑ P = Bundled and active
```

### 7.4.2 Layer 3 EtherChannel (Routed Port Channel)

**Scenario**: Routed link between Layer 3 switches or routers

**Switch A Configuration:**
```
SwitchA(config)# interface range gigabitethernet 0/1-2
SwitchA(config-if-range)# no switchport
SwitchA(config-if-range)# no ip address
SwitchA(config-if-range)# channel-group 1 mode active
SwitchA(config-if-range)# exit

SwitchA(config)# interface port-channel 1
SwitchA(config-if)# description L3 Link to SwitchB
SwitchA(config-if)# ip address 10.1.1.1 255.255.255.252
SwitchA(config-if)# no shutdown
SwitchA(config-if)# exit
```

**Switch B Configuration:**
```
SwitchB(config)# interface range gigabitethernet 0/1-2
SwitchB(config-if-range)# no switchport
SwitchB(config-if-range)# no ip address
SwitchB(config-if-range)# channel-group 1 mode active
SwitchB(config-if-range)# exit

SwitchB(config)# interface port-channel 1
SwitchB(config-if)# description L3 Link to SwitchA
SwitchB(config-if)# ip address 10.1.1.2 255.255.255.252
SwitchB(config-if)# no shutdown
SwitchB(config-if)# exit
```

**Verification:**
```
SwitchA# show ip interface brief | include Port-channel
Port-channel1          10.1.1.1        YES manual up                    up

SwitchA# ping 10.1.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/4 ms
```

## 7.5 LACP Advanced Features

### 7.5.1 LACP Priority

LACP can have more physical links than the maximum bundle size (typically 8 active + 8 standby).

**System Priority:**
- Determines which switch controls link selection
- Lower priority = higher preference
- Default: 32768

**Port Priority:**
- Determines which ports are active vs standby
- Lower priority = active
- Default: 32768

**Configuration:**
```
! Set system priority (globally)
Switch(config)# lacp system-priority 100

! Set port priority (per interface)
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# lacp port-priority 100
Switch(config-if)# exit

Switch(config)# interface gigabitethernet 0/2
Switch(config-if)# lacp port-priority 200
Switch(config-if)# exit
```

**Hot Standby Example:**
```
! Configure 10 links, but only 8 active
Switch(config)# interface range gigabitethernet 0/1-10
Switch(config-if-range)# channel-group 1 mode active
Switch(config-if-range)# exit

! Set priority so Gi0/1-8 are active
Switch(config)# interface range gigabitethernet 0/1-8
Switch(config-if-range)# lacp port-priority 100
Switch(config-if-range)# exit

Switch(config)# interface range gigabitethernet 0/9-10
Switch(config-if-range)# lacp port-priority 200
Switch(config-if-range)# exit
```

**Verification:**
```
Switch# show etherchannel summary
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)       LACP        Gi0/1(P)    Gi0/2(P)    Gi0/3(P)    Gi0/4(P)
                                 Gi0/5(P)    Gi0/6(P)    Gi0/7(P)    Gi0/8(P)
                                 Gi0/9(H)    Gi0/10(H)
                                          ↑ H = Hot-standby
```

### 7.5.2 LACP Fast Rate

Increases LACP PDU transmission rate for faster failure detection.

**Default**: 30 seconds (slow)
**Fast Rate**: 1 second

**Configuration:**
```
Switch(config)# interface range gigabitethernet 0/1-4
Switch(config-if-range)# lacp rate fast
Switch(config-if-range)# exit
```

**Use Case**: Critical links requiring sub-second failover

## 7.6 Load Balancing Methods

### 7.6.1 Understanding Load Balancing

EtherChannel uses **hashing algorithms** to distribute traffic across member links.

**Key Concept**: Traffic from same flow always uses same physical link (maintains packet ordering).

**Flow Definition**: Based on hash of packet headers
- Source/Destination MAC
- Source/Destination IP
- Source/Destination TCP/UDP port

### 7.6.2 Load Balancing Algorithms

**Available Methods:**

| Method | Hash Based On | Use Case |
|--------|---------------|----------|
| `src-mac` | Source MAC | Many clients, few servers |
| `dst-mac` | Destination MAC | Few clients, many servers |
| `src-dst-mac` | Src + Dst MAC | Balanced client/server |
| `src-ip` | Source IP | Many clients, few servers |
| `dst-ip` | Destination IP | Few clients, many servers |
| `src-dst-ip` | Src + Dst IP | **Most common** |
| `src-port` | TCP/UDP src port | Application-specific |
| `dst-port` | TCP/UDP dst port | Multiple services |
| `src-dst-port` | Src + Dst ports | Granular distribution |

**Configuration:**
```
Switch(config)# port-channel load-balance src-dst-ip
```

**View Current Method:**
```
Switch# show etherchannel load-balance
EtherChannel Load-Balancing Configuration:
        src-dst-ip

EtherChannel Load-Balancing Addresses Used Per-Protocol:
Non-IP: Source XOR Destination MAC address
  IPv4: Source XOR Destination IP address
  IPv6: Source XOR Destination IP address
```

### 7.6.3 Choosing the Right Algorithm

**Scenario 1: Many Clients → Few Servers**
```
[100 PCs] → [EtherChannel] → [3 Servers]

Best: src-ip or src-mac
- Source varies greatly (100 clients)
- Destination varies little (3 servers)
- Good distribution based on source
```

**Scenario 2: Few Clients → Many Servers**
```
[3 Core Switches] → [EtherChannel] → [50 Servers]

Best: dst-ip or dst-mac
- Source varies little (3 switches)
- Destination varies greatly (50 servers)
- Good distribution based on destination
```

**Scenario 3: Balanced Traffic**
```
[Switch A] ←→ [EtherChannel] ←→ [Switch B]

Best: src-dst-ip (default recommendation)
- Both source and destination vary
- Most balanced distribution
```

**Scenario 4: Application-Specific**
```
[Web Farm] → [EtherChannel] → [Database Cluster]

Best: src-dst-port
- Multiple connections between same IPs
- Different applications/services
- Port-based distribution
```

### 7.6.4 Testing Load Distribution

**Generate Test Traffic:**
```
! From different source IPs
Switch# test etherchannel load-balance interface port-channel 1 ip 10.1.1.1 10.2.2.1
Would select Gi0/1 of Po1

Switch# test etherchannel load-balance interface port-channel 1 ip 10.1.1.2 10.2.2.1
Would select Gi0/2 of Po1

Switch# test etherchannel load-balance interface port-channel 1 ip 10.1.1.3 10.2.2.1
Would select Gi0/1 of Po1

Switch# test etherchannel load-balance interface port-channel 1 ip 10.1.1.4 10.2.2.1
Would select Gi0/3 of Po1
```

**Monitor Interface Counters:**
```
Switch# show interface port-channel 1 | include rate
  30 second input rate 15000 bits/sec, 25 packets/sec
  30 second output rate 18000 bits/sec, 30 packets/sec

Switch# show interface gigabitethernet 0/1 | include rate
  30 second input rate 5000 bits/sec, 8 packets/sec
  30 second output rate 6000 bits/sec, 10 packets/sec

Switch# show interface gigabitethernet 0/2 | include rate
  30 second input rate 5000 bits/sec, 9 packets/sec
  30 second output rate 6000 bits/sec, 10 packets/sec

Switch# show interface gigabitethernet 0/3 | include rate
  30 second input rate 5000 bits/sec, 8 packets/sec
  30 second output rate 6000 bits/sec, 10 packets/sec
```

**Analyze Traffic Distribution:**
```
Switch# show etherchannel 1 port-channel
                Port-channels in the group:
                ---------------------------

Port-channel: Po1    (Primary Aggregator)
------------

Age of the Port-channel   = 2d:20h:15m:44s
Logical slot/port   = 16/1          Number of ports = 4
HotStandBy port = null
Port state          = Port-channel Ag-Inuse
Protocol            =   LACP
Port security       = Disabled

Ports in the Port-channel:

Index   Load   Port     EC state        No of bits
------+------+------+------------------+-----------
  0     00     Gi0/1    Active             4
  0     00     Gi0/2    Active             4
  0     00     Gi0/3    Active             4
  0     00     Gi0/4    Active             4

Time since last port bundled:    0d:00h:12m:45s    Gi0/4
```

## 7.7 EtherChannel Verification

### 7.7.1 Essential Show Commands

**Summary View:**
```
Switch# show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator

Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)       LACP        Gi0/1(P)    Gi0/2(P)    Gi0/3(P)    Gi0/4(P)
2      Po2(SU)       PAgP        Gi0/5(P)    Gi0/6(P)
```

**Detailed Information:**
```
Switch# show etherchannel 1 detail
Group state = L2
Ports: 4   Maxports = 8
Port-channels: 1 Max Port-channels = 1
Protocol:   LACP
Minimum Links: 0

                Ports in the group:
                -------------------
Port: Gi0/1
------------

Port state    = Up Mstr Assoc In-Bndl
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/1     SA      bndl      32768         0x1       0x1     0x102       0x3D
...
```

**Port-Channel Interface Status:**
```
Switch# show interface port-channel 1
Port-channel1 is up, line protocol is up (connected)
  Hardware is EtherChannel, address is 0024.9773.8401 (bia 0024.9773.8401)
  MTU 1500 bytes, BW 4000000 Kbit/sec, DLY 10 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  Full-duplex, 1000Mb/s, link type is auto, media type is unknown
  Members in this channel: Gi0/1 Gi0/2 Gi0/3 Gi0/4
  ...
```

**LACP Neighbor Information:**
```
Switch# show lacp neighbor
Flags:  S - Device is requesting Slow LACPDUs
        F - Device is requesting Fast LACPDUs
        A - Device is in Active mode       P - Device is in Passive mode

Channel group 1 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/1     SA      32768     0024.9773.8500  29s    0x0    0x1    0x102   0x3D
Gi0/2     SA      32768     0024.9773.8500  28s    0x0    0x1    0x103   0x3D
Gi0/3     SA      32768     0024.9773.8500  11s    0x0    0x1    0x104   0x3D
Gi0/4     SA      32768     0024.9773.8500  19s    0x0    0x1    0x105   0x3D
```

**Traffic Counters:**
```
Switch# show etherchannel 1 traffic

Port-channel1
-------------
        Frames Tx      Frames Rx      Bytes Tx       Bytes Rx
        1234567         9876543        123456789      987654321

Member ports in Po1:
                  Frames Tx      Frames Rx      Bytes Tx       Bytes Rx
Gi0/1             308641         2469135        30864197       246913580
Gi0/2             308642         2469136        30864297       246913680
Gi0/3             308642         2469136        30864297       246913680
Gi0/4             308642         2469136        30864298       246913681
```

### 7.7.2 Debugging EtherChannel

```
Switch# debug etherchannel all
Switch# debug lacp all
Switch# debug pagp all

! Turn off debugging
Switch# undebug all
```

## 7.8 Troubleshooting EtherChannel

### 7.8.1 Issue: Channel Not Forming

**Symptoms:**
- Physical links up, but no Port-channel interface
- Ports showing as `(I)` = stand-alone

**Diagnosis:**
```
Switch# show etherchannel summary
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SD)       LACP        Gi0/1(I)    Gi0/2(I)
                     ↑ SD = Suspended          ↑ I = Stand-alone
```

**Common Causes:**

**1. Protocol Mismatch**
```
SwitchA: channel-group 1 mode active   (LACP)
SwitchB: channel-group 1 mode desirable (PAgP)
```
**Solution**: Use same protocol on both sides

**2. Mode Incompatibility**
```
SwitchA: channel-group 1 mode passive  (LACP)
SwitchB: channel-group 1 mode passive  (LACP)
```
**Solution**: At least one side must be active

**3. Configuration Mismatch**
```
Switch# show etherchannel summary
%EC-5-CANNOT_BUNDLE2: Gi0/2 is not compatible with Gi0/1 and will be suspended
(vlan numbers do not match)
```

**Check Consistency:**
```
Switch# show run interface gigabitethernet 0/1
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 channel-group 1 mode active

Switch# show run interface gigabitethernet 0/2
interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,40  ← DIFFERENT!
 channel-group 1 mode active
```

**Solution:**
```
Switch(config)# interface gigabitethernet 0/2
Switch(config-if)# switchport trunk allowed vlan 10,20,30
```

### 7.8.2 Issue: Err-Disabled Ports

**Symptoms:**
- Ports in err-disabled state
- "channel-misconfig" error message

**Diagnosis:**
```
Switch# show interfaces status err-disabled

Port      Name               Status       Reason               Err-disabled Vlans
Gi0/1                        err-disabled channel-misconfig

Switch# show etherchannel summary
%SPANTREE-2-CHNL_MISCFG: Detected loop due to etherchannel misconfiguration of Gi0/1
```

**Solution:**
```
! Enable EtherChannel guard
Switch(config)# spanning-tree etherchannel guard misconfig

! Recover ports
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# shutdown
Switch(config-if)# no shutdown
Switch(config-if)# exit

! Or enable auto-recovery
Switch(config)# errdisable recovery cause channel-misconfig
Switch(config)# errdisable recovery interval 300
```

### 7.8.3 Issue: Unequal Load Distribution

**Symptoms:**
- All traffic using one or two links
- Other links idle or low utilization

**Diagnosis:**
```
Switch# show etherchannel 1 traffic

Member ports in Po1:
                  Frames Tx      Frames Rx
Gi0/1             9876543        9876543    ← High traffic
Gi0/2             10             12         ← Idle
Gi0/3             8              9          ← Idle
Gi0/4             11             10         ← Idle
```

**Common Causes:**

**1. Traffic Patterns Don't Match Algorithm**
```
! Current: src-dst-mac
! Problem: Single client to single server (same MAC pair always)

Switch# show etherchannel load-balance
EtherChannel Load-Balancing Configuration:
        src-dst-mac  ← May not distribute well for your traffic
```

**Solution - Change Algorithm:**
```
Switch(config)# port-channel load-balance src-dst-ip
! Or try: src-dst-port for application-level distribution
```

**2. Testing Load Balance:**
```
! Verify distribution with different IPs
Switch# test etherchannel load-balance interface port-channel 1 ip 192.168.1.1 10.1.1.1
Would select Gi0/1 of Po1

Switch# test etherchannel load-balance interface port-channel 1 ip 192.168.1.2 10.1.1.1
Would select Gi0/2 of Po1
```

### 7.8.4 Issue: Member Link Flapping

**Symptoms:**
- EtherChannel up/down messages
- Port constantly being added/removed from bundle

**Diagnosis:**
```
Switch# show logging | include BUNDLE
%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to down
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
```

**Common Causes:**

**1. Physical Layer Issues**
```
Switch# show interface gigabitethernet 0/1
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     123 runts, 456 giants, 0 throttles
     789 input errors, 234 CRC, 0 frame, 0 overrun, 0 ignored
                      ↑ Physical errors
```

**Solution**: Check cables, transceivers, port errors

**2. Duplex Mismatch**
```
Switch# show interface gigabitethernet 0/1 | include duplex
  Full-duplex, 1000Mb/s, media type is 1000BaseT
  ↓ But on other end:
  Half-duplex, 1000Mb/s, media type is 1000BaseT  ← MISMATCH!
```

**Solution:**
```
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# duplex full
Switch(config-if)# speed 1000
```

### 7.8.5 Issue: LACP Not Negotiating

**Symptoms:**
- Ports configured for LACP but not bundling
- No LACP neighbor information

**Diagnosis:**
```
Switch# show lacp neighbor
                                ^
% Invalid input detected at '^' marker.
! No LACP neighbors detected

Switch# show etherchannel 1 detail | include Protocol
Protocol:   LACP
```

**Check LACP Transmission:**
```
Switch# debug lacp all
*Mar  1 00:01:23.456: LACP: Gi0/1 port is individually
*Mar  1 00:01:23.456: LACP: Gi0/1 Transmit LACPDU
! If not transmitting, check configuration

Switch# show run interface gigabitethernet 0/1
interface GigabitEthernet0/1
 channel-group 1 mode passive  ← Check if both sides passive!
```

**Solution:**
```
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# channel-group 1 mode active
```

## 7.9 EtherChannel Best Practices

### 7.9.1 Design Recommendations

**1. Use LACP (Not PAgP or Static)**
```
Switch(config-if-range)# channel-group 1 mode active
! Industry standard, better error detection, multi-vendor support
```

**2. Configure Port-Channel Interface Explicitly**
```
Switch(config)# interface port-channel 1
Switch(config-if)# description Trunk to Core Switch
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30
! Ensures consistent configuration even if member ports change
```

**3. Start with 2 Links, Scale as Needed**
- 2 links = redundancy + double bandwidth
- 4 links = good balance of capacity and complexity
- 8 links = maximum on most platforms

**4. Keep Link Counts Power of 2**
- 2, 4, 8 links optimize hash distribution
- Odd numbers (3, 5, 7) can cause uneven distribution

**5. Use Fast LACP Rate for Critical Links**
```
Switch(config-if-range)# lacp rate fast
! 1-second PDUs for faster failure detection
```

**6. Enable EtherChannel Guard**
```
Switch(config)# spanning-tree etherchannel guard misconfig
```

**7. Choose Appropriate Load-Balance Algorithm**
```
! Most versatile for modern networks:
Switch(config)# port-channel load-balance src-dst-ip
```

**8. Document EtherChannel Design**
```
! Port-channel 1: Core to Distribution Layer
!   - 4×10 Gbps links (Gi0/1-4 on both sides)
!   - LACP active mode
!   - Trunk carrying VLANs 10,20,30,99
!   - Load balance: src-dst-ip
```

### 7.9.2 Security Considerations

**1. Avoid Static/On Mode in Production**
- No error detection
- Silent failures possible
- Use LACP active instead

**2. Consistent Configuration on Both Sides**
```
! Use same channel-group number (not required but helpful)
! Use same LACP priority settings
! Use same load-balance algorithm
```

**3. Monitor for Misconfigurations**
```
Switch(config)# spanning-tree etherchannel guard misconfig
Switch(config)# errdisable recovery cause channel-misconfig
```

**4. Secure Unused Ports in EtherChannel Range**
```
! If using Gi0/1-4 for EtherChannel, secure Gi0/5-24
Switch(config)# interface range gigabitethernet 0/5-24
Switch(config-if-range)# shutdown
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 999
```

## 7.10 Real-World Scenarios

### 7.10.1 Scenario: High-Availability Data Center

**Requirements:**
- 20 Gbps bandwidth between core and aggregation
- Sub-second failover
- Load balancing for multiple servers

**Design:**
```
        [Core Switch]
             ║  ║
        (2×10 Gbps LACP)
             ║  ║
      [Aggregation Switch]
       ║  ║  ║  ║  ║  ║
    (6×10 Gbps to server racks)
```

**Core Switch Configuration:**
```
Core(config)# interface range tengigabitethernet 0/1-2
Core(config-if-range)# description LACP to Aggregation
Core(config-if-range)# switchport mode trunk
Core(config-if-range)# switchport trunk allowed vlan 10,20,30,100-110
Core(config-if-range)# channel-group 1 mode active
Core(config-if-range)# lacp rate fast
Core(config-if-range)# exit

Core(config)# interface port-channel 1
Core(config-if)# description Trunk to Aggregation Layer
Core(config-if)# switchport mode trunk
Core(config-if)# switchport trunk allowed vlan 10,20,30,100-110
Core(config-if)# exit

Core(config)# port-channel load-balance src-dst-ip
```

**Aggregation Switch Configuration:**
```
Agg(config)# interface range tengigabitethernet 0/1-2
Agg(config-if-range)# description LACP to Core
Agg(config-if-range)# switchport mode trunk
Agg(config-if-range)# switchport trunk allowed vlan 10,20,30,100-110
Agg(config-if-range)# channel-group 1 mode active
Agg(config-if-range)# lacp rate fast
Agg(config-if-range)# exit

! Downstream to servers (6 separate EtherChannels, each 2×10G)
Agg(config)# interface range tengigabitethernet 0/3-4
Agg(config-if-range)# description LACP to Rack1
Agg(config-if-range)# switchport mode trunk
Agg(config-if-range)# channel-group 2 mode active
Agg(config-if-range)# exit

! Repeat for Rack 2-6 with channel-groups 3-7
```

**Result:**
- 20 Gbps aggregate uplink capacity
- Fast failover with LACP fast rate
- Optimized load distribution for server traffic

### 7.10.2 Scenario: Campus Distribution Layer

**Requirements:**
- Redundant uplinks from access to distribution
- Cost-effective bandwidth increase
- Support for 500 users across multiple VLANs

**Design:**
```
    [Distribution A]---[Distribution B]
          ║  ║              ║  ║
       (4×1G LACP)      (4×1G LACP)
          ║  ║              ║  ║
         [Access Switch 1]
              ║  ║
         (User ports)
```

**Access Switch Configuration:**
```
Access(config)# interface range gigabitethernet 0/23-24
Access(config-if-range)# description Uplink to Dist-A
Access(config-if-range)# switchport mode trunk
Access(config-if-range)# switchport trunk allowed vlan 10,20,30,99
Access(config-if-range)# channel-group 1 mode active
Access(config-if-range)# exit

Access(config)# interface range gigabitethernet 0/21-22
Access(config-if-range)# description Uplink to Dist-B
Access(config-if-range)# switchport mode trunk
Access(config-if-range)# switchport trunk allowed vlan 10,20,30,99
Access(config-if-range)# channel-group 2 mode active
Access(config-if-range)# exit

! User access ports (Gi0/1-20)
Access(config)# interface range gigabitethernet 0/1-20
Access(config-if-range)# switchport mode access
Access(config-if-range)# spanning-tree portfast
Access(config-if-range)# spanning-tree bpduguard enable
Access(config-if-range)# exit
```

**Distribution Switches Configuration:**
```
Dist-A(config)# interface range gigabitethernet 0/1-2
Dist-A(config-if-range)# description Downlink to Access-1
Dist-A(config-if-range)# switchport mode trunk
Dist-A(config-if-range)# switchport trunk allowed vlan 10,20,30,99
Dist-A(config-if-range)# spanning-tree guard root
Dist-A(config-if-range)# channel-group 10 mode active
Dist-A(config-if-range)# exit
```

## Summary

Link Aggregation (EtherChannel) is essential for high-bandwidth, redundant switched networks:

- **Combines multiple links** into single logical link
- **LACP (mode active)** recommended over PAgP or static
- **Requirements**: Matching speed, duplex, VLAN configuration
- **Load balancing**: Choose algorithm based on traffic patterns (src-dst-ip most common)
- **Advanced features**: Hot standby, fast rate, priority control
- **Best practices**: Use LACP, configure port-channel interface, enable fast rate for critical links

In the next chapter, we'll explore Quality of Service (QoS), which prioritizes critical traffic such as voice and video across the network.
