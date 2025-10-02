# Chapter 6: Spanning Tree Protocol (STP)

## 6.1 Introduction to Spanning Tree Protocol

Spanning Tree Protocol (STP) is one of the most critical protocols in switched networks. It prevents Layer 2 loops while maintaining path redundancy, ensuring network stability and availability.

### 6.1.1 Why STP Exists: The Loop Problem

**The Danger of Layer 2 Loops:**

Without STP, redundant switch connections create catastrophic broadcast storms:

```
         [Switch A]
          /      \
         /        \
    [Switch B]---[Switch C]
```

**What Happens Without STP:**

1. PC-A sends broadcast frame to Switch A
2. Switch A floods to Switch B and Switch C
3. Switch B forwards to Switch C
4. Switch C forwards to Switch B
5. Both switches forward back to Switch A
6. Loop continues infinitely

**Consequences:**
- **Broadcast Storm**: Frames multiply exponentially
- **MAC Table Instability**: Constantly changing source port for same MAC
- **Multiple Frame Copies**: End devices receive duplicate frames
- **Network Failure**: Switch CPU/bandwidth exhaustion (often < 30 seconds)

### 6.1.2 How STP Solves the Problem

STP creates a loop-free logical topology by:
1. Electing a root bridge (reference point)
2. Calculating best path to root for each switch
3. Blocking redundant paths
4. Maintaining blocked ports as backup (converge when failures occur)

**STP Topology Example:**
```
         [Root Bridge]
          /        \
    (Forwarding)  (Forwarding)
         /           \
    [Switch B]----(BLOCKED)----[Switch C]
```

## 6.2 STP Standards and Versions

### 6.2.1 STP Evolution

| Standard | Name | Description | Convergence Time |
|----------|------|-------------|------------------|
| 802.1D | Spanning Tree Protocol (STP) | Original standard | 30-50 seconds |
| 802.1w | Rapid Spanning Tree (RSTP) | Faster convergence | 1-3 seconds |
| 802.1s | Multiple Spanning Tree (MSTP) | Multiple instances for VLANs | 1-3 seconds |
| PVST+ | Per-VLAN Spanning Tree | Cisco proprietary, one instance per VLAN | 30-50 seconds |
| RPVST+ | Rapid PVST+ | Cisco rapid version of PVST+ | 1-3 seconds |

### 6.2.2 Default Modes by Platform

**Cisco Catalyst Switches:**
- **Default Mode**: PVST+ (or RPVST+ on newer models)
- **Backward Compatible**: Can interoperate with 802.1D STP
- **Per-VLAN**: Separate spanning tree for each VLAN

**Most Common in Production**: RPVST+ (Rapid Per-VLAN Spanning Tree Plus)

## 6.3 STP Operation and Components

### 6.3.1 Bridge ID

Every switch has a Bridge ID used to elect the root bridge:

```
Bridge ID = [Bridge Priority (2 bytes)] + [MAC Address (6 bytes)]
            [32768 default]              [Unique per switch]
```

**Extended System ID (PVST+):**
```
Bridge ID = [Priority (4 bits)] + [VLAN ID (12 bits)] + [MAC Address (6 bytes)]
            [32768 default]       [0-4095]              [Unique]
```

**Example:**
```
VLAN 10: 32768 + 10 = 32778.0024.9773.8400
VLAN 20: 32768 + 20 = 32788.0024.9773.8400
```

### 6.3.2 Root Bridge Election

**Process:**
1. All switches initially assume they are the root
2. Switches exchange Bridge Protocol Data Units (BPDUs)
3. Switch with **lowest Bridge ID** becomes root bridge
4. If priorities are equal, lowest MAC address wins

**Election Priority:**
```
Lowest Priority Value → Lowest MAC Address
```

**Example Election:**
```
Switch A: Priority 32768, MAC: 0024.9773.8400
Switch B: Priority 32768, MAC: 0024.9773.8200  ← ROOT (lowest MAC)
Switch C: Priority 32768, MAC: 0024.9773.8600
```

### 6.3.3 Port Roles

STP assigns roles to each switch port:

**Port Roles:**
1. **Root Port (RP)**:
   - One per non-root switch
   - Port with best path to root bridge
   - Always in forwarding state

2. **Designated Port (DP)**:
   - One per network segment
   - Forwards traffic toward root bridge
   - All root bridge ports are designated ports

3. **Alternate Port (AP)** (RSTP):
   - Backup path to root bridge
   - Immediately replaces root port if it fails

4. **Backup Port (BP)** (RSTP):
   - Backup for designated port on same switch
   - Rare (requires hub or multiple connections)

5. **Blocked/Discarding Port**:
   - Provides redundancy
   - Does not forward frames
   - Listens to BPDUs

### 6.3.4 Path Cost Calculation

STP uses **path cost** to determine the best path to root bridge.

**IEEE 802.1D-1998 Costs (Original):**
| Link Speed | Cost |
|------------|------|
| 10 Mbps | 100 |
| 100 Mbps | 19 |
| 1 Gbps | 4 |
| 10 Gbps | 2 |

**IEEE 802.1D-2004 Costs (Revised):**
| Link Speed | Cost |
|------------|------|
| 10 Mbps | 2,000,000 |
| 100 Mbps | 200,000 |
| 1 Gbps | 20,000 |
| 10 Gbps | 2,000 |
| 100 Gbps | 200 |

**Path Selection (Tiebreakers):**
1. Lowest root path cost
2. Lowest sender Bridge ID
3. Lowest sender Port ID

### 6.3.5 BPDU (Bridge Protocol Data Unit)

BPDUs are messages switches exchange to maintain the spanning tree topology.

**BPDU Types:**
- **Configuration BPDU**: Sent by root bridge every 2 seconds (Hello Time)
- **Topology Change Notification (TCN) BPDU**: Signals topology changes

**BPDU Contents:**
- Root Bridge ID
- Sender Bridge ID
- Root Path Cost
- Port ID
- Timer values (Hello, Max Age, Forward Delay)
- Flags (Topology Change, TC Acknowledgment)

## 6.4 STP Port States (802.1D)

### 6.4.1 Original STP Port States

| State | Duration | Forwards Data | Learns MACs | Receives BPDUs |
|-------|----------|---------------|-------------|----------------|
| **Disabled** | N/A | No | No | No |
| **Blocking** | 20 sec (Max Age) | No | No | Yes |
| **Listening** | 15 sec (Forward Delay) | No | No | Yes |
| **Learning** | 15 sec (Forward Delay) | No | Yes | Yes |
| **Forwarding** | Steady state | Yes | Yes | Yes |

**Total Convergence Time**: 30-50 seconds (20 + 15 + 15)

### 6.4.2 Port State Transitions

**When Port Comes Online:**
```
Disabled → Blocking → Listening → Learning → Forwarding
             (20s)       (15s)      (15s)
```

**When Topology Changes:**
```
Forwarding → Blocking → Listening → Learning → Forwarding
```

### 6.4.3 Why So Slow?

**Blocking (20 seconds)**:
- Waits for Max Age to expire
- Ensures no old/invalid BPDUs in network

**Listening (15 seconds)**:
- Participates in spanning tree
- Determines if port should forward

**Learning (15 seconds)**:
- Populates MAC address table
- Prepares for forwarding

## 6.5 Rapid Spanning Tree Protocol (RSTP)

### 6.5.1 RSTP Improvements

**Key Enhancements:**
1. **Faster Convergence**: 1-3 seconds vs 30-50 seconds
2. **Explicit Handshake**: Active confirmation of convergence
3. **Edge Port Concept**: Immediate forwarding for end devices
4. **Link Type Awareness**: Point-to-point vs shared media

### 6.5.2 RSTP Port States

RSTP simplifies port states:

| 802.1D State | 802.1w (RSTP) State | Forwards | Learns |
|--------------|---------------------|----------|--------|
| Disabled | Discarding | No | No |
| Blocking | Discarding | No | No |
| Listening | Discarding | No | No |
| Learning | Learning | No | Yes |
| Forwarding | Forwarding | Yes | Yes |

### 6.5.3 RSTP Port Roles

```
Root Port (RP)        ← Best path to root
Designated Port (DP)  ← Forwarding toward root
Alternate Port (AP)   ← Backup path to root (BLOCKED)
Backup Port (BP)      ← Backup designated port (BLOCKED)
```

### 6.5.4 RSTP Edge Ports

**Edge Port (PortFast):**
- Connected to end devices (not switches)
- Transitions immediately to forwarding
- Does not generate topology changes
- Automatically becomes non-edge if BPDU received

**Configuration:**
```
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# spanning-tree portfast
Switch(config-if)# exit

! Or globally for access ports
Switch(config)# spanning-tree portfast default
```

**Verification:**
```
Switch# show spanning-tree interface gigabitethernet 0/1 detail

Port 1 (GigabitEthernet0/1) of VLAN0010 is designated forwarding
  Port path cost 4, Port priority 128, Port Identifier 128.1
  Designated root has priority 32778, address 0024.9773.8200
  Designated bridge has priority 32778, address 0024.9773.8400
  Timers: message age 0, forward delay 0, hold 0
  BPDU: sent 3420, received 0
  The port is in the portfast mode        ← Edge Port
```

### 6.5.5 RSTP Link Types

**Point-to-Point (P2P):**
- Full-duplex connection
- Fast convergence with proposal/agreement mechanism
- Typical switch-to-switch trunk links

**Shared:**
- Half-duplex connection
- Falls back to 802.1D behavior
- Legacy hubs or shared media

**Configuration:**
```
Switch(config-if)# spanning-tree link-type point-to-point
Switch(config-if)# spanning-tree link-type shared
```

### 6.5.6 RSTP Proposal/Agreement

**Fast Convergence Mechanism:**

1. **New switch connects** to existing topology
2. **Sends Proposal BPDU** on root port
3. **Upstream switch receives proposal**:
   - Blocks all non-edge designated ports
   - Sends Agreement BPDU back
4. **New switch receives agreement**:
   - Immediately transitions to forwarding
5. **New switch propagates proposals** downstream

**Result**: Sub-second convergence in stable networks

## 6.6 STP Configuration

### 6.6.1 Basic RSTP Configuration

**Enable RSTP (RPVST+) Globally:**
```
Switch(config)# spanning-tree mode rapid-pvst
```

**Verify Current Mode:**
```
Switch# show spanning-tree summary
Switch is in rapid-pvst mode
Root bridge for: none
Extended system ID           is enabled
Portfast Default             is disabled
PortFast BPDU Guard Default  is disabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is enabled
UplinkFast                   is disabled
BackboneFast                 is disabled
```

### 6.6.2 Root Bridge Configuration

**Method 1: Manual Priority (Recommended)**
```
! Set primary root (priority 24576)
Switch(config)# spanning-tree vlan 10 priority 24576

! Set secondary root (priority 28672)
Switch(config)# spanning-tree vlan 10 priority 28672

! Priority must be multiple of 4096: 0, 4096, 8192, ..., 61440
```

**Method 2: Root Macro (Quick but less control)**
```
! Primary root (sets priority lower than current root)
Switch(config)# spanning-tree vlan 10 root primary

! Secondary root
Switch(config)# spanning-tree vlan 10 root secondary
```

**Best Practice:**
- Core/distribution switches = Root bridges
- Use deterministic priority values
- Document which switch is root for each VLAN

### 6.6.3 Port Cost Modification

**Change Port Cost:**
```
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# spanning-tree vlan 10 cost 100
Switch(config-if)# exit
```

**Change Port Priority:**
```
Switch(config-if)# spanning-tree vlan 10 port-priority 64
! Default is 128, lower is better
! Values: 0, 16, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240
```

### 6.6.4 STP Timers

**Default Timers:**
- **Hello Time**: 2 seconds (how often BPDUs sent)
- **Forward Delay**: 15 seconds (listening + learning)
- **Max Age**: 20 seconds (BPDU timeout)

**Modify Timers (Root Bridge Only):**
```
Switch(config)# spanning-tree vlan 10 hello-time 4
Switch(config)# spanning-tree vlan 10 forward-time 20
Switch(config)# spanning-tree vlan 10 max-age 30
```

**Warning**: Changing timers can cause instability. Use defaults unless specific need.

## 6.7 STP Protection Features

### 6.7.1 PortFast

**Purpose**: Immediately transition access ports to forwarding

**Configuration:**
```
! Per interface
Switch(config)# interface range gigabitethernet 0/1-20
Switch(config-if-range)# spanning-tree portfast
Switch(config-if-range)# exit

! Global for all access ports
Switch(config)# spanning-tree portfast default
```

**Warning**: Never enable on trunk or switch-to-switch links!

### 6.7.2 BPDU Guard

**Purpose**: Disable ports receiving BPDUs (prevent rogue switches)

**Configuration:**
```
! Per interface
Switch(config-if)# spanning-tree bpduguard enable

! Global (applies to PortFast ports)
Switch(config)# spanning-tree portfast bpduguard default
```

**Behavior:**
- Port receives BPDU → Port enters err-disabled state
- Prevents user from plugging in switch
- Requires manual recovery or auto-recovery

**Recovery:**
```
! Manual recovery
Switch(config)# interface gigabitethernet 0/5
Switch(config-if)# shutdown
Switch(config-if)# no shutdown
Switch(config-if)# exit

! Auto-recovery
Switch(config)# errdisable recovery cause bpduguard
Switch(config)# errdisable recovery interval 300
```

### 6.7.3 BPDU Filter

**Purpose**: Suppress sending/receiving BPDUs on edge ports

**Configuration:**
```
! Per interface (dangerous)
Switch(config-if)# spanning-tree bpdufilter enable

! Global (safer - applies to PortFast ports)
Switch(config)# spanning-tree portfast bpdufilter default
```

**Global Behavior:**
- If port receives BPDU → PortFast and BPDU filter disabled
- Port returns to normal STP operation

**Warning**: Interface-level BPDU filter completely disables STP on port!

### 6.7.4 Root Guard

**Purpose**: Prevent inferior BPDUs from becoming root bridge

**Configuration:**
```
Switch(config)# interface gigabitethernet 0/23
Switch(config-if)# spanning-tree guard root
Switch(config-if)# exit
```

**Behavior:**
- Port receives superior BPDU → Port enters root-inconsistent state
- Blocks traffic until superior BPDUs stop
- Automatically recovers when BPDUs stop

**Use Case**: Configure on all designated ports facing access layer

### 6.7.5 Loop Guard

**Purpose**: Prevent alternate ports from becoming designated due to unidirectional link failure

**Configuration:**
```
! Per interface
Switch(config-if)# spanning-tree guard loop

! Global
Switch(config)# spanning-tree loopguard default
```

**Behavior:**
- Port stops receiving BPDUs → Port enters loop-inconsistent state
- Prevents port from transitioning to forwarding
- Automatically recovers when BPDUs resume

**Use Case**: Configure on point-to-point links in redundant topology

### 6.7.6 UplinkFast

**Purpose**: Fast convergence when direct uplink fails (access layer only)

**Configuration:**
```
Switch(config)# spanning-tree uplinkfast
```

**Behavior:**
- Immediately transitions alternate port to forwarding
- Convergence: ~1-3 seconds
- Increases bridge priority to 49152 (prevents becoming root)

**Use Case**: Access layer switches with redundant uplinks

### 6.7.7 BackboneFast

**Purpose**: Fast convergence when indirect link fails

**Configuration:**
```
Switch(config)# spanning-tree backbonefast
```

**Behavior:**
- Reduces Max Age timer from 20 to 0 seconds
- Must be enabled on ALL switches
- Convergence: ~30 seconds → ~10 seconds

**Note**: Not needed with RSTP/RPVST+ (obsolete)

## 6.8 Multiple Spanning Tree Protocol (MSTP)

### 6.8.1 MSTP Overview

**Problem with PVST+:**
- Separate spanning tree instance per VLAN
- 1000 VLANs = 1000 STP instances = high CPU/memory usage

**MSTP Solution:**
- Maps multiple VLANs to a single spanning tree instance
- Reduces overhead while maintaining per-VLAN load balancing

### 6.8.2 MSTP Concepts

**Regions:**
- MST Region = group of switches with identical configuration
- All switches in region must have same:
  - MST configuration name
  - MST configuration revision number
  - VLAN-to-instance mapping

**Instances:**
- **IST (Instance 0)**: Internal Spanning Tree for region
- **MSTIs**: Multiple Spanning Tree Instances (1-15)

### 6.8.3 MSTP Configuration

**Basic MSTP Setup:**
```
! Enable MST mode
Switch(config)# spanning-tree mode mst

! Enter MST configuration
Switch(config)# spanning-tree mst configuration
Switch(config-mst)# name COMPANY_REGION
Switch(config-mst)# revision 1
Switch(config-mst)# instance 1 vlan 10,20,30
Switch(config-mst)# instance 2 vlan 40,50,60
Switch(config-mst)# exit

! Set root priorities per instance
Switch(config)# spanning-tree mst 1 priority 24576
Switch(config)# spanning-tree mst 2 priority 28672
```

**Verification:**
```
Switch# show spanning-tree mst configuration
Name      [COMPANY_REGION]
Revision  1     Instances configured 3

Instance  Vlans mapped
--------  ---------------------------------------------------------------------
0         1-9,11-19,21-29,31-39,41-49,51-4094
1         10,20,30
2         40,50,60
```

## 6.9 STP Verification and Monitoring

### 6.9.1 Essential Show Commands

**Overall STP Status:**
```
Switch# show spanning-tree

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    24586
             Address     0024.9773.8200
             Cost        4
             Port        24 (GigabitEthernet0/24)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     0024.9773.8400
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Desg FWD 4         128.1    P2p
Gi0/24              Root FWD 4         128.24   P2p
```

**Specific VLAN:**
```
Switch# show spanning-tree vlan 10
```

**Per-Interface Detail:**
```
Switch# show spanning-tree interface gigabitethernet 0/24

Vlan                Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
VLAN0010            Root FWD 4         128.24   P2p
```

**Root Bridge Information:**
```
Switch# show spanning-tree root

                                        Root    Hello Max Fwd
Vlan                   Root ID          Cost    Time  Age Dly  Root Port
---------------- -------------------- --------- ----- --- ---  ------------
VLAN0010         24586 0024.9773.8200         4     2  20  15  Gi0/24
VLAN0020         24596 0024.9773.8200         4     2  20  15  Gi0/24
```

**Summary:**
```
Switch# show spanning-tree summary
Switch is in rapid-pvst mode
Root bridge for: none
Extended system ID           is enabled
Portfast Default             is disabled
PortFast BPDU Guard Default  is enabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is enabled
UplinkFast                   is disabled
BackboneFast                 is disabled

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0010               0        0         0        2          2
VLAN0020               0        0         0        2          2
---------------------- -------- --------- -------- ---------- ----------
2 vlans                0        0         0        4          4
```

**Inconsistent Ports:**
```
Switch# show spanning-tree inconsistentports

Name                 Interface              Inconsistency
-------------------- ---------------------- ------------------
VLAN0010             GigabitEthernet0/5     Port BPDU Guard Inconsistent

Number of inconsistent ports (segments) in the system : 1
```

### 6.9.2 Debug Commands

```
! STP events
Switch# debug spanning-tree events

! BPDU transmission/reception
Switch# debug spanning-tree bpdu

! Root bridge election
Switch# debug spanning-tree root

! Turn off debugging
Switch# undebug all
```

## 6.10 Common STP Issues and Troubleshooting

### 6.10.1 Issue: Slow Convergence

**Symptoms:**
- Network unavailable for 30-50 seconds after link failure
- Users experience timeouts during topology changes

**Diagnosis:**
```
Switch# show spanning-tree summary
Switch is in pvst mode     ← Not using RSTP!
```

**Solution:**
```
Switch(config)# spanning-tree mode rapid-pvst
```

### 6.10.2 Issue: Incorrect Root Bridge

**Symptoms:**
- Suboptimal traffic paths
- Access switch elected as root
- Inconsistent root bridge across VLANs

**Diagnosis:**
```
Switch# show spanning-tree root

VLAN0010         32778 0024.9773.FFFF         0     2  20  15  Gi0/1
                 ↑ Default priority (should be lower for root)
```

**Solution:**
```
CoreSwitch(config)# spanning-tree vlan 10 priority 24576
CoreSwitch(config)# spanning-tree vlan 20 priority 24576
```

### 6.10.3 Issue: Broadcast Storm

**Symptoms:**
- Network completely down
- Switch CPU at 100%
- All link lights flashing rapidly
- Duplicate frames received

**Diagnosis:**
```
Switch# show processes cpu sorted
CPU utilization for five seconds: 99%/0%; one minute: 99%; five minutes: 99%

Switch# show interfaces gigabitethernet 0/1 | include broadcast
  30 second input rate 980000000 bits/sec, 950000 packets/sec
  30 second output rate 980000000 bits/sec, 950000 packets/sec
```

**Solution:**
1. **Immediately disconnect suspect links**
2. Verify STP is enabled:
   ```
   Switch# show spanning-tree summary
   ```
3. Check for err-disabled ports:
   ```
   Switch# show interfaces status err-disabled
   ```
4. Enable BPDU Guard on access ports:
   ```
   Switch(config)# spanning-tree portfast bpduguard default
   ```

### 6.10.4 Issue: Root Port Flapping

**Symptoms:**
- Frequent topology changes
- Root port alternating between interfaces
- Unstable network performance

**Diagnosis:**
```
Switch# show spanning-tree vlan 10

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/23              Root FWD 4         128.23   P2p  ← Root port
Gi0/24              Altn BLK 4         128.24   P2p  ← Equal cost!
```

**Solution - Modify Cost or Priority:**
```
Switch(config)# interface gigabitethernet 0/23
Switch(config-if)# spanning-tree vlan 10 cost 3
Switch(config-if)# exit
```

### 6.10.5 Issue: Unidirectional Link Failure

**Symptoms:**
- Port appears up but doesn't receive BPDUs
- Loop forms when port incorrectly transitions to forwarding

**Diagnosis:**
```
Switch# show spanning-tree interface gigabitethernet 0/5 detail
...
BPDU: sent 5420, received 0   ← Not receiving BPDUs!
```

**Solution:**
```
Switch(config)# spanning-tree loopguard default

! Or per-interface
Switch(config-if)# spanning-tree guard loop
```

### 6.10.6 Issue: BPDU Guard Err-Disabled Ports

**Symptoms:**
- Ports randomly go err-disabled
- Users complaining about intermittent connectivity

**Diagnosis:**
```
Switch# show interfaces status err-disabled

Port      Name               Status       Reason               Err-disabled Vlans
Gi0/5                        err-disabled bpduguard

Switch# show errdisable recovery
ErrDisable Reason            Timer Status
-----------------            --------------
bpduguard                    Disabled
```

**Solution:**
```
! Enable auto-recovery
Switch(config)# errdisable recovery cause bpduguard
Switch(config)# errdisable recovery interval 300

! Manually recover
Switch(config)# interface gigabitethernet 0/5
Switch(config-if)# shutdown
Switch(config-if)# no shutdown
```

**Prevention:**
- Don't connect switches to PortFast ports
- Document switch port locations
- User training

## 6.11 STP Best Practices

### 6.11.1 Design Recommendations

1. **Enable RSTP (Rapid-PVST+)**
   ```
   Switch(config)# spanning-tree mode rapid-pvst
   ```

2. **Manually Configure Root Bridges**
   - Primary root: Priority 24576
   - Secondary root: Priority 28672
   - Configure on core/distribution switches

3. **Enable PortFast on Access Ports**
   ```
   Switch(config)# spanning-tree portfast default
   ```

4. **Enable BPDU Guard Globally**
   ```
   Switch(config)# spanning-tree portfast bpduguard default
   ```

5. **Use Root Guard on Designated Ports**
   ```
   Switch(config-if)# spanning-tree guard root
   ```

6. **Enable Loop Guard Globally**
   ```
   Switch(config)# spanning-tree loopguard default
   ```

7. **Don't Modify Timers Unless Necessary**
   - Default timers are well-tested
   - Changes can cause instability

8. **Monitor for Topology Changes**
   ```
   Switch# show spanning-tree vlan 10 | include changes
   Topology change flag not set, detected flag not set
   Number of topology changes 12 last change occurred 2d17h ago
   ```

### 6.11.2 Security Hardening

**Complete Access Port Configuration:**
```
Switch(config)# interface range gigabitethernet 0/1-20
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# spanning-tree portfast
Switch(config-if-range)# spanning-tree bpduguard enable
Switch(config-if-range)# exit
```

**Complete Trunk Port Configuration:**
```
Switch(config)# interface range gigabitethernet 0/23-24
Switch(config-if-range)# switchport mode trunk
Switch(config-if-range)# spanning-tree guard root
Switch(config-if-range)# spanning-tree link-type point-to-point
Switch(config-if-range)# exit
```

### 6.11.3 Documentation Template

```
SPANNING TREE DESIGN - VLAN 10

Root Bridge: CoreSwitch1 (Priority: 24576, MAC: 0024.9773.8200)
Secondary Root: CoreSwitch2 (Priority: 28672, MAC: 0024.9773.8300)

Topology:
         [CoreSwitch1 - ROOT]
          /                  \
      (Gi0/24)              (Gi0/24)
         /                      \
    [AccessSW1]-------------(BLOCKED)------------[AccessSW2]
                           (Gi0/23)

Port Roles:
  CoreSwitch1 Gi0/24 → Designated (to AccessSW1)
  CoreSwitch1 Gi0/23 → Designated (to AccessSW2)
  AccessSW1 Gi0/24 → Root Port (to CoreSwitch1)
  AccessSW1 Gi0/23 → Designated (to AccessSW2)
  AccessSW2 Gi0/24 → Root Port (to CoreSwitch1)
  AccessSW2 Gi0/23 → Alternate (BLOCKED)

Protection:
  - BPDU Guard: Enabled on all access ports (Gi0/1-20)
  - Root Guard: Enabled on all designated ports
  - Loop Guard: Enabled globally
  - PortFast: Enabled on all access ports
```

## 6.12 Real-World Scenario

### Enterprise Campus STP Design

**Network Topology:**
```
        [Core1]----------[Core2]
         /    \          /     \
        /      \        /       \
    [Dist1]---[Dist2]---[Dist3]---[Dist4]
      |         |         |         |
   [Acc1]    [Acc2]    [Acc3]    [Acc4]
```

**Requirements:**
- High availability with redundant paths
- Fast convergence (< 3 seconds)
- Load balancing across VLANs
- Protection against rogue switches

**Design:**

**Core Layer (Root Bridges):**
```
! Core1 - Primary root for VLANs 10, 30, 50
Core1(config)# spanning-tree mode rapid-pvst
Core1(config)# spanning-tree vlan 10,30,50 priority 24576
Core1(config)# spanning-tree vlan 20,40,60 priority 28672

! Core2 - Primary root for VLANs 20, 40, 60
Core2(config)# spanning-tree mode rapid-pvst
Core2(config)# spanning-tree vlan 20,40,60 priority 24576
Core2(config)# spanning-tree vlan 10,30,50 priority 28672
```

**Distribution Layer:**
```
Dist1(config)# spanning-tree mode rapid-pvst
Dist1(config)# interface range gigabitethernet 0/23-24
Dist1(config-if-range)# spanning-tree guard root
Dist1(config-if-range)# spanning-tree link-type point-to-point
Dist1(config-if-range)# exit
```

**Access Layer:**
```
Acc1(config)# spanning-tree mode rapid-pvst
Acc1(config)# spanning-tree portfast default
Acc1(config)# spanning-tree portfast bpduguard default
Acc1(config)# spanning-tree loopguard default

! Uplinks to distribution
Acc1(config)# interface range gigabitethernet 0/23-24
Acc1(config-if-range)# spanning-tree link-type point-to-point
Acc1(config-if-range)# exit

! Access ports
Acc1(config)# interface range gigabitethernet 0/1-20
Acc1(config-if-range)# switchport mode access
Acc1(config-if-range)# spanning-tree portfast
Acc1(config-if-range)# spanning-tree bpduguard enable
Acc1(config-if-range)# exit
```

**Result:**
- Sub-second convergence with RSTP
- Load balancing: Core1 root for VLANs 10,30,50; Core2 root for VLANs 20,40,60
- Protection against loops and rogue devices
- Automatic recovery from link failures

## Summary

Spanning Tree Protocol is essential for loop-free, redundant switched networks:

- **STP prevents Layer 2 loops** by blocking redundant paths
- **RSTP offers fast convergence** (1-3 seconds vs 30-50 seconds)
- **Port roles**: Root, Designated, Alternate, Backup
- **Port states**: Discarding, Learning, Forwarding
- **Protection features**: PortFast, BPDU Guard, Root Guard, Loop Guard
- **Best practice**: Use Rapid-PVST+, manually configure root bridges, enable protection features

In the next chapter, we'll explore Link Aggregation (EtherChannel), which combines multiple physical links into a single logical link for increased bandwidth and redundancy.
