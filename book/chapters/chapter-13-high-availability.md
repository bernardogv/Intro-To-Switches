# Chapter 13: High Availability and Redundancy

## Introduction

Network downtime is expensive. For e-commerce sites, every minute offline costs thousands in lost revenue. For hospitals, network failures can impact patient care. For financial institutions, connectivity loss means regulatory violations and customer distrust. High availability (HA) design ensures networks continue operating despite hardware failures, software bugs, or human errors.

This chapter explores redundancy strategies, failover mechanisms, and best practices for building networks that achieve "five nines" availability (99.999% uptime = 5.26 minutes of downtime per year).

---

## 13.1 Designing for High Availability

### The Cost of Downtime

**Industry Statistics:**
- Average cost of network downtime: $5,600 per minute
- E-commerce: $300,000 per hour
- Healthcare: Patient safety risks, regulatory fines
- Manufacturing: Production line stoppages

**Availability Tiers:**

| Availability | Downtime/Year | Downtime/Month | Use Case |
|--------------|---------------|----------------|----------|
| 99% | 3.65 days | 7.2 hours | Non-critical systems |
| 99.9% (3 nines) | 8.76 hours | 43.2 minutes | Standard business |
| 99.99% (4 nines) | 52.6 minutes | 4.32 minutes | Financial services |
| 99.999% (5 nines) | 5.26 minutes | 26 seconds | Telecom, critical infrastructure |

### HA Design Principles

**1. Eliminate Single Points of Failure (SPOFs)**

```
❌ Single Point of Failure:
                Internet
                    │
                    ▼
               [Router] ← SPOF!
                    │
               [Core Switch] ← SPOF!
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
    [Access]    [Access]    [Access]

✅ Redundant Design:
                Internet
                 │   │
        ┌────────┘   └────────┐
        ▼                     ▼
    [Router 1]           [Router 2]
        │                     │
        └────────┬───────┬────┘
                 ▼       ▼
            [Core 1]  [Core 2]
                 │       │
        ┌────────┼───────┼────────┐
        ▼        ▼       ▼        ▼
    [Access Dual-Homed to Both Cores]
```

**2. Redundant Paths**
- Multiple uplinks from access to distribution
- Dual connections to core
- Diverse physical paths (different conduits)

**3. Fast Failover**
- Sub-second convergence times
- Automatic detection and switchover
- No manual intervention required

**4. Geographic Diversity**
- Primary and backup data centers
- Disaster recovery sites
- Protection against localized failures (fire, flood, power outage)

---

## 13.2 Link Redundancy: EtherChannel Review

### Why EtherChannel is Critical for HA

**Without EtherChannel:**
```
Access Switch
    │
    ├──► Link 1 to Core (Active)
    └──► Link 2 to Core (Blocked by STP)

Problem: 50% of bandwidth wasted
```

**With EtherChannel:**
```
Access Switch
    │
    ├──► Link 1 to Core (Active)
    └──► Link 2 to Core (Active)

Benefit: Both links active, automatic failover
```

**Load Balancing + Redundancy:**

```cisco
! Access switch configuration
Switch(config)# interface range gigabitethernet0/23-24
Switch(config-if-range)# channel-group 1 mode active
Switch(config-if-range)# exit

Switch(config)# interface port-channel 1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan all

! Verification
Switch# show etherchannel summary
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------
1      Po1(SU)       LACP        Gi0/23(P)   Gi0/24(P)

! Test failover
Switch(config)# interface gigabitethernet0/23
Switch(config-if)# shutdown

! Verify traffic continues on Gi0/24
Switch# show etherchannel 1 port-channel
                Port-channels in the group:
                ---------------------------
Port-channel: Po1
------------
Age of the Port-channel   = 0d:01h:15m:30s
Logical slot/port   = 16/1       Number of ports = 1  ← One port remaining
```

**Failover Time:** < 1 second (LACP detects failure via missed hello packets)

---

## 13.3 First-Hop Redundancy Protocols (FHRP)

### The Problem: Single Default Gateway

```
Clients configured with default gateway: 10.1.1.1

10.1.1.1 ──► [Router 1] ◄── Primary
                 │
                 X  ← Router 1 fails

Result: All clients lose connectivity (even though Router 2 is available)
```

### HSRP (Hot Standby Router Protocol)

**Cisco proprietary, most common**

```
Virtual IP: 10.1.1.1 (clients use this as default gateway)
Router 1: 10.1.1.2 (Active)
Router 2: 10.1.1.3 (Standby)

Active router responds to ARP requests for 10.1.1.1
If Active fails, Standby becomes Active in ~3 seconds
```

**Configuration:**

```cisco
! Router 1 (Primary)
interface vlan 10
 ip address 10.1.1.2 255.255.255.0
 standby version 2
 standby 1 ip 10.1.1.1
 standby 1 priority 110
 standby 1 preempt
 standby 1 track gigabitethernet0/24 10

! Router 2 (Secondary)
interface vlan 10
 ip address 10.1.1.3 255.255.255.0
 standby version 2
 standby 1 ip 10.1.1.1
 standby 1 priority 100
```

**Key Parameters:**

- **Priority:** Higher priority becomes Active (default 100)
- **Preempt:** Higher priority router takes over when it comes online
- **Track:** Monitor uplink interface; reduce priority if uplink fails

**Verification:**

```cisco
Switch# show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Vl10        1    110 P Active   local           10.1.1.3        10.1.1.1
```

### VRRP (Virtual Router Redundancy Protocol)

**Open standard (RFC 5798), similar to HSRP**

```cisco
! Router 1
interface vlan 10
 ip address 10.1.1.2 255.255.255.0
 vrrp 1 ip 10.1.1.1
 vrrp 1 priority 110
 vrrp 1 preempt

! Router 2
interface vlan 10
 ip address 10.1.1.3 255.255.255.0
 vrrp 1 ip 10.1.1.1
 vrrp 1 priority 100
```

**HSRP vs VRRP:**

| Feature | HSRP | VRRP |
|---------|------|------|
| Standard | Cisco proprietary | RFC 5798 (open) |
| Virtual MAC | 0000.0C07.ACxx | 0000.5E00.01xx |
| Multicast Address | 224.0.0.2 | 224.0.0.18 |
| Hello Timer | 3 sec (default) | 1 sec (default) |
| Preempt | Optional | Enabled by default |
| Authentication | MD5 supported | MD5 supported |

**When to Use:**
- **HSRP:** Cisco-only environment, need advanced features (object tracking)
- **VRRP:** Multi-vendor environment, standards compliance required

### GLBP (Gateway Load Balancing Protocol)

**Cisco proprietary, provides load balancing + redundancy**

**Problem with HSRP/VRRP:** Standby router is idle (wasted bandwidth)

**GLBP Solution:** Multiple routers answer ARP requests with different virtual MACs

```
Client 1 ARPs for 10.1.1.1 → Gets MAC1 (points to Router 1)
Client 2 ARPs for 10.1.1.1 → Gets MAC2 (points to Router 2)
Client 3 ARPs for 10.1.1.1 → Gets MAC3 (points to Router 3)

All routers actively forward traffic (load balanced)
```

**Configuration:**

```cisco
! Router 1
interface vlan 10
 ip address 10.1.1.2 255.255.255.0
 glbp 1 ip 10.1.1.1
 glbp 1 priority 110
 glbp 1 preempt
 glbp 1 load-balancing round-robin

! Router 2
interface vlan 10
 ip address 10.1.1.3 255.255.255.0
 glbp 1 ip 10.1.1.1
 glbp 1 priority 100
```

**Verification:**

```cisco
Switch# show glbp brief
Interface   Grp  Fwd Pri State    Address         Active router   Standby router
Vl10        1    -   110 Active   10.1.1.1        local           10.1.1.3
Vl10        1    1   -   Active   0007.b400.0101  local           -
Vl10        1    2   -   Listen   0007.b400.0102  10.1.1.3        -
```

---

## 13.4 Switch Redundancy: Dual-Homing and Stack Failover

### Dual-Homed Access Switches

**Configuration:**

```
Access Switch
    ├──► Port-Channel to Core 1 (Gi0/23-24)
    └──► Port-Channel to Core 2 (Gi0/21-22)

Both uplinks active (load balanced by STP/EtherChannel)
If Core 1 fails, all traffic instantly reroutes to Core 2
```

**Best Practices:**

```cisco
! Access switch
interface range gigabitethernet0/23-24
 channel-group 1 mode active
 description Uplink to Core-1

interface range gigabitethernet0/21-22
 channel-group 2 mode active
 description Uplink to Core-2

! Ensure both port-channels are in different STP paths
spanning-tree vlan 1-100 priority 61440
```

### StackWise Failover

**Stack Master Failure:**

```
1. Master switch fails
2. Stack members detect loss via stack protocol
3. Election process: Highest priority becomes new master
4. Configuration preserved (synchronized across stack)
5. Failover time: ~30 seconds (full reconvergence)
```

**Best Practices:**

```cisco
! Set priorities to control master election
Switch(config)# switch 1 priority 15  ← Preferred master
Switch(config)# switch 2 priority 10  ← Backup master
Switch(config)# switch 3 priority 5   ← Last resort

! Monitor stack status
Switch# show switch
Switch/Stack Mac Address : 0050.56be.0001
                                           H/W   Current
Switch#  Role   Mac Address     Priority Version  State
--------------------------------------------------------------
*1       Master 0050.56be.0001     15     V01     Ready
 2       Member 0050.56be.0002     10     V01     Ready
 3       Member 0050.56be.0003      5     V01     Ready
```

---

## 13.5 Spanning Tree and High Availability

### Rapid PVST+ for Fast Convergence

**Standard STP Convergence:** 30-50 seconds (unacceptable for HA)
**Rapid PVST+ Convergence:** 1-2 seconds

**Configuration:**

```cisco
! Enable Rapid PVST+ (default on modern switches)
Switch(config)# spanning-tree mode rapid-pvst

! Set root bridge priorities
! Core-1 (Primary Root)
Switch(config)# spanning-tree vlan 1-100 priority 4096

! Core-2 (Secondary Root)
Switch(config)# spanning-tree vlan 1-100 priority 8192

! Access switches (high priority = never root)
Switch(config)# spanning-tree vlan 1-100 priority 61440
```

### PortFast and BPDU Guard

**Speed up port activation for end devices:**

```cisco
! Access ports with hosts
interface range gigabitethernet0/1-20
 spanning-tree portfast
 spanning-tree bpduguard enable

! Automatically enable on all access ports
Switch(config)# spanning-tree portfast default
Switch(config)# spanning-tree portfast bpduguard default
```

**PortFast:** Port bypasses STP states (goes straight to forwarding)
**BPDU Guard:** Disables port if BPDU received (prevents loops)

### MST (Multiple Spanning Tree)

**Problem:** PVST+ runs one STP instance per VLAN (resource intensive for 100+ VLANs)

**MST Solution:** Group VLANs into instances

```cisco
! Configure MST
Switch(config)# spanning-tree mode mst
Switch(config)# spanning-tree mst configuration
Switch(config-mst)# name REGION1
Switch(config-mst)# revision 1
Switch(config-mst)# instance 1 vlan 1-50
Switch(config-mst)# instance 2 vlan 51-100
Switch(config-mst)# exit

! Set priorities per instance
Switch(config)# spanning-tree mst 1 priority 4096
Switch(config)# spanning-tree mst 2 priority 4096
```

**Benefits:**
- Reduces CPU/memory usage
- Faster convergence
- Load balancing across instances

---

## 13.6 Hardware Redundancy

### Redundant Power Supplies

**Dual Power Supplies with Separate Circuits:**

```
Switch
├── PSU 1 ──► Circuit A (Utility Power)
└── PSU 2 ──► Circuit B (Generator/UPS)

If Circuit A fails, PSU 2 continues powering switch
No downtime
```

**Monitoring:**

```cisco
Switch# show environment power
Power                              Fan States
Supply  Model                Watts Output  Status   Sensor
------  -------------------  -----  ------  -------  ------
PS1     C3KX-PWR-1100WAC      1100   Good   On       Good
PS2     C3KX-PWR-1100WAC      1100   Good   On       Good

! Configure SNMP alerts
Switch(config)# snmp-server enable traps envmon
```

### Redundant Fans and Cooling

**Environmental Monitoring:**

```cisco
Switch# show environment temperature
Temperature                        Status
-------------------------------------------
System Temperature                 OK
Intake Temperature                 32 Celsius (89 Fahrenheit)
Hotspot Temperature                45 Celsius (113 Fahrenheit)
ASIC Temperature                   52 Celsius (125 Fahrenheit)

Switch# show environment fan
Fan                    Status
-------------------------------------------
Fan 1                  OK
Fan 2                  OK
Fan 3                  OK
```

### Modular Switches: Hot-Swappable Components

**Supervisor Module Redundancy:**

```
Chassis
├── Supervisor A (Active)
├── Supervisor B (Standby)
├── Line Card 1
├── Line Card 2
└── Line Card 3

If Supervisor A fails:
- Supervisor B takes over
- No configuration loss (synced)
- Downtime: < 30 seconds
```

**Configuration Synchronization:**

```cisco
! Enable config sync between supervisors
Switch(config)# redundancy
Switch(config-red)# mode sso
Switch(config-red)# main-cpu
Switch(config-red-main-cpu)# auto-sync standard
```

---

## 13.7 Disaster Recovery and Backup Strategies

### Configuration Backups

**Automated Backup Script:**

```bash
#!/bin/bash
# Backup switch configs daily

SWITCHES="192.168.1.1 192.168.1.2 192.168.1.3"
DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup/switch-configs/$DATE"

mkdir -p $BACKUP_DIR

for SWITCH in $SWITCHES; do
    scp admin@$SWITCH:running-config $BACKUP_DIR/${SWITCH}_config.txt
    echo "Backed up $SWITCH"
done

# Retain 30 days
find /backup/switch-configs -type d -mtime +30 -exec rm -rf {} \;
```

**RANCID (Really Awesome New Cisco Config Differ):**
- Monitors config changes
- Maintains version history (Git integration)
- Email alerts on changes
- Multi-vendor support

### Geographic Redundancy

**Primary and Secondary Data Centers:**

```
Primary DC (Location A)
├── Core Switches (Active)
├── Distribution
└── Access

Secondary DC (Location B)
├── Core Switches (Standby)
├── Distribution
└── Access

Connected via:
- Dark fiber (dedicated fiber connection)
- MPLS WAN
- VPN backup
```

**DCI (Data Center Interconnect):**

```cisco
! Layer 2 extension between sites (VLAN stretching)
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30
Switch(config-if)# description DCI to Secondary DC
```

**⚠️ Caution:** Layer 2 DCI can cause STP issues over long distances. Use VPLS or OTV for large geographic separation.

---

## 13.8 Monitoring and Maintenance for HA

### Proactive Health Checks

**Daily Monitoring:**

```cisco
! Check hardware status
show environment all

! Verify redundancy status
show redundancy
show stack

! Check STP topology
show spanning-tree summary

! Monitor uplink utilization
show interfaces counters

! Review logs for warnings
show logging | include %
```

**SNMP Traps for Critical Events:**

```cisco
snmp-server enable traps snmp linkdown linkup
snmp-server enable traps envmon fan shutdown supply temperature
snmp-server enable traps config
snmp-server enable traps hsrp
snmp-server enable traps port-security
```

### Scheduled Maintenance Windows

**Change Management Best Practices:**

1. **Test in lab first** (replicate production topology)
2. **Schedule during low-traffic periods** (2 AM on weekends)
3. **Have rollback plan** ready
4. **Notify stakeholders** via change control
5. **Perform changes incrementally** (one switch at a time)

**Example Maintenance Procedure:**

```
1. Backup current configuration
2. Announce maintenance window
3. Upgrade standby supervisor first
4. Verify standby is operational
5. Perform switchover to upgraded supervisor
6. Upgrade former active (now standby)
7. Verify all features operational
8. Close maintenance window
```

---

## Key Takeaways

1. **Eliminate SPOFs** through redundant hardware, links, and paths
2. **EtherChannel** provides both bandwidth aggregation and failover
3. **FHRP** (HSRP/VRRP/GLBP) ensures gateway redundancy; GLBP adds load balancing
4. **Dual-homing** access switches to multiple cores prevents single switch failures
5. **Rapid PVST+** and PortFast accelerate convergence to 1-2 seconds
6. **Redundant power supplies** and hot-swappable components minimize hardware downtime
7. **Configuration backups** and geographic redundancy protect against disasters
8. **Proactive monitoring** detects issues before they cause outages

High availability is achieved through layered redundancy—no single failure should bring down the network. With proper design, modern switched networks can achieve 99.99% or higher uptime.

---

**Chapter 13 Complete** | Next: Chapter 14 - Data Center Switching
