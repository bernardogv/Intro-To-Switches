# Chapter 2: Networking Fundamentals

## The OSI Model: A Framework for Understanding Networks

The **Open Systems Interconnection (OSI) model** is a conceptual framework that standardizes how network systems communicate. While it has seven layers, switches primarily operate at Layers 2 and 3, so we'll focus our attention there.

### Complete OSI Model Overview

```
┌─────────────────────────────────────────────┐
│ Layer 7: Application   │ HTTP, FTP, DNS     │ ◄── User applications
├─────────────────────────────────────────────┤
│ Layer 6: Presentation  │ SSL/TLS, JPEG      │ ◄── Data format/encryption
├─────────────────────────────────────────────┤
│ Layer 5: Session       │ NetBIOS, RPC       │ ◄── Connection management
├─────────────────────────────────────────────┤
│ Layer 4: Transport     │ TCP, UDP           │ ◄── End-to-end delivery
├─────────────────────────────────────────────┤
│ Layer 3: Network       │ IP, ICMP, Routing  │ ◄── LAYER 3 SWITCHES
├─────────────────────────────────────────────┤
│ Layer 2: Data Link     │ Ethernet, MAC      │ ◄── LAYER 2 SWITCHES
├─────────────────────────────────────────────┤
│ Layer 1: Physical      │ Cables, Signals    │ ◄── Physical connectivity
└─────────────────────────────────────────────┘
```

**Mnemonic to remember (bottom-up):** "Please Do Not Throw Sausage Pizza Away"

### Layer 1: Physical Layer (Foundation)

**What it does**: Transmits raw bits over physical medium

**Components:**
- Cables (copper, fiber)
- Connectors (RJ45, SFP)
- Electrical signals/light pulses
- Voltage levels, frequencies

**Switch role at Layer 1:**
- Provides physical ports (RJ45, SFP+)
- Converts electrical signals to data
- Auto-negotiates speed (10/100/1000 Mbps)
- Monitors link status (up/down)

**Real-world perspective**: Layer 1 is "can I see a blinking light on the port?" If the physical layer fails, nothing else matters.

### Layer 2: Data Link Layer (Switching Core)

**What it does**: Provides node-to-node data transfer and handles error correction from the physical layer

**Key Concepts:**

#### 1. MAC Addresses (Media Access Control)
Every network interface has a unique 48-bit hardware address:

```
MAC Address: AA:BB:CC:DD:EE:FF
             │     │      │
             │     │      └──── Device identifier (24 bits)
             │     └──────────── Manufacturer ID (24 bits - OUI)
             └────────────────── Organizationally Unique Identifier

Examples:
- 00:50:56:C0:00:01  (VMware virtual NIC)
- F0:18:98:XX:XX:XX  (Apple device)
- 00:0C:29:XX:XX:XX  (VMware)
```

**MAC Address Structure:**
- **First 24 bits (OUI)**: Manufacturer identifier assigned by IEEE
  - `00:50:56` = VMware
  - `00:1A:2B` = Cisco
  - `D8:9E:F3` = Apple
- **Last 24 bits**: Device serial number assigned by manufacturer
- **Special addresses**:
  - `FF:FF:FF:FF:FF:FF` = Broadcast (all devices)
  - `01:00:5E:XX:XX:XX` = Multicast addresses

#### 2. Ethernet Frames

Data at Layer 2 is encapsulated in **frames**:

```
┌──────────┬─────────┬─────────┬─────────┬──────────┬──────┬─────┐
│ Preamble │   DA    │   SA    │  Type   │   Data   │ FCS  │ IFG │
│  (7+1)   │ (6 B)   │ (6 B)   │ (2 B)   │(46-1500) │(4 B) │     │
└──────────┴─────────┴─────────┴─────────┴──────────┴──────┴─────┘

Components:
- Preamble + SFD: Synchronization (8 bytes)
- Destination Address (DA): Target MAC (6 bytes)
- Source Address (SA): Sender MAC (6 bytes)
- Type/Length: Protocol identifier (2 bytes)
- Data (Payload): 46-1500 bytes
- FCS (Frame Check Sequence): CRC error detection (4 bytes)
- IFG (Interframe Gap): Spacing between frames (12 bytes)

Total frame size: 64-1518 bytes (standard Ethernet)
```

**Frame processing by switch:**
1. Receives frame on ingress port
2. Reads destination MAC address
3. Looks up MAC in forwarding table
4. Forwards to appropriate egress port
5. Verifies frame integrity (FCS check)

#### 3. MAC Address Table (CAM Table)

Switches build a **Content Addressable Memory (CAM) table** by learning:

```
Port | MAC Address        | VLAN | Timestamp
-----|-------------------|------|------------
 1   | AA:BB:CC:DD:EE:01 |  10  | 00:00:15
 2   | AA:BB:CC:DD:EE:02 |  10  | 00:00:30
 3   | AA:BB:CC:DD:EE:03 |  20  | 00:01:45
24   | AA:BB:CC:DD:EE:18 |  10  | 00:02:00
```

**Learning process:**
1. Frame arrives on port 1
2. Switch reads **source MAC** from frame
3. Records: "MAC AA:BB:CC:DD:EE:01 is on port 1"
4. Reads **destination MAC** from frame
5. If MAC in table → forward to specific port
6. If MAC unknown → flood to all ports (except source)
7. Entries age out (typically 300 seconds of inactivity)

**Example scenario:**
```
Initial state: Empty MAC table

1. PC-A (port 1) sends to PC-B
   → Switch learns: PC-A is on port 1
   → Destination unknown: floods all ports

2. PC-B (port 2) replies to PC-A
   → Switch learns: PC-B is on port 2
   → Destination known: forwards only to port 1

3. Future PC-A ↔ PC-B traffic
   → Both MACs known: direct forwarding
   → No flooding needed!
```

#### 4. Layer 2 Switching Modes

**Store-and-Forward** (most common):
- Receives entire frame
- Checks FCS for errors
- Then forwards frame
- **Pros**: Error checking, quality assurance
- **Cons**: Higher latency (~10-40 microseconds)

**Cut-Through**:
- Starts forwarding after reading destination MAC
- No error checking
- **Pros**: Ultra-low latency (~2-5 microseconds)
- **Cons**: Forwards bad frames

**Fragment-Free** (modified cut-through):
- Reads first 64 bytes (collision fragment size)
- Balances speed and error detection
- **Pros**: Catches most errors with low latency
- **Cons**: Still forwards some corrupted frames

### Layer 3: Network Layer (Routing)

**What it does**: Provides logical addressing and routing between networks

**Key Concepts:**

#### 1. IP Addressing

**IPv4 Address Structure:**
```
192.168.1.10
 │   │  │ │
 │   │  │ └── Host identifier
 │   │  └──── Network identifier
 │   └─────── Subnet
 └─────────── Network class

With subnet mask: 255.255.255.0 (/24)
Network portion: 192.168.1.0
Broadcast: 192.168.1.255
Usable hosts: 192.168.1.1 - 192.168.1.254
```

**IP Address Classes (historical):**
- **Class A**: 1.0.0.0 - 126.255.255.255 (large networks)
- **Class B**: 128.0.0.0 - 191.255.255.255 (medium networks)
- **Class C**: 192.0.0.0 - 223.255.255.255 (small networks)
- **Class D**: 224.0.0.0 - 239.255.255.255 (multicast)
- **Class E**: 240.0.0.0 - 255.255.255.255 (experimental)

**Private IP Ranges (RFC 1918):**
- 10.0.0.0/8 (10.0.0.0 - 10.255.255.255)
- 172.16.0.0/12 (172.16.0.0 - 172.31.255.255)
- 192.168.0.0/16 (192.168.0.0 - 192.168.255.255)

#### 2. Layer 3 Switching (Inter-VLAN Routing)

Layer 3 switches combine switching and routing:

```
VLAN 10 (192.168.10.0/24)          VLAN 20 (192.168.20.0/24)
      │                                   │
      ├── PC-A: 192.168.10.5             ├── PC-C: 192.168.20.5
      └── PC-B: 192.168.10.6             └── PC-D: 192.168.20.6
             │                                   │
             └─────── Layer 3 Switch ───────────┘
                    (Routes between VLANs)

Communication flow (PC-A to PC-C):
1. PC-A sends to gateway (Layer 3 switch: 192.168.10.1)
2. Switch routes packet from VLAN 10 to VLAN 20
3. Switch forwards to PC-C (192.168.20.5)
```

**Layer 3 features:**
- Static routing
- Dynamic routing (RIP, OSPF, EIGRP)
- Inter-VLAN routing
- Access Control Lists (ACLs)
- IP address management

## Ethernet Standards: The Physical Foundation

Ethernet has evolved dramatically since its 1973 invention at Xerox PARC. Understanding these standards is crucial for switch selection and network design.

### Ethernet Speed Evolution

```
1980s      1990s      2000s      2010s      2020s
  │          │          │          │          │
10Base-T  100Base-TX  1000Base-T  10GBase-T  25/40/100/400G
(10 Mbps) (100 Mbps)   (1 Gbps)  (10 Gbps)  (Datacenter)
```

### Detailed Standards

#### 10 Mbps Ethernet (Legacy)

**10Base-T** (IEEE 802.3i - 1990)
- **Speed**: 10 Mbps
- **Cable**: Cat3 or better, UTP
- **Distance**: 100 meters
- **Connector**: RJ45
- **Usage**: Obsolete, historical interest only

#### 100 Mbps Fast Ethernet

**100Base-TX** (IEEE 802.3u - 1995)
- **Speed**: 100 Mbps full-duplex (200 Mbps aggregate)
- **Cable**: Cat5 or better, UTP (2 pairs)
- **Distance**: 100 meters
- **Connector**: RJ45
- **Usage**: Still common for end devices (PCs, printers)
- **Power**: Low cost, adequate for basic use

**100Base-FX** (Fiber variant)
- **Speed**: 100 Mbps
- **Cable**: Multimode fiber
- **Distance**: 2 km
- **Usage**: Building-to-building connections

#### 1000 Mbps Gigabit Ethernet

**1000Base-T** (IEEE 802.3ab - 1999)
- **Speed**: 1 Gbps (1000 Mbps)
- **Cable**: Cat5e or better, UTP (4 pairs)
- **Distance**: 100 meters
- **Connector**: RJ45
- **Usage**: Current standard for most networks
- **Notes**: Requires all 8 wires (unlike Fast Ethernet)

**1000Base-SX** (Short-range fiber)
- **Speed**: 1 Gbps
- **Cable**: Multimode fiber (850 nm)
- **Distance**: 550 meters
- **Connector**: LC, SC
- **Usage**: Datacenter, server connections

**1000Base-LX** (Long-range fiber)
- **Speed**: 1 Gbps
- **Cable**: Single-mode or multimode fiber (1310 nm)
- **Distance**: 5 km (multimode), 10 km (single-mode)
- **Connector**: LC, SC
- **Usage**: Campus backbone, long distances

#### 10 Gigabit Ethernet

**10GBase-T** (IEEE 802.3an - 2006)
- **Speed**: 10 Gbps
- **Cable**: Cat6a or Cat7, UTP/STP
- **Distance**: 100 meters (Cat6a), 55 meters (Cat6)
- **Connector**: RJ45
- **Usage**: Server connections, uplinks
- **Power**: Higher power consumption (2-5W per port)

**10GBase-SR** (Short-range fiber)
- **Speed**: 10 Gbps
- **Cable**: Multimode fiber (850 nm)
- **Distance**: 300 meters (OM3), 400 meters (OM4)
- **Connector**: LC
- **Usage**: Datacenter top-of-rack to aggregation

**10GBase-LR** (Long-range fiber)
- **Speed**: 10 Gbps
- **Cable**: Single-mode fiber (1310 nm)
- **Distance**: 10 km
- **Connector**: LC
- **Usage**: Datacenter interconnect, MAN

#### 25/40/100 Gigabit Ethernet (Datacenter)

**25GBase-SR** (IEEE 802.3by - 2016)
- **Speed**: 25 Gbps per lane
- **Cable**: Multimode fiber
- **Distance**: 100 meters
- **Usage**: Modern server NICs (2×25G = 50G)

**40GBase-SR4** (IEEE 802.3ba - 2010)
- **Speed**: 40 Gbps (4×10G lanes)
- **Cable**: Multimode fiber (MPO/MTP connector)
- **Distance**: 100-150 meters
- **Usage**: Datacenter spine/leaf architecture

**100GBase-SR4**
- **Speed**: 100 Gbps (4×25G lanes)
- **Cable**: Multimode fiber (MPO/MTP)
- **Distance**: 100 meters
- **Usage**: Hyperscale datacenter cores

**400GBase-SR8** (IEEE 802.3bs - 2017)
- **Speed**: 400 Gbps (8×50G lanes)
- **Cable**: Multimode fiber
- **Distance**: 100 meters
- **Usage**: Next-gen datacenter backbones

### Transceiver Types (Pluggable Optics)

Modern switches use hot-swappable transceivers:

**SFP (Small Form-factor Pluggable)**
- **Speed**: 100 Mbps - 1 Gbps
- **Types**: SX (multimode), LX (single-mode), T (copper)
- **Usage**: Gigabit Ethernet connections

**SFP+ (Enhanced SFP)**
- **Speed**: 10 Gbps
- **Types**: SR, LR, ER (fiber), DAC (copper)
- **Usage**: 10 Gigabit Ethernet

**QSFP+ (Quad SFP)**
- **Speed**: 40 Gbps (4×10G)
- **Usage**: 40G connections

**QSFP28**
- **Speed**: 100 Gbps (4×25G)
- **Usage**: 100G connections

**Comparison Table:**

| Standard | Speed | Cable Type | Distance | Common Use |
|----------|-------|-----------|----------|------------|
| 100Base-TX | 100 Mbps | Cat5 UTP | 100m | End devices |
| 1000Base-T | 1 Gbps | Cat5e UTP | 100m | Desktop/access |
| 1000Base-SX | 1 Gbps | MM fiber | 550m | Building links |
| 10GBase-T | 10 Gbps | Cat6a UTP | 100m | Server uplinks |
| 10GBase-SR | 10 Gbps | MM fiber | 300m | Datacenter |
| 25GBase-SR | 25 Gbps | MM fiber | 100m | Server NICs |
| 40GBase-SR4 | 40 Gbps | MM fiber | 150m | Spine switches |
| 100GBase-SR4 | 100 Gbps | MM fiber | 100m | Core switches |

## MAC Addresses and Switching Process

### MAC Address Deep Dive

**Format variations:**
```
Canonical form: AA:BB:CC:DD:EE:FF (colon-separated)
Windows style: AA-BB-CC-DD-EE-FF (hyphen-separated)
Cisco style: aabb.ccdd.eeff (dotted groups)
Compressed: AABBCCDDEEFF (no separators)
```

**Special MAC addresses:**
```
FF:FF:FF:FF:FF:FF    → Layer 2 broadcast (all devices)
01:00:5E:XX:XX:XX    → IPv4 multicast
33:33:XX:XX:XX:XX    → IPv6 multicast
01:80:C2:00:00:00    → Spanning Tree Protocol (STP)
```

**MAC address assignment:**
- **Burned-in Address (BIA)**: Hardcoded in NIC ROM
- **Locally Administered**: Software-configured (2nd bit of first byte = 1)
- **Unicast**: Normal device address (LSB of first byte = 0)
- **Multicast**: Group address (LSB of first byte = 1)

### Switching Decisions: The Complete Flow

```
Packet arrives at switch
         │
         ▼
    ┌────────────────┐
    │ Frame received │
    │  on Port X     │
    └────────┬───────┘
             │
             ▼
    ┌────────────────┐
    │ Read Source MAC│ → Update CAM table
    │ (MAC Learning) │   (Port X has this MAC)
    └────────┬───────┘
             │
             ▼
    ┌────────────────┐
    │Read Destination│
    │      MAC       │
    └────────┬───────┘
             │
             ▼
    ┌────────────────┐
    │  MAC in CAM?   │
    └────────┬───────┘
             │
        ┌────┴─────┐
        │          │
       YES        NO
        │          │
        ▼          ▼
    Forward     Flood to
    to port     all ports
    (unicast)   (unknown)
```

**Detailed example:**

```
Network topology:
Port 1: PC-A (MAC: AA:AA:AA:AA:AA:01)
Port 2: PC-B (MAC: BB:BB:BB:BB:BB:02)
Port 3: PC-C (MAC: CC:CC:CC:CC:CC:03)

Scenario 1: PC-A sends to PC-B (first time)
────────────────────────────────────────
1. Frame arrives on Port 1
   - Source MAC: AA:AA:AA:AA:AA:01
   - Dest MAC: BB:BB:BB:BB:BB:02

2. Switch learns: "Port 1 → MAC AA:AA:AA:AA:AA:01"

3. Switch checks CAM for BB:BB:BB:BB:BB:02
   - Not found (first communication)

4. Switch floods frame to ports 2, 3
   (all ports except source port 1)

5. PC-B receives frame and replies
   - Source MAC: BB:BB:BB:BB:BB:02
   - Dest MAC: AA:AA:AA:AA:AA:01

6. Switch learns: "Port 2 → MAC BB:BB:BB:BB:BB:02"

7. Switch checks CAM for AA:AA:AA:AA:AA:01
   - Found! Forward to Port 1 only

8. CAM table now contains:
   Port 1 → AA:AA:AA:AA:AA:01
   Port 2 → BB:BB:BB:BB:BB:02

Scenario 2: PC-A sends to PC-B (subsequent)
───────────────────────────────────────────
1. Frame arrives on Port 1

2. Switch already knows both MACs

3. Direct forward Port 1 → Port 2
   (no flooding needed!)
```

## Frames, Packets, and Data Flow

### Encapsulation: The Layer Model in Practice

As data travels down the OSI stack, it gets wrapped in headers:

```
Application Layer (Layer 7)
         │
         │ User types: "Hello"
         ▼
     [ DATA ]
         │
         ▼
Transport Layer (Layer 4)
         │
         │ Add TCP header (ports, sequence numbers)
         ▼
 [ TCP | DATA ]
  └─ Segment
         │
         ▼
Network Layer (Layer 3)
         │
         │ Add IP header (source/dest IP addresses)
         ▼
 [ IP | TCP | DATA ]
  └─── Packet
         │
         ▼
Data Link Layer (Layer 2)
         │
         │ Add Ethernet header + trailer
         ▼
 [ ETH | IP | TCP | DATA | FCS ]
  └──────── Frame ────────────┘
         │
         ▼
Physical Layer (Layer 1)
         │
         │ Convert to electrical signals
         ▼
    1010110101... (bits on wire)
```

**Terminology:**
- **Bits**: Layer 1 (Physical)
- **Frames**: Layer 2 (Data Link)
- **Packets**: Layer 3 (Network)
- **Segments**: Layer 4 (Transport)
- **Data**: Layer 7 (Application)

### Broadcast, Unicast, and Multicast

**Unicast** (one-to-one):
```
Source: AA:AA:AA:AA:AA:01
Dest: BB:BB:BB:BB:BB:02
Result: Switch forwards to specific port only
```

**Broadcast** (one-to-all):
```
Source: AA:AA:AA:AA:AA:01
Dest: FF:FF:FF:FF:FF:FF
Result: Switch floods to all ports in VLAN
Examples: ARP requests, DHCP discovery
```

**Multicast** (one-to-group):
```
Source: AA:AA:AA:AA:AA:01
Dest: 01:00:5E:XX:XX:XX (multicast group)
Result: Switch forwards to subscribed ports
Examples: Video streaming, stock tickers
```

### Data Flow Example: Web Request

```
User opens browser and types: http://www.example.com

┌─────────────────────────────────────────────────────────┐
│ Step 1: DNS Resolution (Application Layer)              │
├─────────────────────────────────────────────────────────┤
│ PC needs IP address for www.example.com                │
│ Sends DNS query to DNS server                          │
└─────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────┐
│ Step 2: ARP Request (Layer 2)                           │
├─────────────────────────────────────────────────────────┤
│ PC knows DNS server IP: 192.168.1.1                    │
│ PC doesn't know MAC address of 192.168.1.1             │
│ Broadcasts ARP: "Who has 192.168.1.1?"                 │
│ Dest MAC: FF:FF:FF:FF:FF:FF (broadcast)                │
└─────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────┐
│ Step 3: Switch Floods ARP Broadcast                     │
├─────────────────────────────────────────────────────────┤
│ Switch receives broadcast on Port 5                     │
│ Floods to all ports in same VLAN (except Port 5)       │
│ DNS server receives ARP request                         │
└─────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────┐
│ Step 4: ARP Reply (Unicast)                            │
├─────────────────────────────────────────────────────────┤
│ DNS server replies: "192.168.1.1 is AA:BB:CC:DD:EE:01"│
│ Dest MAC: PC's MAC address (unicast)                   │
│ Switch forwards to Port 5 only (learned from Step 3)   │
└─────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────┐
│ Step 5: DNS Query Sent                                  │
├─────────────────────────────────────────────────────────┤
│ PC now knows DNS server's MAC address                  │
│ Sends DNS query for www.example.com                    │
│ Switch forwards to DNS server (unicast)                │
└─────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────┐
│ Step 6: HTTP Request Follows Same Pattern               │
├─────────────────────────────────────────────────────────┤
│ PC receives IP address: 93.184.216.34                  │
│ If destination is remote, ARP for default gateway      │
│ Send HTTP GET request through gateway/router           │
└─────────────────────────────────────────────────────────┘
```

**Key observations:**
- **Layer 2 switching**: Fast, hardware-based MAC forwarding
- **ARP**: Bridges Layer 2 (MAC) and Layer 3 (IP)
- **Learning**: Switch builds MAC table from source addresses
- **Flooding**: Used for broadcasts and unknown unicasts
- **Efficiency**: Subsequent traffic uses learned MAC table

## Key Takeaways

1. **OSI Model provides structure**: Layers 2-3 are the switch's domain. Layer 2 handles MAC-based forwarding; Layer 3 adds IP routing capabilities.

2. **Ethernet has evolved dramatically**: From 10 Mbps to 400 Gbps, with the sweet spot currently at 1 Gbps for access and 10-100 Gbps for uplinks.

3. **MAC addresses are fundamental**: 48-bit hardware addresses enable Layer 2 switching. The CAM table is the heart of switch operation.

4. **Frames are the Layer 2 unit**: Ethernet frames contain source/destination MACs, data payload, and error checking (FCS).

5. **Switching is about learning and forwarding**:
   - Learn source MACs → build CAM table
   - Forward to known destinations → unicast
   - Flood unknown destinations → temporary until learned

6. **Standards matter**: Choose the right Ethernet standard based on distance, speed requirements, and budget (copper vs. fiber, Cat5e vs. Cat6a).

7. **Encapsulation explains data flow**: Each layer adds headers, creating frames (L2), packets (L3), and segments (L4).

## What's Next?

In Chapter 3, we'll explore the different types of switches available—unmanaged, managed, smart switches, PoE variants, and enterprise vs. SMB options. You'll learn how to select the right switch for your specific needs based on features, performance, and budget.

---

**Chapter 2 Complete** | Next: Chapter 3 - Switch Types and Selection
