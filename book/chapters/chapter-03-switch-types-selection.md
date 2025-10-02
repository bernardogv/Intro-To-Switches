# Chapter 3: Switch Types and Selection

## Overview: Choosing the Right Switch

Not all switches are created equal. The market offers everything from $20 unmanaged switches for home use to $50,000+ chassis-based systems for datacenters. Understanding the differences is crucial for making cost-effective, future-proof decisions.

**This chapter covers:**
- Switch categories and management capabilities
- Enterprise vs. SMB requirements
- Power over Ethernet (PoE) options
- Selection criteria and decision frameworks

## Unmanaged Switches: Plug-and-Play Simplicity

### What Are Unmanaged Switches?

**Unmanaged switches** are basic Layer 2 switches that work out of the box with zero configuration. They're essentially "intelligent hubs" with MAC address learning but no user-configurable features.

```
Unmanaged Switch Characteristics:
┌─────────────────────────────────────────────┐
│ ✓ Plug and play - no setup needed          │
│ ✓ Auto-MDI/MDIX - any cable works          │
│ ✓ Auto-negotiation - speed/duplex automatic│
│ ✗ No configuration interface                │
│ ✗ No VLANs, QoS, or security features     │
│ ✗ No monitoring or management              │
└─────────────────────────────────────────────┘
```

### When to Use Unmanaged Switches

**Ideal scenarios:**
- **Home networks**: Connecting gaming consoles, smart TVs, PCs
- **Small offices** (< 10 devices): Basic connectivity with no special requirements
- **Network expansion**: Adding ports to an existing managed network
- **Temporary setups**: Events, trade shows, pop-up offices
- **Budget-constrained**: When cost is the primary concern

**Real-world example:**
```
Home Network:
Internet ──► Router ──┬──► WiFi devices
                      │
                      └──► Unmanaged Switch ──┬──► Gaming PC
                                               ├──► Smart TV
                                               ├──► NAS storage
                                               └──► Game console
```

### Advantages of Unmanaged Switches

**1. Zero Learning Curve**
- Anyone can plug it in and use it
- No IT knowledge required
- No misconfigurations possible

**2. Low Cost**
- **5-port**: $15-$30
- **8-port**: $20-$50
- **16-port**: $60-$120
- **24-port**: $100-$200

**3. Reliability**
- Fewer features = fewer failure points
- Passive cooling (fanless for small models)
- Long MTBF (Mean Time Between Failures)

**4. Low Power Consumption**
- Typically 5-15W for 8-port models
- No configuration overhead
- Energy-efficient designs

### Limitations of Unmanaged Switches

**1. No Network Segmentation**
- All devices in single broadcast domain
- No VLANs for security/organization
- Cannot separate guest from corporate traffic

**2. No Quality of Service (QoS)**
- VoIP and video compete with file transfers
- No traffic prioritization
- Best-effort delivery only

**3. No Monitoring or Troubleshooting**
- Can't see port status remotely
- No traffic statistics
- No SNMP monitoring
- Black box during issues

**4. No Security Features**
- No port security (MAC filtering)
- No access control lists (ACLs)
- No 802.1X authentication
- Vulnerable to ARP spoofing, MAC flooding

**5. Limited Scalability**
- Typically max 24-48 ports
- No stacking capabilities
- Cannot manage multiple switches centrally

### Popular Unmanaged Switch Models

| Model | Ports | Speed | Price | Use Case |
|-------|-------|-------|-------|----------|
| Netgear GS305 | 5 | 1 Gbps | $20 | Home desk expansion |
| TP-Link TL-SG108 | 8 | 1 Gbps | $25 | Small office |
| Netgear GS116 | 16 | 1 Gbps | $80 | Small business |
| TP-Link TL-SG1024D | 24 | 1 Gbps | $120 | Medium office |
| Netgear GS110MX | 8+2 | 1G+10G | $180 | High-speed uplinks |

### Selection Criteria for Unmanaged Switches

**Questions to ask:**
1. **How many ports?** Count devices + 20% growth buffer
2. **What speed?** Gigabit (1 Gbps) is standard; avoid Fast Ethernet (100 Mbps)
3. **Metal or plastic case?** Metal = better cooling and durability
4. **Fanless?** Critical for quiet environments (offices, studios)
5. **Desktop or rack-mount?** Form factor matters for installation
6. **Warranty?** Lifetime warranties common (Netgear, TP-Link)

**Decision matrix:**
```
If: Home network, < 10 devices, budget < $50
   → 8-port unmanaged gigabit switch

If: Small office, 10-20 devices, basic needs
   → 24-port unmanaged gigabit switch

If: Any VLANs, QoS, or monitoring needed
   → Skip unmanaged, choose managed/smart switch
```

## Managed Switches: Full Control and Features

### What Are Managed Switches?

**Managed switches** provide complete configuration control through web GUI, CLI (Command Line Interface), or SNMP. They offer advanced features for performance, security, and network design.

```
Managed Switch Capabilities:
┌─────────────────────────────────────────────┐
│ ✓ VLANs - network segmentation              │
│ ✓ QoS - traffic prioritization              │
│ ✓ Port mirroring - troubleshooting          │
│ ✓ Link aggregation (LACP) - redundancy     │
│ ✓ Spanning Tree Protocol (STP) - loop free │
│ ✓ SNMP monitoring - health tracking         │
│ ✓ Access control (ACLs, 802.1X)            │
│ ✓ Layer 3 routing (on L3 models)           │
└─────────────────────────────────────────────┘
```

### Management Interfaces

**1. Command Line Interface (CLI)**
```
Switch> enable
Switch# configure terminal
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# spanning-tree portfast
Switch(config-if)# exit
Switch(config)# exit
Switch# write memory
```
- **Pros**: Scriptable, fast for experts, consistent across devices
- **Cons**: Steep learning curve, syntax-specific
- **Common syntaxes**: Cisco IOS, Arista EOS, Juniper Junos

**2. Web GUI (Graphical User Interface)**
```
Browser-based interface:
┌─────────────────────────────────────────┐
│ Dashboard                                │
├─────────────────────────────────────────┤
│ Port Status: 24/48 Up                   │
│ CPU Usage: 12%                          │
│ Temperature: 42°C                       │
│                                         │
│ [Configure VLANs] [Port Settings]      │
│ [View Statistics] [System Logs]        │
└─────────────────────────────────────────┘
```
- **Pros**: Visual, intuitive, good for occasional management
- **Cons**: Slower than CLI, vendor-specific layouts

**3. SNMP (Simple Network Management Protocol)**
- **Read-only (v2c)**: Monitor status remotely
- **Read-write (v3)**: Configure via NMS (Network Management System)
- **Use case**: Centralized management (Nagios, PRTG, SolarWinds)

**4. Mobile Apps**
- Some vendors (UniFi, Meraki) offer mobile management
- Useful for quick status checks and basic configuration
- Limited compared to full interfaces

### Key Managed Switch Features

#### 1. VLANs (Virtual Local Area Networks)

**Purpose**: Segment single physical switch into multiple logical networks

```
Physical Switch (48 ports):

VLAN 10 - Management (Ports 1-4)
   └─► 192.168.10.0/24

VLAN 20 - Finance (Ports 5-12)
   └─► 192.168.20.0/24

VLAN 30 - Engineering (Ports 13-36)
   └─► 192.168.30.0/24

VLAN 40 - Guest WiFi (Ports 37-44)
   └─► 192.168.40.0/24

VLAN 99 - Trunk (Ports 45-48)
   └─► Uplinks to other switches
```

**Benefits:**
- **Security**: Finance can't see Engineering traffic
- **Performance**: Reduced broadcast domains
- **Flexibility**: Logical grouping independent of physical location
- **Compliance**: Isolate PCI/HIPAA systems

**VLAN types:**
- **Access VLAN**: Single VLAN per port (end devices)
- **Trunk VLAN**: Multiple VLANs on one port (switch-to-switch)
- **Voice VLAN**: Separate VLAN for VoIP phones
- **Native VLAN**: Untagged traffic on trunk (typically VLAN 1)

#### 2. Quality of Service (QoS)

**Purpose**: Prioritize time-sensitive traffic over bulk transfers

```
Traffic Priority (802.1p CoS - Class of Service):
┌──────────────────────────────────────────┐
│ Priority 7: Network Control (highest)    │
│ Priority 6: Voice (VoIP)                 │
│ Priority 5: Video Conferencing           │
│ Priority 4: Critical Applications        │
│ Priority 3: Excellent Effort             │
│ Priority 2: Best Effort (default)        │
│ Priority 1: Background                   │
│ Priority 0: Spare (lowest)               │
└──────────────────────────────────────────┘
```

**QoS methods:**
- **802.1p**: Layer 2 CoS markings (0-7)
- **DSCP**: Layer 3 IP packet markings (0-63)
- **Rate limiting**: Cap bandwidth per port/VLAN
- **Queue management**: Weighted fair queuing, strict priority

**Real-world example:**
```
Before QoS:
VoIP call (needs 100 Kbps, < 150ms latency)
+ File transfer (using 900 Mbps)
= Choppy voice, packet loss

After QoS:
VoIP traffic gets priority queue
File transfer uses remaining bandwidth
= Clear voice, slightly slower file transfer
```

#### 3. Spanning Tree Protocol (STP)

**Purpose**: Prevent Layer 2 loops while providing redundancy

```
Without STP:
Switch A ──┬──► Switch B
           │       │
           └───────┘
Result: Broadcast storm, network meltdown!

With STP:
Switch A ──┬──► Switch B
           │       │
           └───X───┘ (blocked to prevent loop)
If primary link fails → backup link activates
```

**STP variants:**
- **STP (802.1D)**: Original, slow convergence (30-50 seconds)
- **RSTP (802.1w)**: Rapid, fast convergence (< 10 seconds)
- **MSTP (802.1s)**: Multiple Spanning Tree, VLAN-aware
- **PVST+**: Cisco proprietary, per-VLAN spanning tree

#### 4. Link Aggregation (LACP)

**Purpose**: Combine multiple links for increased bandwidth and redundancy

```
Server                Switch
┌────┐              ┌────┐
│    │──Port 1──────│ 1  │
│    │──Port 2──────│ 2  │  LAG (4 Gbps total)
│    │──Port 3──────│ 3  │  Failover if link dies
│    │──Port 4──────│ 4  │
└────┘              └────┘

Configuration:
- 4×1 Gbps links = 4 Gbps aggregate
- Traffic load-balanced across links
- If one link fails, traffic redistributed
```

**LACP modes:**
- **Active**: Actively negotiates LACP
- **Passive**: Waits for LACP negotiation
- **On**: Forces aggregation (no negotiation - risky)

#### 5. Port Mirroring (SPAN)

**Purpose**: Copy traffic from one port to another for analysis

```
┌─────────────────────────────────────┐
│      Port 1 ──► Application server  │
│         │                            │
│         │ (mirrored to)             │
│         ▼                            │
│      Port 24 ──► Wireshark laptop   │
└─────────────────────────────────────┘

Use cases:
- Network troubleshooting
- Security monitoring (IDS/IPS)
- Traffic analysis
- Compliance auditing
```

#### 6. Access Control Lists (ACLs)

**Purpose**: Filter traffic based on rules (Layer 2-4)

```
Example ACL: Block Telnet to servers
─────────────────────────────────────
deny tcp any 192.168.100.0/24 eq 23
permit ip any any

Result: Blocks TCP port 23 (Telnet) to server subnet,
        allows all other traffic
```

**ACL types:**
- **Standard ACLs**: Filter by source IP only
- **Extended ACLs**: Filter by source/dest IP, port, protocol
- **MAC ACLs**: Filter by source/dest MAC address

#### 7. 802.1X Network Access Control

**Purpose**: Authenticate users/devices before granting network access

```
Device connects → Switch blocks traffic
                ↓
      802.1X authentication request
                ↓
    User enters credentials (EAP)
                ↓
   RADIUS server validates credentials
                ↓
        Switch grants/denies access
                ↓
    If granted, assigns VLAN and ACLs
```

**Benefits:**
- Only authenticated devices access network
- Dynamic VLAN assignment
- Guest vs. employee differentiation
- NAC (Network Access Control) integration

### When to Use Managed Switches

**Required scenarios:**
- **VLANs needed**: Multiple departments, security zones
- **VoIP deployment**: QoS critical for call quality
- **Network redundancy**: Need STP and LACP
- **Compliance**: PCI-DSS, HIPAA, SOX requirements
- **Monitoring**: SNMP, syslog, traffic analysis
- **Security**: ACLs, 802.1X, port security
- **Enterprise environments**: 50+ users, mission-critical

**Cost considerations:**
- **Entry managed** (SMB): $200-$1,000 (8-48 ports)
- **Enterprise access**: $1,000-$5,000 (24-48 ports)
- **Enterprise distribution**: $5,000-$20,000
- **Core chassis**: $20,000-$100,000+

### Popular Managed Switch Lines

| Vendor | Product Line | Target Market | Key Features |
|--------|-------------|---------------|--------------|
| Cisco | Catalyst 9000 | Enterprise | Full feature set, automation |
| Cisco | Catalyst 1000 | SMB | Affordable, essential features |
| Arista | 7000 Series | Datacenter | High performance, cloud vision |
| Juniper | EX Series | Enterprise | Junos OS, virtual chassis |
| HPE | Aruba CX | Enterprise | Cloud-managed, AOS-CX |
| Dell | PowerSwitch | Enterprise | Open networking, SONiC support |
| Ubiquiti | UniFi | SMB | Cloud management, low cost |
| Netgear | M4300/M4500 | SMB/Enterprise | Affordable, full featured |

## Smart/Web-Managed Switches: The Middle Ground

### What Are Smart Switches?

**Smart switches** (also called "web-managed" or "lite-managed") bridge the gap between unmanaged and fully managed switches. They offer essential features with simpler configuration.

```
Feature Comparison:
┌──────────────────────┬──────────┬───────┬──────────┐
│ Feature              │Unmanaged │ Smart │ Managed  │
├──────────────────────┼──────────┼───────┼──────────┤
│ VLANs                │    ✗     │   ✓   │    ✓     │
│ QoS                  │    ✗     │   ✓   │    ✓     │
│ Port mirroring       │    ✗     │   ✓   │    ✓     │
│ SNMP                 │    ✗     │ Basic │   Full   │
│ CLI access           │    ✗     │   ✗   │    ✓     │
│ Advanced routing     │    ✗     │   ✗   │    ✓     │
│ Stacking             │    ✗     │   ✗   │    ✓     │
│ 802.1X               │    ✗     │Limited│   Full   │
│ Price (24-port)      │  $100    │  $250 │  $800+   │
└──────────────────────┴──────────┴───────┴──────────┘
```

### Smart Switch Advantages

**1. Simplified Management**
- Web GUI only (no complex CLI)
- Wizards for common tasks
- Pre-configured templates

**2. Essential Features Included**
- VLANs (usually up to 256)
- QoS (basic priority queuing)
- Port monitoring
- Link aggregation
- Basic security

**3. Cost-Effective**
- 50-70% cheaper than enterprise managed
- Feature-rich compared to unmanaged
- Good ROI for small-medium businesses

**4. Lower Complexity**
- Faster deployment
- Less training required
- Reduced misconfiguration risk

### Smart Switch Limitations

**1. No CLI Access**
- Can't script configurations
- Slower for bulk changes
- No standardized command set

**2. Limited Advanced Features**
- Basic ACLs (not full extended ACLs)
- Simple QoS (not granular DSCP mapping)
- No dynamic routing protocols
- Limited stacking (if any)

**3. Scalability Constraints**
- Typically max 48 ports
- No modular expansion
- Can't stack 10+ switches

**4. Reduced SNMP Functionality**
- Read-only MIBs common
- Limited configuration via SNMP
- Fewer OIDs exposed

### Ideal Use Cases for Smart Switches

**Perfect for:**
- **Growing SMBs**: 20-100 employees needing VLANs
- **Branch offices**: Remote sites with local IT
- **Educational institutions**: Schools needing basic segmentation
- **Healthcare clinics**: VLAN isolation without complexity
- **Retail locations**: QoS for VoIP, VLANs for PCI compliance

**Example deployment:**
```
Retail Store Network:
Internet ──► Router/Firewall
                │
                ▼
         Smart Switch (24-port)
         │    │    │    │
         ▼    ▼    ▼    ▼
      VLAN10 VLAN20 VLAN30 VLAN40
      (POS)  (Back  (Guest (VoIP
             Office) WiFi)  Phones)
```

### Popular Smart Switch Models

| Model | Ports | Features | Price | Best For |
|-------|-------|----------|-------|----------|
| Netgear GS308T | 8 | VLAN, QoS, LACP | $60 | Small office |
| TP-Link TL-SG2428P | 24+4 | PoE+, VLANs, QoS | $280 | SMB with PoE needs |
| Netgear GS724Tv4 | 24 | Smart managed | $180 | Budget SMB |
| Zyxel GS1900-24 | 24 | VLANs, QoS, PoE | $150 | Education |
| D-Link DGS-1210-52 | 48+4 | Full smart features | $400 | Large SMB |

## Power over Ethernet (PoE) Switches

### What is PoE?

**Power over Ethernet** delivers electrical power over Ethernet cables, eliminating separate power adapters for devices.

```
Traditional Setup:
Device ──AC Adapter──► Power Outlet
       └──Ethernet───► Switch

PoE Setup:
Device ──Ethernet/Power──► PoE Switch
(Single cable provides data + power)
```

### PoE Standards

| Standard | Power (Port) | Power (Device) | Use Cases |
|----------|-------------|----------------|-----------|
| **PoE (802.3af)** | 15.4W | 12.95W | VoIP phones, basic cameras |
| **PoE+ (802.3at)** | 30W | 25.5W | WiFi APs, PTZ cameras |
| **PoE++ (802.3bt Type 3)** | 60W | 51W | Videoconferencing, high-power APs |
| **PoE++ (802.3bt Type 4)** | 100W | 71W | LED lighting, thin clients |

**Passive PoE** (non-standard):
- **24V passive**: Ubiquiti devices (older models)
- **48V passive**: Some vendor-specific equipment
- **Warning**: Not compatible with standard PoE - can damage devices!

### PoE Power Budgets

**Critical concept**: Total power available across all ports

```
Example PoE Switch:
- 24 ports, PoE+ capable (30W each)
- Total theoretical: 24 × 30W = 720W
- Actual PoE budget: 370W (limited by power supply)

Reality:
- Can power ~12 devices at 30W each
- Or 24 devices at ~15W each
- Or mix of power levels up to 370W total

Planning tip: Sum all device power needs + 20% overhead
```

**Power budget table:**

| Switch Model | Ports | Standard | Total PoE Budget | Avg per Port |
|--------------|-------|----------|------------------|--------------|
| Netgear GS308P | 8 | PoE+ | 123W | 15.3W |
| Cisco CBS250-24P | 24 | PoE+ | 195W | 8.1W |
| UniFi Switch 24 PoE | 24 | PoE+ | 250W | 10.4W |
| Cisco C9300-48P | 48 | PoE+ | 740W | 15.4W |
| Aruba 6300M 48-port | 48 | PoE++ | 1440W | 30W |

### PoE Device Power Requirements

**Common devices:**
```
Low Power (< 15W - PoE 802.3af):
- VoIP desk phones: 5-7W
- Basic IP cameras: 4-8W
- IoT sensors: 2-5W
- Card readers: 3-6W

Medium Power (15-30W - PoE+ 802.3at):
- WiFi access points: 15-20W
- PTZ cameras: 20-25W
- Video phones: 12-15W
- Thin clients: 18-25W

High Power (30-60W - PoE++ Type 3):
- High-power APs (WiFi 6E): 30-40W
- Video conferencing systems: 40-55W
- Building automation: 35-50W
- Digital signage: 45-60W

Ultra Power (60-100W - PoE++ Type 4):
- LED lighting arrays: 60-90W
- Industrial equipment: 70-95W
- High-end PTZ cameras with heaters: 65-85W
```

### PoE Switch Selection Criteria

**1. Calculate Total Power Needs**
```
Example office deployment:
- 16× VoIP phones @ 7W = 112W
- 4× WiFi APs @ 20W = 80W
- 8× IP cameras @ 12W = 96W
────────────────────────────
Total: 288W + 20% overhead = 346W

Conclusion: Need switch with 350W+ PoE budget
```

**2. Consider Port Count and PoE Distribution**
- Do all ports need PoE? (PoE+ models cost more)
- Partial PoE (e.g., 24 ports, 16 PoE) often sufficient
- Uplink ports rarely need PoE

**3. Plan for Future Growth**
- WiFi 6E APs use more power than WiFi 5
- Camera resolution upgrades increase power
- Add 30% buffer for expansion

**4. Verify PoE Standard Compatibility**
- Don't buy 802.3af if devices need 802.3at
- Check vendor compatibility lists
- Test passive PoE vs. standard PoE

### PoE Management Features

**Advanced PoE switches offer:**
- **Per-port power limits**: Prevent one device from consuming all budget
- **Power prioritization**: Critical devices get power first during shortage
- **PoE scheduling**: Turn off power during off-hours (energy savings)
- **PoE monitoring**: Real-time power consumption per port
- **PoE reset**: Remote power cycling of frozen devices

**Example CLI configuration (Cisco):**
```
Switch(config)# interface gigabitethernet 0/5
Switch(config-if)# power inline priority high
Switch(config-if)# power inline max 15400
Switch(config-if)# power inline delay 5
```

## Enterprise vs. SMB Switches

### Feature Comparison

| Feature | SMB Switches | Enterprise Switches |
|---------|-------------|---------------------|
| **Port density** | 8-48 ports | 48-400+ ports (chassis) |
| **Stacking** | Limited (2-4) | Extensive (9-16 units) |
| **Power supply** | Single, internal | Redundant, hot-swap |
| **Fans** | Fixed | Redundant, hot-swap |
| **Warranty** | 1-5 years | Lifetime common |
| **CLI** | Basic/none | Full-featured |
| **Routing** | Static only | Dynamic (OSPF, BGP) |
| **Backplane** | 52-128 Gbps | 1-10 Tbps |
| **Stacking bandwidth** | 20-80 Gbps | 480 Gbps+ |
| **Management** | Web GUI | CLI, SNMP, automation |
| **Price (24-port)** | $200-$1,000 | $2,000-$10,000 |

### SMB Switch Characteristics

**Target audience**: Small-medium businesses, 10-500 employees

**Key features:**
- Simplified management (web GUI primary)
- Pre-configured templates
- Essential VLAN and QoS
- Affordable PoE options
- Desktop or rack-mount
- Limited redundancy

**Popular SMB lines:**
- Cisco Small Business (CBS/SG series)
- Netgear Pro/Smart series
- TP-Link Omada/JetStream
- Ubiquiti UniFi
- Zyxel GS/XGS series

**Typical deployment:**
```
Small Office (50 employees):
- 1× Core switch (48-port, PoE+, smart managed)
- 2× Access switches (24-port, PoE+)
- Total cost: ~$1,500
```

### Enterprise Switch Characteristics

**Target audience**: Large corporations, datacenters, campuses, 500+ employees

**Key features:**
- Full CLI management
- Modular chassis designs
- Redundant everything (PSU, fans, supervisors)
- Advanced routing protocols
- High availability (sub-second failover)
- Extensive stacking
- Enterprise support (TAC, 4-hour replacement)

**Popular enterprise lines:**
- Cisco Catalyst 9000 series
- Arista 7000 series
- Juniper EX/QFX series
- HPE Aruba CX series
- Dell PowerSwitch N-series

**Typical deployment:**
```
Enterprise Campus (2,000 employees):
- 2× Core chassis (redundant, 400+ Gbps)
- 20× Distribution switches (48-port, Layer 3)
- 100× Access switches (48-port, PoE++)
- Total cost: $500,000-$2M
```

### Decision Framework: SMB vs. Enterprise

**Choose SMB switches if:**
- < 500 employees
- Single location or few small branches
- Budget-conscious
- Limited IT staff
- Standard applications (email, web, file sharing)
- Can tolerate 4-hour recovery times

**Choose Enterprise switches if:**
- 500+ employees or rapid growth
- Multiple locations requiring standardization
- Mission-critical uptime required
- Dedicated network team
- Advanced requirements (BGP, MPLS, automation)
- Regulatory compliance mandates
- Need vendor support (TAC, professional services)

## Switch Selection Criteria

### 1. Port Count Planning

**Formula**: Current devices + 30% growth + uplinks

```
Example calculation:
- 40 current workstations
- 8 printers/devices
- 30% growth = 14 ports
- 2 uplinks to distribution
─────────────────────────
Total: 64 ports needed

Solution: 2× 48-port switches (stacked)
          or 1× 48-port + 1× 24-port
```

**Growth patterns:**
- Small office: +20% over 3 years
- Growing company: +50% over 3 years
- Stable company: +10% over 5 years

### 2. Speed Requirements

| Use Case | Access Ports | Uplink Ports |
|----------|-------------|--------------|
| Basic office | 1 Gbps | 1-10 Gbps |
| VoIP + data | 1 Gbps | 10 Gbps |
| Engineering (CAD) | 1-2.5 Gbps | 10 Gbps |
| Video production | 10 Gbps | 40 Gbps |
| Virtualization hosts | 10-25 Gbps | 40-100 Gbps |
| Datacenter storage | 25-100 Gbps | 100-400 Gbps |

### 3. PoE Requirements

**Audit all PoE devices:**
```
Device Type       | Quantity | Watts Each | Total
------------------|----------|------------|-------
VoIP phones       |    30    |     7W     |  210W
WiFi APs (WiFi 6) |     6    |    20W     |  120W
IP cameras        |    12    |    10W     |  120W
                                    Total:  450W
                          + 20% buffer:  540W

Recommendation: Switch with 600W PoE budget
```

### 4. Management Needs

**Decision tree:**
```
Do you need VLANs?
├─ NO  → Unmanaged switch
└─ YES → Do you need CLI/scripting?
          ├─ NO  → Smart/web-managed switch
          └─ YES → Fully managed switch
```

### 5. Physical Environment

**Considerations:**
- **Noise tolerance**: Fanless for offices, loud fans OK for datacenter
- **Temperature**: Industrial switches for extreme environments
- **Mounting**: Desktop, wall-mount, or 19" rack
- **Depth**: Full-depth (15"+) or shallow (10") for limited rack space

### 6. Budget Analysis

**Total Cost of Ownership (TCO) over 5 years:**

```
Unmanaged Switch:
- Purchase: $150
- Management: $0 (no config)
- Downtime: $500 (no monitoring)
────────────────────────────
TCO: $650

Managed Switch:
- Purchase: $800
- Management: $200/year × 5 = $1,000
- Downtime: $100 (proactive monitoring)
────────────────────────────
TCO: $1,900

Enterprise Switch:
- Purchase: $4,000
- Support contract: $600/year × 5 = $3,000
- Management: $300/year × 5 = $1,500
- Downtime: $50 (high availability)
────────────────────────────
TCO: $8,550
```

**ROI justification:**
- Downtime cost: Calculate hourly revenue loss
- Productivity: VLANs reduce broadcast storms
- Security: Compliance fines avoidance
- Scalability: Avoid rip-and-replace

### 7. Vendor Ecosystem

**Questions:**
- **Existing infrastructure**: Standardize on current vendor?
- **Support**: Local VAR (Value-Added Reseller) availability?
- **Training**: Staff knowledge of vendor CLI?
- **Interoperability**: Multi-vendor or single-vendor shop?

## Selection Decision Matrix

### Quick Reference Table

| Scenario | Recommended Switch | Rationale |
|----------|-------------------|-----------|
| Home network (5-8 devices) | 8-port unmanaged Gigabit | Cost-effective, plug-and-play |
| Small office (20 users) | 24-port smart switch | VLANs for printers, no CLI needed |
| Office with VoIP (30 users) | 48-port PoE+ managed | QoS for voice, PoE for phones |
| Branch office | 24-port Layer 3 switch | Inter-VLAN routing locally |
| Datacenter rack (20 servers) | 48-port 10GbE managed | High-speed server connectivity |
| Campus building | Stackable 48-port PoE++ | WiFi 6E APs, unified management |
| ISP/datacenter core | Chassis-based 100GbE | Massive throughput, redundancy |

## Key Takeaways

1. **Unmanaged switches**: Best for simple, small networks where plug-and-play suffices. No configuration means no complexity but also no control.

2. **Smart switches**: Sweet spot for SMBs needing VLANs and QoS without CLI complexity. Web GUI keeps management accessible.

3. **Managed switches**: Essential for enterprises requiring full control, advanced features, scripting, and scalability.

4. **PoE is a force multiplier**: Calculate power budgets carefully—total device wattage plus 20% buffer. Don't forget future WiFi/camera upgrades.

5. **SMB vs. Enterprise**: Not just about features—consider support, redundancy, warranty, and total cost of ownership.

6. **Right-size your investment**: Over-buying wastes budget; under-buying causes painful upgrades. Plan for 3-5 year lifecycle.

7. **Selection framework**: Port count → Speed → PoE → Management → Budget → Vendor ecosystem

## What's Next?

In Chapter 4, we'll get hands-on with switch configuration. You'll learn how to access switches via console, navigate the CLI, set up hostnames and passwords, configure ports, and save configurations. We'll focus on practical, real-world setup procedures you'll use daily.

---

**Chapter 3 Complete** | Next: Chapter 4 - Basic Switch Configuration
