# Chapter 14: Data Center Switching

## Introduction

Data center switches operate in a fundamentally different environment than campus or enterprise switches. They must support massive east-west traffic (server-to-server), ultra-low latency for distributed applications, non-blocking architectures for storage networks, and seamless integration with virtualization platforms and cloud orchestration systems.

This chapter explores spine-leaf architectures, overlay networks (VXLAN), data center protocols (FCoE, iSCSI), and the unique challenges of building high-performance, scalable data center fabrics.

---

## 14.1 Data Center Architecture Evolution

### Traditional Three-Tier Architecture

```
                    [Core Layer]
                    (Routing, WAN)
                         │
                ┌────────┴────────┐
                ▼                 ▼
         [Aggregation]      [Aggregation]
         (Distribution)     (Distribution)
                │                 │
        ┌───────┼───────┐   ┌─────┼──────┐
        ▼       ▼       ▼   ▼     ▼      ▼
     [ToR]   [ToR]   [ToR] [ToR] [ToR] [ToR]
    (Access) (Access) (Access)
```

**Problems:**
- **Oversubscription:** Limited uplink bandwidth (20:1 or worse)
- **STP blocking:** 50% of links unused
- **North-South optimized:** Poor for server-to-server traffic
- **Scalability limits:** Adding capacity requires forklift upgrades

### Modern Spine-Leaf (Clos) Architecture

```
        [Spine 1]  [Spine 2]  [Spine 3]  [Spine 4]
            │ \      / │  \      /  │  \     /  │
            │   \  /   │    \  /    │    \ /    │
            │     X    │      X     │      X    │
            │   /  \   │    /  \    │    /  \   │
            │ /      \ │  /      \  │  /      \ │
         [Leaf 1]  [Leaf 2]  [Leaf 3]  [Leaf 4]
           │││        │││        │││        │││
         [Servers]  [Servers]  [Servers]  [Servers]
```

**Characteristics:**
- Every leaf connects to every spine (full mesh)
- All server-to-server paths equal length (consistent latency)
- Easy to scale: Add leaf switches for more servers, add spines for more bandwidth
- No Spanning Tree (uses Layer 3 routing or VXLAN)

**Benefits:**
- **Predictable performance:** Any server to any server = 1 hop through spine
- **Non-blocking:** Can achieve 1:1 oversubscription
- **Horizontal scaling:** Add capacity incrementally
- **Redundancy:** Multiple equal-cost paths

---

## 14.2 Spine-Leaf Design Principles

### Underlay: The Physical Network

**Layer 3 Underlay (Most Common):**

```cisco
! Leaf Switch Configuration
interface Ethernet1/1
 description to-Spine-1
 no switchport
 ip address 10.0.1.1/31
 ip ospf network point-to-point
 ip ospf area 0.0.0.0

interface Ethernet1/2
 description to-Spine-2
 no switchport
 ip address 10.0.2.1/31
 ip ospf network point-to-point
 ip ospf area 0.0.0.0

! Enable OSPF for underlay routing
router ospf 1
 router-id 10.255.255.1
 passive-interface default
 no passive-interface Ethernet1/1
 no passive-interface Ethernet1/2
```

**Why Layer 3?**
- Equal-Cost Multi-Path (ECMP) routing
- Fast convergence (sub-second with BFD)
- No Spanning Tree overhead
- Scalable to thousands of switches

### Overlay: VXLAN for Layer 2 Extension

**Problem:** Applications need Layer 2 connectivity across the fabric (VM mobility, clusters)

**VXLAN Solution:** Encapsulates Layer 2 frames in UDP/IP packets

```
Original Frame:
[Eth Header][IP][TCP][Data]

VXLAN Encapsulated:
[Outer IP][UDP][VXLAN Header][Original Eth Header][IP][TCP][Data]
                   ↑
            24-bit VNI (16 million networks vs 4096 VLANs)
```

**Configuration Example (Cisco Nexus):**

```cisco
! Enable VXLAN feature
feature nv overlay
feature vn-segment-vlan-based

! Create VLAN and map to VXLAN
vlan 10
  vn-segment 10010

! Configure VTEP (VXLAN Tunnel Endpoint)
interface nve1
  no shutdown
  source-interface loopback0
  member vni 10010
    ingress-replication protocol bgp

! BGP EVPN for control plane
router bgp 65001
  neighbor 10.255.255.100 remote-as 65000
  address-family l2vpn evpn
    send-community extended
    advertise-pip
```

**Benefits:**
- 16 million logical networks (vs 4096 VLANs)
- Layer 2 over Layer 3 (VM mobility)
- Multi-tenancy (network segmentation)
- Optimal traffic paths (no STP blocking)

---

## 14.3 Data Center Protocols

### EVPN (Ethernet VPN): Control Plane for VXLAN

**Traditional VXLAN:** Flood-and-learn (like MAC learning in switches)
**EVPN:** Centralized control plane using BGP

```cisco
! EVPN BGP configuration
router bgp 65001
  router-id 10.255.255.1
  neighbor 10.255.255.100 remote-as 65000
  address-family l2vpn evpn
    send-community extended
    route-reflector-client

! Advertise MAC addresses via BGP
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
```

**EVPN Features:**
- MAC address distribution (no flooding)
- ARP suppression (reduces broadcast)
- Multi-homing with active-active
- Fast convergence

### FCoE (Fibre Channel over Ethernet)

**Use Case:** Converged network for storage and data

**Traditional SAN:**
```
Servers ──[Ethernet]──► Ethernet Switches
       └─[Fibre Channel]──► FC Switches ──► Storage
```

**FCoE Converged:**
```
Servers ──[FCoE]──► Converged Switches ──► Storage
    (Ethernet + Storage on same cable)
```

**Configuration (Cisco Nexus):**

```cisco
! Enable FCoE
feature fcoe

! Create VSAN (like VLAN for storage)
vsan database
  vsan 10 name "Production-Storage"

! Configure FCoE VLAN
vlan 1000
  fcoe vsan 10
  name FCoE-VLAN

! Map interface to VSAN
interface ethernet 1/10
  switchport mode trunk
  switchport trunk allowed vlan 1000
  spanning-tree port type edge trunk
```

### iSCSI: IP-Based Storage

**Alternative to FCoE:** Uses standard Ethernet (no special hardware)

**Configuration:**

```cisco
! Dedicated iSCSI VLAN
vlan 100
  name iSCSI-Storage

! Enable Jumbo Frames (critical for performance)
interface ethernet 1/5-10
  mtu 9216
  switchport access vlan 100

! QoS for iSCSI (prioritize storage traffic)
class-map match-all iSCSI
  match access-group name iSCSI-ACL

policy-map iSCSI-Policy
  class iSCSI
    set dscp 26
    priority level 1

interface ethernet 1/5
  service-policy input iSCSI-Policy
```

---

## 14.4 Data Center Performance Optimization

### Low Latency Switching

**Cut-Through vs Store-and-Forward:**

**Store-and-Forward (Traditional):**
```
Receive entire frame → Check CRC → Forward
Latency: 10-30 microseconds
```

**Cut-Through:**
```
Read destination MAC → Immediately forward (while still receiving)
Latency: < 1 microsecond
```

**Configuration:**

```cisco
! Enable cut-through switching
system switching-mode cut-through

! Or per-interface
interface ethernet 1/1
  switching-mode cut-through
```

**Trade-off:** Cut-through may forward corrupt frames (no CRC check until after forwarding)

### Buffer Management

**Shared vs Dedicated Buffers:**

```cisco
! Nexus 9000: Dynamic buffer allocation
hardware qos queuing mode priority

! Monitor buffer usage
show hardware internal buffer info detail

! Adjust buffer allocation for specific traffic
policy-map type network-qos buffer-policy
  class type network-qos class-default
    mtu 9216
    queue-limit 50000 bytes
```

### ECMP Load Balancing

**Hash-Based Load Balancing:**

```cisco
! Configure ECMP hash algorithm
port-channel load-balance ethernet source-dest-ip
port-channel load-balance ethernet source-dest-port

! Verify ECMP paths
show ip route 172.16.0.0/24
  [110/41] via 10.0.1.0, Ethernet1/1
  [110/41] via 10.0.2.0, Ethernet1/2
  [110/41] via 10.0.3.0, Ethernet1/3
  [110/41] via 10.0.4.0, Ethernet1/4

! 4 equal-cost paths = traffic load balanced
```

---

## 14.5 Data Center QoS and Congestion Management

### Priority Flow Control (PFC)

**Problem:** Lossless Ethernet required for FCoE and RDMA

**PFC Solution:** Per-class pause frames

```cisco
! Enable PFC on interface
interface ethernet 1/10
  priority-flow-control mode on
  priority-flow-control watch-dog-interval 200

! Configure no-drop class
system qos
  service-policy type network-qos pfc-policy

class-map type network-qos match-all no-drop
  match qos-group 1

policy-map type network-qos pfc-policy
  class no-drop
    pause pfc-cos 3
    mtu 9216
```

### ECN (Explicit Congestion Notification)

**Alternative to packet drops:** Mark packets instead of dropping

```cisco
! Enable ECN
policy-map type queuing ecn-policy
  class type queuing class-default
    random-detect minimum-threshold 100 kbytes maximum-threshold 500 kbytes

interface ethernet 1/1
  service-policy type queuing output ecn-policy
```

### Data Center Bridging (DCB)

**Three pillars:**
1. **PFC:** Lossless Ethernet
2. **ETS (Enhanced Transmission Selection):** Bandwidth allocation
3. **DCBX (Data Center Bridging Exchange):** Auto-negotiation

```cisco
! Enable DCB
feature lldp

system qos
  service-policy type network-qos dcb-policy

class-map type network-qos match-all storage
  match qos-group 1

class-map type network-qos match-all management
  match qos-group 2

policy-map type network-qos dcb-policy
  class storage
    pause pfc-cos 3
    mtu 9216
    bandwidth percent 50
  class management
    mtu 1500
    bandwidth percent 10
```

---

## 14.6 Multi-Tenancy and Network Virtualization

### VRF-Lite for Tenant Isolation

**Separate routing tables per customer:**

```cisco
! Create VRFs
vrf context CUSTOMER-A
  rd auto
  address-family ipv4 unicast

vrf context CUSTOMER-B
  rd auto
  address-family ipv4 unicast

! Assign interfaces to VRF
interface vlan 100
  vrf member CUSTOMER-A
  ip address 10.10.1.1/24

interface vlan 200
  vrf member CUSTOMER-B
  ip address 10.20.1.1/24

! Routing per VRF
router ospf 1
  vrf CUSTOMER-A
    router-id 1.1.1.1

router ospf 2
  vrf CUSTOMER-B
    router-id 2.2.2.2
```

### VXLAN Multi-Tenancy

**VNI per tenant:**

```cisco
! Tenant A: VNI 100000-199999
vlan 10
  vn-segment 100010
vlan 20
  vn-segment 100020

! Tenant B: VNI 200000-299999
vlan 10
  vn-segment 200010
vlan 20
  vn-segment 200020

! Same VLAN ID (10, 20) but different VNIs = complete isolation
```

---

## 14.7 Automation and Orchestration

### Intent-Based Networking (IBN)

**Declarative configuration:**

```yaml
# Ansible playbook for data center fabric
- name: Configure Spine-Leaf Fabric
  hosts: spines:leafs
  tasks:
    - name: Enable OSPF
      nxos_feature:
        feature: ospf
        state: enabled

    - name: Configure Underlay Interfaces
      nxos_interfaces:
        config:
          - name: Ethernet1/1
            description: "to-{{ inventory_hostname }}"
            mode: layer3

    - name: Configure OSPF
      nxos_ospf_vrf:
        ospf: 1
        router_id: "{{ loopback0_ip }}"
```

### Telemetry and Analytics

**Streaming Telemetry:**

```cisco
! Enable model-driven telemetry
feature telemetry

! Create sensor group (what to monitor)
telemetry
  sensor-group interface-stats
    path sys/intf/phys-[eth1/1]/dbgIfIn
    path sys/intf/phys-[eth1/1]/dbgIfOut

! Create destination group (where to send)
  destination-group monitoring-server
    ip address 192.168.100.10 port 5000
    protocol gRPC encoding GPB

! Create subscription (sensor + destination)
  subscription 1
    snsr-grp interface-stats sample-interval 5000
    dst-grp monitoring-server
```

---

## 14.8 Top-of-Rack (ToR) vs End-of-Row (EoR)

### ToR (Top-of-Rack) - Modern Standard

```
Rack 1: [ToR Switch] ──► [Spine]
        └─ Servers

Rack 2: [ToR Switch] ──► [Spine]
        └─ Servers
```

**Pros:**
- Short cable runs (< 10m)
- Easier troubleshooting (switch per rack)
- Better airflow (switches at top of rack)

**Cons:**
- More switches to manage
- Higher upfront cost

### EoR (End-of-Row) - Legacy

```
Row 1:
  Rack 1-10 (Servers only)
  Rack 11: [EoR Switch] ──► [Core]

Long cable runs from each rack to EoR switch
```

**Pros:**
- Fewer switches
- Lower initial cost

**Cons:**
- Cable management nightmare
- Longer cable runs (signal degradation)
- Single point of failure per row

**Verdict:** ToR is industry standard for modern data centers.

---

## Key Takeaways

1. **Spine-leaf architecture** provides predictable latency, horizontal scalability, and non-blocking performance
2. **Layer 3 underlay + VXLAN overlay** separates physical from logical networks, enabling VM mobility and multi-tenancy
3. **EVPN** provides centralized control plane for VXLAN, eliminating flood-and-learn inefficiencies
4. **FCoE and iSCSI** converge storage and data networks onto Ethernet infrastructure
5. **Cut-through switching** achieves sub-microsecond latency for latency-sensitive applications
6. **PFC and ECN** enable lossless Ethernet required for storage protocols
7. **Automation** via Ansible, Python, and APIs is essential for managing large-scale data centers
8. **ToR architecture** is the modern standard for server connectivity

Data center switching demands performance, scalability, and automation far beyond traditional enterprise networking. Understanding these principles is critical for designing cloud-scale infrastructure.

---

**Chapter 14 Complete** | Next: Chapter 15 - Future of Switching and Emerging Technologies
