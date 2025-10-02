# Chapter 8: Quality of Service (QoS)

## 8.1 Introduction to Quality of Service

Quality of Service (QoS) is the set of technologies used to manage network resources and provide preferential treatment to specific types of traffic. In modern converged networks carrying data, voice, and video, QoS ensures that critical applications receive the bandwidth and low latency they require.

### 8.1.1 Why QoS is Necessary

**The Problem:**
Without QoS, all traffic is treated equally (best-effort). Network congestion affects all applications equally, causing:

- **VoIP calls**: Choppy audio, dropouts, delays
- **Video conferencing**: Pixelation, freezing, lip-sync issues
- **Real-time applications**: Unacceptable performance degradation
- **Business-critical apps**: Compete equally with YouTube and Netflix

**Real-World Example:**
```
Network Link: 100 Mbps
Current Traffic:
  - Email downloads: 30 Mbps
  - File transfers: 40 Mbps
  - Video streaming: 25 Mbps
  - VoIP call: 100 Kbps (0.1 Mbps)

Total: 95.1 Mbps → VoIP works fine

Suddenly:
  - Large backup starts: +50 Mbps
Total: 145.1 Mbps → Link saturated!
Result: VoIP packets delayed/dropped → Call quality degrades
```

**With QoS:**
VoIP traffic prioritized → guaranteed bandwidth → call quality maintained even during congestion.

### 8.1.2 QoS Benefits

**Business Benefits:**
- Predictable application performance
- Better user experience
- Support for real-time applications (voice, video)
- Maximized ROI on network infrastructure
- Compliance with SLAs

**Technical Benefits:**
- Efficient bandwidth utilization
- Congestion management
- Packet loss reduction for critical traffic
- Latency and jitter control
- Network scalability

## 8.2 QoS Fundamentals

### 8.2.1 Key QoS Metrics

**Bandwidth:**
- Measured in bits per second (bps, Kbps, Mbps, Gbps)
- Amount of data that can be transmitted per unit time
- VoIP requires: 64-128 Kbps per call
- Video conferencing: 384 Kbps - 2 Mbps per stream

**Latency (Delay):**
- One-way time for packet to travel from source to destination
- VoIP tolerance: < 150 ms one-way
- Interactive applications: < 100 ms
- Causes: Propagation delay, serialization, processing, queuing

**Jitter (Delay Variation):**
- Variation in packet arrival times
- VoIP tolerance: < 30 ms
- Causes voice quality degradation if excessive
- Solved with jitter buffers

**Packet Loss:**
- Percentage of packets dropped during transmission
- VoIP tolerance: < 1%
- Data applications: Generally tolerant (TCP retransmits)
- Real-time apps: Very sensitive (no retransmission time)

### 8.2.2 Application Requirements

| Application | Bandwidth | Latency | Jitter | Loss | Priority |
|-------------|-----------|---------|--------|------|----------|
| Voice (VoIP) | 64-128 Kbps | < 150 ms | < 30 ms | < 1% | Critical |
| Video Conferencing | 384K-2M | < 150 ms | < 30 ms | < 1% | Critical |
| Streaming Video | 1-10 Mbps | < 500 ms | < 50 ms | < 5% | High |
| Transactional Data | Variable | < 200 ms | N/A | < 2% | Medium |
| Email | Low | < 2 sec | N/A | < 5% | Low |
| File Transfer | Variable | Best-effort | N/A | Tolerant | Low |
| Web Browsing | Variable | < 1 sec | N/A | < 5% | Medium |

### 8.2.3 QoS Models

**Best-Effort (No QoS):**
- First-In-First-Out (FIFO) queuing
- No differentiation between traffic types
- Simplest but least effective

**Integrated Services (IntServ):**
- Per-flow resource reservation (RSVP)
- Guaranteed QoS for each flow
- Does not scale (state information per flow)
- Rarely used in modern networks

**Differentiated Services (DiffServ):**
- **Most common model today**
- Classifies traffic into classes
- Each class receives specific treatment
- Scalable (no per-flow state)
- Uses DSCP markings in IP header

## 8.3 QoS Mechanisms

QoS implementation involves three key steps:

### 8.3.1 Classification and Marking

**Classification**: Identifying traffic types
**Marking**: Labeling packets for QoS treatment

**Classification Methods:**

**1. Layer 2 - Class of Service (CoS)**
- 3-bit field in 802.1Q VLAN tag
- Values: 0-7 (8 classes)
- Only on trunk links with VLAN tags
- Lost when crossing Layer 3 boundaries

**802.1Q Frame with CoS:**
```
[Dest MAC | Src MAC | 802.1Q Tag | Type | Data | FCS]
                      |
                      +-- TPID | PCP | DEI | VID
                        0x8100   CoS   Drop  VLAN
                                 (3b)  (1b)  (12b)
```

**Standard CoS Values:**
| CoS | Priority | Traffic Type |
|-----|----------|--------------|
| 0 | Best Effort | Default traffic |
| 1 | Background | Bulk transfers |
| 2 | Spare | - |
| 3 | Excellent Effort | Business-critical data |
| 4 | Controlled Load | Streaming video |
| 5 | Video (< 100ms latency) | Interactive video |
| 6 | Voice (< 10ms latency) | VoIP, critical |
| 7 | Network Control | Routing protocols |

**2. Layer 3 - IP Precedence (Legacy)**
- 3-bit field in IPv4 ToS byte
- Values: 0-7
- Largely replaced by DSCP
- Still used in some legacy systems

**3. Layer 3 - DSCP (Differentiated Services Code Point)**
- **Modern standard**
- 6-bit field in IPv4/IPv6 header
- Values: 0-63 (64 classes)
- Survives Layer 3 routing

**IPv4 Header with DSCP:**
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |   DSCP  | ECN |         Total Length          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                  ↑ 6 bits
```

**Standard DSCP Values (Per-Hop Behaviors):**

| DSCP | Decimal | Binary | Name | PHB | Use Case |
|------|---------|--------|------|-----|----------|
| CS0 | 0 | 000000 | Default | BE | Best-effort traffic |
| CS1 | 8 | 001000 | Scavenger | - | Bulk/background |
| AF11 | 10 | 001010 | Assured Forwarding | AF1 | High-priority data |
| AF21 | 18 | 010010 | Assured Forwarding | AF2 | Medium data |
| AF31 | 26 | 011010 | Assured Forwarding | AF3 | Streaming video |
| AF41 | 34 | 100010 | Assured Forwarding | AF4 | Interactive video |
| EF | 46 | 101110 | Expedited Forwarding | EF | **Voice (VoIP)** |
| CS6 | 48 | 110000 | Network Control | - | Routing protocols |
| CS7 | 56 | 111000 | Network Control | - | Reserved |

**Assured Forwarding (AFxy):**
- **x** = Class (1-4): 1=lowest priority, 4=highest
- **y** = Drop Precedence (1-3): 1=low drop, 3=high drop
- Example: AF11, AF12, AF13 (same class, different drop probabilities)

**4. Layer 4 - Port Numbers**
- TCP/UDP port-based classification
- HTTP: 80, HTTPS: 443, VoIP SIP: 5060

**5. Layer 7 - Deep Packet Inspection**
- Application-layer analysis
- NBAR (Network-Based Application Recognition)
- Can identify applications regardless of port

### 8.3.2 Trust Boundaries

**Trust Boundary**: Where network trusts QoS markings from devices

**Typical Trust Scenarios:**

```
[PC] → [IP Phone] → [Access Switch] → [Core Switch]
 ↑                   ↑                  ↑
 Untrusted           Trust Boundary     Trusted
```

**Best Practice: Trust at Access Edge**
- Trust IP phones (they mark VoIP traffic correctly)
- Remark/classify PC traffic at switch
- Extend trust to network infrastructure

**Configuration:**
```
! Trust IP Phone CoS markings
Switch(config-if)# mls qos trust cos

! Trust DSCP from upstream switch
Switch(config-if)# mls qos trust dscp

! Don't trust (reclassify all traffic)
Switch(config-if)# no mls qos trust
```

## 8.4 Queuing Strategies

Queuing determines how packets are serviced when congestion occurs.

### 8.4.1 FIFO (First-In-First-Out)

**Simplest queuing:**
- Single queue, no prioritization
- Packets serviced in arrival order
- **Not suitable for QoS**

```
[Packet 1] → [Packet 2] → [Packet 3] → → → Output
   (VoIP)      (Email)     (Video)
```

### 8.4.2 Priority Queuing (PQ)

**Strict priority:**
- Multiple queues
- High-priority queues always serviced first
- **Risk of starvation** for low-priority traffic

```
Queue 1 (High):   [VoIP] [VoIP] → → →
Queue 2 (Medium): [Video] [Video]
Queue 3 (Low):    [Email] [Bulk] [Bulk]  ← May never be serviced!
                                            ↓
                                        Output
```

**Use Case**: Voice traffic only (small, constant rate)

### 8.4.3 Weighted Fair Queuing (WFQ)

**Flow-based fairness:**
- Automatically creates queues per flow
- Low-bandwidth flows get priority
- High-bandwidth flows share remaining capacity
- No configuration needed (default on serial < 2 Mbps)

**Limitation**: Not suitable for high-speed LAN interfaces

### 8.4.4 Class-Based Weighted Fair Queuing (CBWFQ)

**Bandwidth guarantees per class:**
- Define traffic classes
- Assign minimum bandwidth to each class
- Unused bandwidth shared among classes

**Configuration:**
```
! Define classes
Switch(config)# class-map match-all VOICE
Switch(config-cmap)# match ip dscp ef
Switch(config-cmap)# exit

Switch(config)# class-map match-all VIDEO
Switch(config-cmap)# match ip dscp af41
Switch(config-cmap)# exit

! Define policy
Switch(config)# policy-map WAN-POLICY
Switch(config-pmap)# class VOICE
Switch(config-pmap-c)# bandwidth percent 30
Switch(config-pmap-c)# class VIDEO
Switch(config-pmap-c)# bandwidth percent 40
Switch(config-pmap-c)# class class-default
Switch(config-pmap-c)# fair-queue
Switch(config-pmap-c)# exit

! Apply to interface
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# service-policy output WAN-POLICY
```

### 8.4.5 Low Latency Queuing (LLQ)

**Priority + CBWFQ combination:**
- Priority queue for delay-sensitive traffic (voice)
- CBWFQ for remaining traffic classes
- **Industry standard for VoIP**

**Configuration:**
```
Switch(config)# policy-map WAN-LLQ
Switch(config-pmap)# class VOICE
Switch(config-pmap-c)# priority percent 20
                        ↑ Strict priority + bandwidth limit
Switch(config-pmap-c)# class VIDEO
Switch(config-pmap-c)# bandwidth percent 30
Switch(config-pmap-c)# class BUSINESS-DATA
Switch(config-pmap-c)# bandwidth percent 30
Switch(config-pmap-c)# class class-default
Switch(config-pmap-c)# fair-queue
Switch(config-pmap-c)# exit
```

**Priority Percent**: Guarantees bandwidth and strict priority, but limits to prevent starvation

## 8.5 Congestion Avoidance

### 8.5.1 Tail Drop (Default Behavior)

**How it works:**
- Queue fills to maximum
- All new packets dropped until space available
- **Problem**: TCP global synchronization

```
Queue: [============================] 100% Full
New packets: DROPPED ✗ ✗ ✗ ✗
```

**TCP Global Synchronization:**
1. Queue fills → all TCP flows drop packets
2. All TCP flows back off simultaneously
3. Link underutilized
4. All flows ramp up again
5. Queue fills → repeat

### 8.5.2 Random Early Detection (RED)

**Proactive dropping:**
- Drops packets **before** queue is full
- Probability increases as queue fills
- Prevents global synchronization

```
Drop Probability
      ^
100%  |                    ___________
      |                   /
      |                  /
      |                 /
  0%  |________________/
      +----------------+-------+------>
      0              Min     Max      Queue Depth
                     Threshold
```

### 8.5.3 Weighted Random Early Detection (WRED)

**Drop based on IP Precedence/DSCP:**
- Different drop profiles per traffic class
- High-priority traffic dropped less aggressively
- **Standard congestion avoidance mechanism**

**Configuration:**
```
Switch(config)# policy-map WAN-POLICY
Switch(config-pmap)# class BUSINESS-DATA
Switch(config-pmap-c)# bandwidth percent 40
Switch(config-pmap-c)# random-detect dscp-based
Switch(config-pmap-c)# exit
```

**Drop Profiles by DSCP:**
```
EF (Voice):      Drop at 90% queue depth (preserve VoIP)
AF41 (Video):    Drop at 70% queue depth
AF21 (Data):     Drop at 50% queue depth
Default:         Drop at 30% queue depth (most aggressive)
```

## 8.6 Traffic Shaping and Policing

Both limit traffic rate, but behave differently.

### 8.6.1 Traffic Policing

**Enforces rate limit by dropping/remarking excess traffic:**
- Excess traffic **immediately** dropped or remarked
- Can cause TCP retransmissions
- Used on ingress or egress
- Efficient (no buffering)

**Configuration:**
```
Switch(config)# policy-map POLICE-100M
Switch(config-pmap)# class class-default
Switch(config-pmap-c)# police 100000000 2000000 exceed-action drop
                       ↑ Rate (bps)  ↑ Burst   ↑ Action
Switch(config-pmap-c)# exit

Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# service-policy input POLICE-100M
```

**Actions for Excess Traffic:**
- `drop`: Discard packets
- `transmit`: Send anyway (defeats purpose)
- `set-dscp-transmit`: Remark to lower priority

### 8.6.2 Traffic Shaping

**Buffers and smooths traffic to conform to rate:**
- Excess traffic **buffered** and delayed
- Smooths bursty traffic
- Better for TCP (no drops)
- Only used on egress (requires buffering)

**Configuration:**
```
Switch(config)# policy-map SHAPE-50M
Switch(config-pmap)# class class-default
Switch(config-pmap-c)# shape average 50000000
                                    ↑ Rate (bps)
Switch(config-pmap-c)# exit

Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# service-policy output SHAPE-50M
```

**When to Use:**

| Scenario | Use | Reason |
|----------|-----|--------|
| Provider rate limit (hard) | Policing | Immediate enforcement, no buffering |
| WAN link bandwidth limit | Shaping | Smooth traffic, better TCP performance |
| Prevent bursts | Shaping | Buffer and smooth |
| Punish violators | Policing | Drop or remark excess |

## 8.7 QoS Configuration on Cisco Switches

### 8.7.1 Enable QoS Globally

**Catalyst Switches:**
```
Switch(config)# mls qos
```

**Verification:**
```
Switch# show mls qos
QoS is enabled
QoS ip packet dscp rewrite is enabled
```

### 8.7.2 Configure Trust Boundary

**Trust IP Phone CoS:**
```
Switch(config)# interface gigabitethernet 0/5
Switch(config-if)# description IP Phone + PC
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# switchport voice vlan 100
Switch(config-if)# mls qos trust cos
Switch(config-if)# exit
```

**Trust DSCP (Inter-Switch Links):**
```
Switch(config)# interface gigabitethernet 0/24
Switch(config-if)# description Trunk to Core
Switch(config-if)# switchport mode trunk
Switch(config-if)# mls qos trust dscp
Switch(config-if)# exit
```

**Don't Trust (Classify at Switch):**
```
Switch(config)# interface gigabitethernet 0/10
Switch(config-if)# description Server
Switch(config-if)# no mls qos trust
Switch(config-if)# exit
```

### 8.7.3 Classification and Marking

**Classify by Access-List:**
```
Switch(config)# access-list 100 permit udp any any range 16384 32767
                                         ↑ RTP port range (voice)

Switch(config)# class-map VOICE-CLASS
Switch(config-cmap)# match access-group 100
Switch(config-cmap)# exit

Switch(config)# policy-map MARK-VOICE
Switch(config-pmap)# class VOICE-CLASS
Switch(config-pmap-c)# set dscp ef
Switch(config-pmap-c)# exit

Switch(config)# interface gigabitethernet 0/10
Switch(config-if)# service-policy input MARK-VOICE
Switch(config-if)# exit
```

**Classify by CoS (from IP Phone):**
```
Switch(config)# class-map VOICE-COS
Switch(config-cmap)# match cos 5
Switch(config-cmap)# exit

Switch(config)# policy-map COS-TO-DSCP
Switch(config-pmap)# class VOICE-COS
Switch(config-pmap-c)# set dscp ef
Switch(config-pmap-c)# exit
```

### 8.7.4 Egress Queuing

**Configure Queue Weights:**
```
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# mls qos trust dscp
Switch(config-if)# wrr-queue bandwidth percent 10 20 30 40
                   ↑ 4 queues with weighted round-robin
Switch(config-if)# exit
```

**Priority Queue for Voice:**
```
Switch(config-if)# priority-queue out
Switch(config-if)# mls qos trust dscp
```

## 8.8 VoIP Prioritization Example

Complete configuration for VoIP QoS in a typical office.

### 8.8.1 Network Topology

```
[IP Phones] → [Access Switch] → [Core Switch] → [Router] → [WAN/Internet]
   VLAN 100      Trust CoS        Trust DSCP      LLQ
```

### 8.8.2 Access Switch Configuration

```
! Enable QoS globally
Switch(config)# mls qos

! Configure CoS-to-DSCP mapping
Switch(config)# mls qos map cos-dscp 0 8 16 24 32 46 48 56
                                            ↑ CoS 5 → DSCP 46 (EF)

! Configure voice VLAN
Switch(config)# vlan 100
Switch(config-vlan)# name VOICE
Switch(config-vlan)# exit

! Access ports with IP phones
Switch(config)# interface range gigabitethernet 0/1-20
Switch(config-if-range)# description User ports with IP Phones
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# switchport voice vlan 100
Switch(config-if-range)# mls qos trust cos
                          ↑ Trust phone's CoS markings
Switch(config-if-range)# spanning-tree portfast
Switch(config-if-range)# exit

! Uplink to core (trust DSCP)
Switch(config)# interface gigabitethernet 0/24
Switch(config-if)# description Trunk to Core
Switch(config-if)# switchport mode trunk
Switch(config-if)# mls qos trust dscp
Switch(config-if)# exit
```

### 8.8.3 Core Switch Configuration

```
! Enable QoS
CoreSwitch(config)# mls qos

! Trust DSCP on all inter-switch links
CoreSwitch(config)# interface range gigabitethernet 0/1-24
CoreSwitch(config-if-range)# mls qos trust dscp
CoreSwitch(config-if-range)# exit
```

### 8.8.4 WAN Router Configuration

```
! Define traffic classes
Router(config)# class-map match-all VOICE
Router(config-cmap)# match ip dscp ef
Router(config-cmap)# exit

Router(config)# class-map match-all SIGNALING
Router(config-cmap)# match ip dscp cs3
Router(config-cmap)# exit

Router(config)# class-map match-all VIDEO
Router(config-cmap)# match ip dscp af41
Router(config-cmap)# exit

Router(config)# class-map match-all BUSINESS-DATA
Router(config-cmap)# match ip dscp af21
Router(config-cmap)# exit

! Create LLQ policy
Router(config)# policy-map WAN-EDGE
Router(config-pmap)# class VOICE
Router(config-pmap-c)# priority percent 20
                        ↑ Strict priority + 20% guaranteed
Router(config-pmap-c)# class SIGNALING
Router(config-pmap-c)# bandwidth percent 5
Router(config-pmap-c)# class VIDEO
Router(config-pmap-c)# bandwidth percent 30
Router(config-pmap-c)# class BUSINESS-DATA
Router(config-pmap-c)# bandwidth percent 25
Router(config-pmap-c)# random-detect dscp-based
Router(config-pmap-c)# class class-default
Router(config-pmap-c)# bandwidth percent 20
Router(config-pmap-c)# random-detect dscp-based
Router(config-pmap-c)# exit

! Apply to WAN interface
Router(config)# interface gigabitethernet 0/0
Router(config-if)# ip address 203.0.113.1 255.255.255.252
Router(config-if)# service-policy output WAN-EDGE
Router(config-if)# exit
```

**Result:**
- Voice traffic prioritized with guaranteed 20% bandwidth
- Call signaling protected
- Video gets 30% minimum bandwidth
- Business data and default traffic share remaining capacity
- WRED prevents congestion

## 8.9 QoS Verification and Monitoring

### 8.9.1 Verification Commands

**QoS Status:**
```
Switch# show mls qos
QoS is enabled
QoS ip packet dscp rewrite is enabled

Switch# show mls qos interface gigabitethernet 0/5
GigabitEthernet0/5
trust state: trust cos
trust mode: trust cos
trust enabled flag: ena
COS override: dis
default COS: 0
DSCP Mutation Map: Default DSCP Mutation Map
Trust device: cisco-phone
```

**Queue Statistics:**
```
Switch# show mls qos interface gigabitethernet 0/1 queuing

GigabitEthernet0/1
Egress expedite queue:   ena
Queue Id    Scheduling  Num of thresholds
---------------------------------------
   01       WRR            01
   02       WRR            01
   03       WRR            01
   04       WRR            01

WRR bandwidth ratios:    10[queue 1] 20[queue 2] 30[queue 3] 40[queue 4]
```

**Policy Map Statistics (Router):**
```
Router# show policy-map interface gigabitethernet 0/0

GigabitEthernet0/0

  Service-policy output: WAN-EDGE

    Class-map: VOICE (match-all)
      25000 packets, 2500000 bytes
      5 minute offered rate 128000 bps, drop rate 0 bps
      Match: ip dscp ef (46)
      Queueing
        Strict Priority
        Output Queue: Conversation 264
        Bandwidth 20 (%)
        (pkts matched/bytes matched) 25000/2500000
        (total drops/bytes drops) 0/0

    Class-map: VIDEO (match-all)
      150000 packets, 180000000 bytes
      5 minute offered rate 2400000 bps, drop rate 0 bps
      Match: ip dscp af41 (34)
      Queueing
        Output Queue: Conversation 265
        Bandwidth 30 (%)
        (pkts matched/bytes matched) 150000/180000000
        (total drops/bytes drops) 12/14400
```

**CoS/DSCP Mapping:**
```
Switch# show mls qos maps cos-dscp
   Cos-dscp map:
         cos:   0  1  2  3  4  5  6  7
      --------------------------------
       dscp:   0  8 16 24 32 46 48 56
```

### 8.9.2 Monitoring Tools

**Real-Time Monitoring:**
```
! Watch drops in real-time
Router# show policy-map interface gi0/0 | include drop
      5 minute offered rate 128000 bps, drop rate 0 bps
      (total drops/bytes drops) 0/0

! Monitor queue depth
Switch# show queueing interface gigabitethernet 0/1
```

**IP SLA for VoIP Quality:**
```
Router(config)# ip sla 1
Router(config-ip-sla)# udp-jitter 10.1.1.10 16384 codec g711alaw
Router(config-ip-sla-jitter)# frequency 60
Router(config-ip-sla-jitter)# exit
Router(config)# ip sla schedule 1 life forever start-time now

Router# show ip sla statistics 1
Round Trip Time (RTT) for       Index 1
        Latest RTT: 45 milliseconds
Latest operation start time: *12:34:56.789 UTC Fri Mar 1 2024
Latest operation return code: OK
RTT Values:
        Number Of RTT: 10              RTT Min/Avg/Max: 40/45/52 milliseconds
Latency one-way time:
        Number of Latency one-way Samples: 10
        Source to Destination Latency one way Min/Avg/Max: 20/22/28 milliseconds
        Destination to Source Latency one way Min/Avg/Max: 20/23/26 milliseconds
Jitter Time:
        Number of SD Jitter Samples: 9
        Number of DS Jitter Samples: 9
        Source to Destination Jitter Min/Avg/Max: 1/3/8 milliseconds
        Destination to Source Jitter Min/Avg/Max: 1/2/6 milliseconds
Packet Loss Values:
        Loss Source to Destination: 0        Loss Destination to Source: 0
        Out Of Sequence: 0      Tail Drop: 0        Packet Late Arrival: 0
```

## 8.10 Troubleshooting QoS

### 8.10.1 Issue: Poor VoIP Quality

**Symptoms:**
- Choppy or garbled audio
- One-way audio
- Call drops

**Diagnosis Steps:**

**1. Verify QoS is Enabled:**
```
Switch# show mls qos
QoS is disabled  ← Problem!
```

**Solution:**
```
Switch(config)# mls qos
```

**2. Check Trust Configuration:**
```
Switch# show mls qos interface gigabitethernet 0/5
GigabitEthernet0/5
trust state: not trusted  ← Voice packets not prioritized!
```

**Solution:**
```
Switch(config-if)# mls qos trust cos
```

**3. Verify Voice VLAN:**
```
Switch# show interfaces gigabitethernet 0/5 switchport
Name: Gi0/5
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
Access Mode VLAN: 10 (DATA)
Voice VLAN: none  ← Missing voice VLAN!
```

**Solution:**
```
Switch(config-if)# switchport voice vlan 100
```

**4. Check for Packet Loss:**
```
Router# show policy-map interface gi0/0 | include VOICE -A 5
    Class-map: VOICE (match-all)
      25000 packets, 2500000 bytes
      (total drops/bytes drops) 1250/125000  ← 5% packet loss!
```

**Solution**: Increase priority bandwidth allocation

**5. Verify DSCP Markings:**
```
! Capture packets and verify DSCP
Router# show access-lists 100
Extended IP access list 100
    10 permit ip any any dscp ef (25000 matches)  ← Voice traffic marked correctly
```

### 8.10.2 Issue: Unbalanced Queue Utilization

**Symptoms:**
- Some queues constantly full
- Others mostly empty
- Suboptimal performance

**Diagnosis:**
```
Switch# show mls qos interface gi0/1 statistics

GigabitEthernet0/1 (Egress)

  dscp: incoming
-------------------------------
  0 -  4 :  1000000      0      0      0      0
  8 - 12 :        0      0      0      0      0
 46 - 50 :    25000      0      0      0      0  ← DSCP 46 (Voice)

  queue               dropped  [cos-map]
---------------------------------------
    1                      0  [0 ]
    2                      0  [1 ]
    3                      0  [2 3 ]
    4                 125000  [4 5 6 7 ]  ← Queue 4 dropping packets
```

**Solution**: Adjust queue weights
```
Switch(config-if)# wrr-queue bandwidth percent 5 10 25 60
                                                     ↑ Increase high-priority queue
```

### 8.10.3 Issue: QoS Policy Not Applied

**Symptoms:**
- Policy configured but no effect
- All traffic treated equally

**Diagnosis:**
```
Router# show policy-map interface gigabitethernet 0/0
 GigabitEthernet0/0
% Policy not attached  ← Policy missing!
```

**Solution:**
```
Router(config-if)# service-policy output WAN-EDGE
```

**Verify Classes Match Traffic:**
```
Router# show policy-map interface gi0/0 | include match
      Match: ip dscp ef (46)
        (pkts matched/bytes matched) 0/0  ← No traffic matching!
```

**Possible Causes:**
- DSCP not marked upstream
- Wrong DSCP value in class-map
- ACL not matching traffic

## 8.11 QoS Best Practices

### 8.11.1 Design Guidelines

**1. Follow RFC 4594 - DSCP Recommendations**
```
Voice:           DSCP EF (46)
Call Signaling:  DSCP CS3 (24)
Video:           DSCP AF41 (34)
Business Data:   DSCP AF21 (18)
Best Effort:     DSCP 0 (default)
Scavenger:       DSCP CS1 (8)
```

**2. Trust IP Phones, Classify Everything Else**
```
IP Phones → Trust CoS/DSCP
PCs → Reclassify at switch
Servers → Classify based on application
Inter-switch → Trust DSCP
```

**3. Limit Priority Queue to < 33%**
```
! Voice should not exceed 1/3 of bandwidth
Router(config-pmap-c)# priority percent 30
! Prevents starvation of other traffic
```

**4. Use LLQ for Voice**
```
! Not just priority, but LLQ (priority + policing)
Router(config-pmap-c)# priority percent 20
```

**5. Enable WRED for TCP Traffic**
```
Router(config-pmap-c)# random-detect dscp-based
! Prevents global TCP synchronization
```

**6. Shape Before WAN**
```
! Shape to slightly below WAN speed
! Prevents provider from dropping packets
Router(config-pmap-c)# shape average 95000000
                                    ↑ 95 Mbps for 100M WAN
```

### 8.11.2 Documentation Template

```
QoS POLICY SUMMARY

Traffic Classes:
  1. Voice (EF, DSCP 46)
     - Bandwidth: 20% guaranteed
     - Queuing: Strict priority (LLQ)
     - Applications: VoIP (SIP, H.323, RTP)

  2. Call Signaling (CS3, DSCP 24)
     - Bandwidth: 5% guaranteed
     - Queuing: CBWFQ
     - Applications: SIP, H.323, SCCP

  3. Interactive Video (AF41, DSCP 34)
     - Bandwidth: 30% guaranteed
     - Queuing: CBWFQ with WRED
     - Applications: Video conferencing, telepresence

  4. Business-Critical Data (AF21, DSCP 18)
     - Bandwidth: 25% guaranteed
     - Queuing: CBWFQ with WRED
     - Applications: ERP, CRM, database

  5. Best Effort (Default, DSCP 0)
     - Bandwidth: 20% guaranteed
     - Queuing: Fair-queue with WRED
     - Applications: Email, web, general traffic

Trust Boundaries:
  - Access Layer: Trust Cisco IP Phones (CoS), remark PC traffic
  - Distribution/Core: Trust DSCP
  - WAN Edge: Apply LLQ policy

Applied Policies:
  - WAN-EDGE: Applied to WAN interfaces (outbound)
  - MARK-VOICE: Applied to server ports (inbound)
```

## 8.12 Real-World Scenario

### Enterprise Branch Office QoS

**Requirements:**
- 50 employees with IP phones
- Video conferencing capability
- Business applications (ERP, CRM)
- Guest WiFi
- WAN: 100 Mbps MPLS link

**Design:**

**Traffic Classes:**
1. Voice: 20% (20 Mbps) - LLQ
2. Video: 30% (30 Mbps) - CBWFQ
3. Business Data: 30% (30 Mbps) - CBWFQ
4. Guest/Internet: 20% (20 Mbps) - Best-effort

**Access Switch:**
```
! Enable QoS
Switch(config)# mls qos

! User ports with IP phones
Switch(config)# interface range gi0/1-48
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# switchport voice vlan 100
Switch(config-if-range)# mls qos trust cos
Switch(config-if-range)# exit

! Guest WiFi access point
Switch(config)# interface gi0/20
Switch(config-if)# switchport access vlan 30
Switch(config-if)# no mls qos trust
Switch(config-if)# exit
```

**WAN Router:**
```
! Traffic classification
Router(config)# class-map VOICE
Router(config-cmap)# match dscp ef
Router(config)# class-map VIDEO
Router(config-cmap)# match dscp af41
Router(config)# class-map BUSINESS
Router(config-cmap)# match dscp af21
Router(config)# class-map GUEST
Router(config-cmap)# match access-group 110
Router(config)# access-list 110 permit ip 192.168.30.0 0.0.0.255 any

! QoS policy
Router(config)# policy-map WAN-100M
Router(config-pmap)# class VOICE
Router(config-pmap-c)# priority percent 20
Router(config-pmap-c)# class VIDEO
Router(config-pmap-c)# bandwidth percent 30
Router(config-pmap-c)# class BUSINESS
Router(config-pmap-c)# bandwidth percent 30
Router(config-pmap-c)# class GUEST
Router(config-pmap-c)# bandwidth percent 10
Router(config-pmap-c)# police 10000000
Router(config-pmap-c)# class class-default
Router(config-pmap-c)# bandwidth percent 10
Router(config-pmap-c)# random-detect
Router(config-pmap-c)# exit

! Apply to WAN
Router(config)# interface gi0/0
Router(config-if)# service-policy output WAN-100M
Router(config-if)# exit
```

**Result:**
- VoIP quality guaranteed even during congestion
- Video conferencing receives adequate bandwidth
- Guest traffic limited to 10 Mbps (policed)
- Business applications protected
- Automatic congestion avoidance with WRED

## Summary

Quality of Service (QoS) ensures critical applications receive necessary network resources:

- **Classification and Marking**: DSCP (Layer 3) and CoS (Layer 2) identify traffic
- **Trust Boundaries**: Trust IP phones, reclassify at switch edge
- **Queuing**: LLQ provides strict priority for voice, CBWFQ for other classes
- **Congestion Avoidance**: WRED prevents global TCP synchronization
- **Shaping vs Policing**: Shape for WAN links, police for rate limiting
- **VoIP Requirements**: < 150 ms latency, < 30 ms jitter, < 1% loss, DSCP EF

**Best Practices:**
- Follow RFC 4594 DSCP guidelines
- Limit priority queue to < 33% bandwidth
- Use LLQ for voice traffic
- Apply QoS at WAN edge
- Document and test QoS policies

This completes our comprehensive coverage of switching fundamentals. The next chapters will cover advanced topics such as security features, switch stacking, virtual chassis technologies, and network automation.
