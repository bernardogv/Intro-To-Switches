# Chapter 10: Layer 3 Switching and Inter-VLAN Routing

## Introduction

Up to this point, we've focused on Layer 2 switching—forwarding frames based on MAC addresses within VLANs. But what happens when devices in different VLANs need to communicate? Traditionally, this required routing through an external router. Layer 3 switches combine the speed of switching with the intelligence of routing, performing inter-VLAN routing at wire speed using specialized ASIC hardware.

This chapter explores how Layer 3 switches work, when to use them versus traditional routers, and how to configure routing protocols and advanced features for optimal network performance.

---

## 10.1 Understanding Layer 3 Switching

### The Evolution from Router-on-a-Stick

**Traditional Inter-VLAN Routing (Router-on-a-Stick):**

```
VLAN 10 (10.1.10.0/24) ──┐
                          ├──► Layer 2 Switch ──► Router ──► VLAN 20
VLAN 20 (10.1.20.0/24) ──┘      (Trunk)          (Routing)

Problem: All inter-VLAN traffic must traverse the router
         Bandwidth bottleneck
         Higher latency (multiple hops)
```

**Layer 3 Switch Solution:**

```
VLAN 10 (10.1.10.0/24) ──┐
                          ├──► Layer 3 Switch (Routing + Switching)
VLAN 20 (10.1.20.0/24) ──┘      Wire-speed performance
                                 No external router needed
```

### How Layer 3 Switches Work

**Hardware vs. Software Routing:**

1. **Traditional Router:**
   - CPU processes routing decisions
   - Software-based forwarding
   - Throughput: 100 Mbps - 10 Gbps
   - Latency: microseconds

2. **Layer 3 Switch:**
   - ASIC (Application-Specific Integrated Circuit) hardware
   - Hardware-based forwarding
   - Throughput: Line-rate (all ports at full speed)
   - Latency: nanoseconds

**Packet Flow in Layer 3 Switch:**

```
1. Frame arrives on port
2. Layer 2 lookup: Check destination MAC
   ├── Local VLAN? → Forward at Layer 2
   └── Router MAC? → Process at Layer 3
3. Layer 3 lookup: Check destination IP in routing table
4. ASIC performs routing decision (hardware acceleration)
5. Rewrite MAC header (new source/dest MAC)
6. Forward to destination port at wire speed
```

### When to Use Layer 3 Switches

**✅ Use Layer 3 Switches For:**
- Inter-VLAN routing within a campus or data center
- High-speed routing between subnets
- Distribution layer in hierarchical networks
- Cost-effective alternative to routers for internal traffic
- Environments requiring line-rate performance

**❌ Use Traditional Routers For:**
- WAN connections (Internet, MPLS, VPN)
- Complex routing protocols (full BGP tables)
- Advanced security features (firewall, deep packet inspection)
- Network Address Translation (NAT)
- Traffic shaping and policy-based routing

---

## 10.2 Configuring Layer 3 Interfaces

### Switched Virtual Interfaces (SVIs)

SVIs are virtual Layer 3 interfaces associated with VLANs. They serve as the default gateway for devices in that VLAN.

**Configuration Example:**

```cisco
! Enable IP routing globally
Switch(config)# ip routing

! Create VLAN
Switch(config)# vlan 10
Switch(config-vlan)# name Engineering
Switch(config-vlan)# exit

! Create SVI for VLAN 10
Switch(config)# interface vlan 10
Switch(config-if)# ip address 10.1.10.1 255.255.255.0
Switch(config-if)# description Gateway for Engineering VLAN
Switch(config-if)# no shutdown
Switch(config-if)# exit

! Assign ports to VLAN
Switch(config)# interface range gigabitethernet0/1-10
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
```

**Multiple VLANs Example:**

```cisco
! VLAN 10 - Engineering
interface vlan 10
 ip address 10.1.10.1 255.255.255.0
 no shutdown

! VLAN 20 - Sales
interface vlan 20
 ip address 10.1.20.1 255.255.255.0
 no shutdown

! VLAN 30 - Management
interface vlan 30
 ip address 10.1.30.1 255.255.255.0
 no shutdown
```

**Verification:**

```cisco
Switch# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
Vlan10                 10.1.10.1       YES manual up                    up
Vlan20                 10.1.20.1       YES manual up                    up
Vlan30                 10.1.30.1       YES manual up                    up

Switch# show ip route
Codes: C - connected, S - static, R - RIP, O - OSPF

Gateway of last resort is not set

C    10.1.10.0/24 is directly connected, Vlan10
C    10.1.20.0/24 is directly connected, Vlan20
C    10.1.30.0/24 is directly connected, Vlan30
```

### Routed Ports (Layer 3 Physical Interfaces)

Convert a physical switch port to function as a router interface.

**Configuration:**

```cisco
Switch(config)# interface gigabitethernet0/24
Switch(config-if)# no switchport
Switch(config-if)# ip address 192.168.100.1 255.255.255.0
Switch(config-if)# description Uplink to Core Router
Switch(config-if)# no shutdown
```

**Use Cases for Routed Ports:**
- Point-to-point links between switches
- Connections to routers
- WAN interfaces
- Server farm connectivity

**Verification:**

```cisco
Switch# show interfaces gigabitethernet0/24
GigabitEthernet0/24 is up, line protocol is up
  Hardware is Gigabit Ethernet, address is 0050.56be.0024
  Internet address is 192.168.100.1/24

Switch# show ip interface gigabitethernet0/24
GigabitEthernet0/24 is up, line protocol is up
  Internet address is 192.168.100.1/24
  Broadcast address is 255.255.255.255
  MTU is 1500 bytes
  Helper address is not set
  Directed broadcast forwarding is disabled
```

---

## 10.3 Static Routing on Layer 3 Switches

### Basic Static Routes

```cisco
! Syntax: ip route <destination network> <subnet mask> <next-hop IP>

! Route to remote network 172.16.0.0/16 via 192.168.100.254
Switch(config)# ip route 172.16.0.0 255.255.0.0 192.168.100.254

! Default route (all unknown traffic)
Switch(config)# ip route 0.0.0.0 0.0.0.0 192.168.100.254
```

### Static Routes with Exit Interface

```cisco
! Using next-hop IP
Switch(config)# ip route 172.16.0.0 255.255.0.0 192.168.100.254

! Using exit interface (preferred for point-to-point links)
Switch(config)# ip route 172.16.0.0 255.255.0.0 gigabitethernet0/24

! Using both (most specific)
Switch(config)# ip route 172.16.0.0 255.255.0.0 gigabitethernet0/24 192.168.100.254
```

### Floating Static Routes (Backup Routes)

```cisco
! Primary route via Gi0/24 (AD 1, default for static routes)
Switch(config)# ip route 0.0.0.0 0.0.0.0 gigabitethernet0/24 192.168.100.1

! Backup route via Gi0/23 (AD 10, higher than primary)
Switch(config)# ip route 0.0.0.0 0.0.0.0 gigabitethernet0/23 192.168.100.2 10
```

**Administrative Distance (AD):**
- Connected: 0
- Static: 1
- EIGRP: 90
- OSPF: 110
- RIP: 120

Lower AD is preferred. Floating static routes use higher AD to serve as backups.

### Verification

```cisco
Switch# show ip route
Gateway of last resort is 192.168.100.254 to network 0.0.0.0

S*   0.0.0.0/0 [1/0] via 192.168.100.254
C    10.1.10.0/24 is directly connected, Vlan10
C    10.1.20.0/24 is directly connected, Vlan20
C    192.168.100.0/24 is directly connected, GigabitEthernet0/24
S    172.16.0.0/16 [1/0] via 192.168.100.254

Switch# show ip route static
S*   0.0.0.0/0 [1/0] via 192.168.100.254
S    172.16.0.0/16 [1/0] via 192.168.100.254
```

---

## 10.4 Dynamic Routing Protocols

### OSPF (Open Shortest Path First)

**Why OSPF on Switches:**
- Link-state protocol (knows entire topology)
- Fast convergence
- Scalable to large networks
- Industry standard (RFC 2328)

**Basic OSPF Configuration:**

```cisco
! Enable OSPF process 1
Switch(config)# router ospf 1
Switch(config-router)# router-id 1.1.1.1

! Advertise connected networks
Switch(config-router)# network 10.1.10.0 0.0.0.255 area 0
Switch(config-router)# network 10.1.20.0 0.0.0.255 area 0
Switch(config-router)# network 192.168.100.0 0.0.0.255 area 0
```

**OSPF Areas Explained:**

```
Area 0 (Backbone) ──┬──► Area 1 (Engineering)
                    ├──► Area 2 (Sales)
                    └──► Area 3 (Data Center)

All areas must connect to Area 0
```

**Advanced OSPF Configuration:**

```cisco
router ospf 1
 router-id 10.1.1.1
 log-adjacency-changes
 passive-interface default
 no passive-interface gigabitethernet0/24
 network 10.1.0.0 0.0.255.255 area 0
 network 192.168.100.0 0.0.0.3 area 0

! Set reference bandwidth for 10G links
 auto-cost reference-bandwidth 10000

! Tune timers (default: hello 10s, dead 40s)
interface vlan 10
 ip ospf hello-interval 5
 ip ospf dead-interval 20
 ip ospf priority 100
```

**Verification:**

```cisco
Switch# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.1.1.2        1     FULL/BDR        00:00:35    192.168.100.2   Gi0/24

Switch# show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Vl10         1     0               10.1.10.1/24       10    DR    0/0
Vl20         1     0               10.1.20.1/24       10    DR    0/0
Gi0/24       1     0               192.168.100.1/30   1     BDR   1/1

Switch# show ip route ospf
O    172.16.0.0/16 [110/2] via 192.168.100.2, 00:10:25, GigabitEthernet0/24
O    10.2.0.0/16 [110/2] via 192.168.100.2, 00:10:25, GigabitEthernet0/24
```

### EIGRP (Enhanced Interior Gateway Routing Protocol)

**Cisco Proprietary (Legacy) / Open Standard (Named Mode):**

```cisco
! Classic EIGRP configuration
Switch(config)# router eigrp 100
Switch(config-router)# network 10.1.0.0
Switch(config-router)# network 192.168.100.0 0.0.0.3
Switch(config-router)# no auto-summary
Switch(config-router)# eigrp router-id 1.1.1.1
```

**Named EIGRP (Modern Approach):**

```cisco
Switch(config)# router eigrp CAMPUS
Switch(config-router)# address-family ipv4 unicast autonomous-system 100
Switch(config-router-af)# network 10.1.0.0
Switch(config-router-af)# network 192.168.100.0 0.0.0.3
Switch(config-router-af)# eigrp router-id 1.1.1.1
Switch(config-router-af)# af-interface default
Switch(config-router-af-interface)# passive-interface
Switch(config-router-af-interface)# exit
Switch(config-router-af)# af-interface gigabitethernet0/24
Switch(config-router-af-interface)# no passive-interface
```

**Verification:**

```cisco
Switch# show ip eigrp neighbors
IP-EIGRP neighbors for process 100
H   Address         Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                    (sec)         (ms)       Cnt Num
0   192.168.100.2   Gi0/24          13   01:20:15 10     200  0  45

Switch# show ip route eigrp
D    172.16.0.0/16 [90/3072] via 192.168.100.2, 01:20:15, GigabitEthernet0/24
```

### Route Redistribution

Connecting OSPF and EIGRP networks:

```cisco
! Redistribute EIGRP into OSPF
router ospf 1
 redistribute eigrp 100 subnets metric 100

! Redistribute OSPF into EIGRP
router eigrp 100
 redistribute ospf 1 metric 10000 100 255 1 1500
```

**Caution:** Redistribution can cause routing loops. Use route filtering and tagging.

---

## 10.5 Advanced Layer 3 Features

### Policy-Based Routing (PBR)

Route traffic based on criteria beyond destination IP (source, protocol, port).

**Use Case:** Route guest traffic to separate Internet link

```cisco
! Create ACL to match guest traffic
Switch(config)# ip access-list extended GUEST-TRAFFIC
Switch(config-ext-nacl)# permit ip 10.1.99.0 0.0.0.255 any

! Create route map
Switch(config)# route-map GUEST-TO-ISP2 permit 10
Switch(config-route-map)# match ip address GUEST-TRAFFIC
Switch(config-route-map)# set ip next-hop 203.0.113.1

! Apply to interface
Switch(config)# interface vlan 99
Switch(config-if)# ip policy route-map GUEST-TO-ISP2
```

### VRF (Virtual Routing and Forwarding)

Separate routing tables within a single switch—like VLANs for routing.

**Use Case:** Multi-tenant data center with customer isolation

```cisco
! Create VRF instances
Switch(config)# ip vrf CUSTOMER-A
Switch(config-vrf)# rd 100:1
Switch(config-vrf)# exit

Switch(config)# ip vrf CUSTOMER-B
Switch(config-vrf)# rd 100:2
Switch(config-vrf)# exit

! Assign interfaces to VRF
Switch(config)# interface vlan 10
Switch(config-if)# ip vrf forwarding CUSTOMER-A
Switch(config-if)# ip address 10.1.10.1 255.255.255.0

Switch(config)# interface vlan 20
Switch(config-if)# ip vrf forwarding CUSTOMER-B
Switch(config-if)# ip address 10.1.20.1 255.255.255.0

! Routing within VRF
Switch(config)# router ospf 10 vrf CUSTOMER-A
Switch(config-router)# network 10.1.10.0 0.0.0.255 area 0
```

**Verification:**

```cisco
Switch# show ip vrf
  Name                             Default RD          Interfaces
  CUSTOMER-A                       100:1               Vl10
  CUSTOMER-B                       100:2               Vl20

Switch# show ip route vrf CUSTOMER-A
C    10.1.10.0/24 is directly connected, Vlan10
```

### HSRP/VRRP: First-Hop Redundancy

Provide gateway redundancy for high availability.

**HSRP (Hot Standby Router Protocol) - Cisco Proprietary:**

```cisco
! Switch 1 (Primary)
interface vlan 10
 ip address 10.1.10.2 255.255.255.0
 standby 1 ip 10.1.10.1
 standby 1 priority 110
 standby 1 preempt

! Switch 2 (Backup)
interface vlan 10
 ip address 10.1.10.3 255.255.255.0
 standby 1 ip 10.1.10.1
 standby 1 priority 100
```

**VRRP (Virtual Router Redundancy Protocol) - Open Standard:**

```cisco
! Switch 1
interface vlan 10
 ip address 10.1.10.2 255.255.255.0
 vrrp 1 ip 10.1.10.1
 vrrp 1 priority 110
 vrrp 1 preempt
```

**Virtual IP:** 10.1.10.1 (clients use as default gateway)
**Physical IPs:** 10.1.10.2, 10.1.10.3 (actual switch IPs)

**How it works:**
1. Active switch responds to ARP for 10.1.10.1
2. Standby switch monitors active via hello packets
3. If active fails, standby takes over in ~3 seconds
4. Clients continue using 10.1.10.1 (no reconfiguration)

---

## 10.6 Performance Tuning and Optimization

### IP Routing Cache (CEF - Cisco Express Forwarding)

**Traditional Process Switching (Slow):**
```
Every packet → CPU → Routing table lookup → Forward
```

**CEF (Fast):**
```
First packet → CPU → Lookup → Create CEF entry → Hardware forwarding
Subsequent packets → ASIC → CEF table → Forward (nanoseconds)
```

**Enable CEF (usually default):**

```cisco
Switch(config)# ip cef
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# ip route-cache cef
```

**Verification:**

```cisco
Switch# show ip cef
Prefix              Next Hop             Interface
0.0.0.0/0           192.168.100.254      GigabitEthernet0/24
10.1.10.0/24        attached             Vlan10
10.1.20.0/24        attached             Vlan20

Switch# show cef adjacency gigabitethernet0/24 detail
```

### Load Balancing

**Per-Destination (Default):**
```cisco
Switch(config)# ip cef load-sharing algorithm original
```
- Same source/dest always takes same path
- Better for maintaining TCP session order

**Per-Packet:**
```cisco
Switch(config)# ip load-sharing per-packet
```
- Round-robin across equal-cost paths
- Can cause out-of-order packets

**Universal (Hash-Based):**
```cisco
Switch(config)# ip cef load-sharing algorithm universal <id>
```
- Improved load distribution
- Maintains packet order within flows

---

## 10.7 Troubleshooting Layer 3 Switching

### Connectivity Issues

**Problem: Devices in different VLANs can't communicate**

```cisco
! Step 1: Verify IP routing is enabled
Switch# show ip route
% IP routing table does not exist

! Solution:
Switch(config)# ip routing

! Step 2: Check SVI status
Switch# show ip interface brief | include Vlan
Vlan10    10.1.10.1   YES manual administratively down  down

! Solution: Ensure VLAN exists and SVI is up
Switch(config)# interface vlan 10
Switch(config-if)# no shutdown

! Step 3: Verify routing table
Switch# show ip route
C    10.1.10.0/24 is directly connected, Vlan10
C    10.1.20.0/24 is directly connected, Vlan20

! Step 4: Test connectivity from switch
Switch# ping 10.1.20.10 source 10.1.10.1
```

### Routing Protocol Problems

**OSPF Neighbors Not Forming:**

```cisco
! Check neighbor status
Switch# show ip ospf neighbor

! Verify OSPF is running on interfaces
Switch# show ip ospf interface brief

! Common issues:
! 1. Mismatched area numbers
Switch# show run | section router ospf

! 2. Passive interface enabled
Switch(config)# router ospf 1
Switch(config-router)# no passive-interface gigabitethernet0/24

! 3. MTU mismatch
Switch(config)# interface gigabitethernet0/24
Switch(config-if)# ip mtu 1500

! 4. Access list blocking
Switch# show ip access-lists
```

### Performance Degradation

```cisco
! Check CEF switching
Switch# show cef linecard detail

! Verify hardware resources
Switch# show platform hardware capacity

! Check for software switching (bad sign)
Switch# show process cpu sorted
PID  Runtime(ms)  Invoked  uSecs   5Sec   1Min   5Min TTY Process
...
If "IP Input" is high → packets being software-switched
```

---

## Key Takeaways

1. **Layer 3 switches** combine switching speed with routing intelligence using ASIC hardware
2. **SVIs** provide Layer 3 interfaces for VLANs; use routed ports for point-to-point links
3. **Static routing** is simple but doesn't scale; use dynamic protocols (OSPF, EIGRP) for larger networks
4. **OSPF** is the industry-standard link-state protocol; EIGRP is Cisco-enhanced distance-vector
5. **Advanced features** like VRF, PBR, and HSRP enable sophisticated routing scenarios
6. **CEF** enables hardware-based forwarding at wire speed
7. **Troubleshooting** requires verifying configuration, routing tables, and protocol states

Layer 3 switching is essential for modern networks, providing the performance of switching with the flexibility of routing at a fraction of the cost of traditional routers.

---

**Chapter 10 Complete** | Next: Chapter 11 - Switch Monitoring and Troubleshooting
