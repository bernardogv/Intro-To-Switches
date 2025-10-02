# Chapter 5: VLANs (Virtual LANs)

## 5.1 Introduction to VLANs

Virtual LANs (VLANs) are one of the most fundamental features in modern switched networks. They allow network administrators to logically segment a physical network into multiple broadcast domains, regardless of the physical topology.

### 5.1.1 What is a VLAN?

A VLAN is a logical broadcast domain that can span multiple physical LAN segments. Without VLANs, switches forward broadcast traffic to all ports except the ingress port. With VLANs, switches only forward broadcasts to ports belonging to the same VLAN.

**Key Concepts:**
- **Broadcast Domain**: A VLAN creates a logical broadcast domain
- **VLAN ID**: Each VLAN is identified by a number (1-4094)
- **VLAN 1**: Default VLAN on all switches (cannot be deleted)
- **Reserved VLANs**: 0, 4095, and extended VLANs (1006-4094) on some platforms

### 5.1.2 Benefits of VLANs

**Security:**
- Isolates sensitive traffic (e.g., HR, Finance, Guest networks)
- Reduces attack surface by limiting broadcast domains
- Enables granular access control policies

**Performance:**
- Reduces broadcast traffic by limiting broadcast domain size
- Improves overall network efficiency
- Better bandwidth utilization

**Flexibility:**
- Users can be grouped logically regardless of physical location
- Easy to move users between VLANs without physical recabling
- Simplified network management

**Cost Savings:**
- Reduces need for multiple physical switches
- Eliminates router ports for simple segmentation
- Easier to scale and reorganize

## 5.2 VLAN Types and Ranges

### 5.2.1 VLAN Ranges

| VLAN Range | Type | Description |
|------------|------|-------------|
| 0, 4095 | Reserved | Cannot be used |
| 1 | Default | Factory default, cannot be deleted |
| 2-1001 | Normal Range | Standard VLANs, stored in vlan.dat |
| 1002-1005 | Reserved | Legacy (FDDI, Token Ring) |
| 1006-4094 | Extended Range | Requires VTP transparent mode |

### 5.2.2 VLAN Types

**Data VLANs:**
- Carry user-generated traffic
- Separated from voice and management traffic
- Most common type of VLAN

**Voice VLANs:**
- Dedicated to VoIP traffic
- Enables QoS prioritization
- Cisco IP Phones can tag traffic automatically

**Management VLANs:**
- Used for switch management (SSH, SNMP, etc.)
- Should never be VLAN 1 (security best practice)
- Restricted access for administrative tasks

**Native VLANs:**
- Used on trunk ports for untagged traffic
- Default is VLAN 1 (should be changed)
- Security consideration for VLAN hopping attacks

## 5.3 VLAN Configuration

### 5.3.1 Creating VLANs

**Method 1: VLAN Database Mode (Legacy)**
```
Switch> enable
Switch# vlan database
Switch(vlan)# vlan 10 name SALES
Switch(vlan)# vlan 20 name ENGINEERING
Switch(vlan)# exit
```

**Method 2: Global Configuration Mode (Recommended)**
```
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name SALES
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name ENGINEERING
Switch(config-vlan)# exit

Switch(config)# vlan 30
Switch(config-vlan)# name HR
Switch(config-vlan)# exit

Switch(config)# vlan 99
Switch(config-vlan)# name MANAGEMENT
Switch(config-vlan)# exit
```

**Verification:**
```
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
10   SALES                            active
20   ENGINEERING                      active
30   HR                               active
99   MANAGEMENT                       active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

### 5.3.2 Assigning Ports to VLANs

**Single Port Assignment:**
```
Switch(config)# interface fastethernet 0/5
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit
```

**Multiple Ports Using Range:**
```
Switch(config)# interface range fastethernet 0/6-10
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
Switch(config-if-range)# exit

Switch(config)# interface range fastethernet 0/11-15
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 30
Switch(config-if-range)# exit
```

**Voice VLAN Configuration:**
```
Switch(config)# interface fastethernet 0/7
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# switchport voice vlan 150
Switch(config-if)# mls qos trust cos
Switch(config-if)# exit
```

### 5.3.3 Deleting VLANs

**Single VLAN:**
```
Switch(config)# no vlan 30
```

**Important Notes:**
- Ports assigned to deleted VLAN become inactive
- Reassign ports before deleting VLANs
- Cannot delete VLAN 1
- Use `delete flash:vlan.dat` to erase entire VLAN database

## 5.4 Access Ports vs Trunk Ports

### 5.4.1 Access Ports

Access ports belong to a single VLAN and carry untagged traffic.

**Configuration:**
```
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# spanning-tree portfast
Switch(config-if)# spanning-tree bpduguard enable
Switch(config-if)# exit
```

**Best Practices:**
- Always explicitly configure `switchport mode access`
- Enable PortFast on end-user ports
- Enable BPDU Guard to prevent topology loops
- Document VLAN assignments

### 5.4.2 Trunk Ports

Trunk ports carry traffic for multiple VLANs using IEEE 802.1Q tagging.

**802.1Q Frame Format:**
```
[Dest MAC | Src MAC | 802.1Q Tag | Type | Data | FCS]
                      |
                      +-- TPID (0x8100) | PCP | DEI | VID
                          2 bytes           3b    1b   12b
```

- **TPID**: Tag Protocol Identifier (0x8100)
- **PCP**: Priority Code Point (for QoS)
- **DEI**: Drop Eligible Indicator
- **VID**: VLAN Identifier (12 bits = 4094 VLANs)

**Basic Trunk Configuration:**
```
Switch(config)# interface gigabitethernet 0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 99
Switch(config-if)# switchport trunk allowed vlan 10,20,30,99
Switch(config-if)# exit
```

**Trunk Allowed VLANs Management:**
```
! Add VLANs to existing list
Switch(config-if)# switchport trunk allowed vlan add 40,50

! Remove VLANs from list
Switch(config-if)# switchport trunk allowed vlan remove 30

! Allow all VLANs except specified
Switch(config-if)# switchport trunk allowed vlan except 1-9,31-98

! Allow all VLANs (not recommended)
Switch(config-if)# switchport trunk allowed vlan all
```

### 5.4.3 Dynamic Trunking Protocol (DTP)

DTP automatically negotiates trunk links between switches. **Not recommended for production** due to security concerns.

**DTP Modes:**
```
Switch(config-if)# switchport mode ?
  access        Set trunking mode to ACCESS unconditionally
  dynamic       Set trunking mode to dynamically negotiate
  trunk         Set trunking mode to TRUNK unconditionally

Switch(config-if)# switchport mode dynamic ?
  auto          Set trunking mode dynamic negotiation parameter to AUTO
  desirable     Set trunking mode dynamic negotiation parameter to DESIRABLE
```

**DTP Mode Combinations:**

| Local Mode | Remote Mode | Result |
|------------|-------------|--------|
| trunk | trunk | Trunk |
| trunk | dynamic desirable | Trunk |
| trunk | dynamic auto | Trunk |
| dynamic desirable | dynamic desirable | Trunk |
| dynamic desirable | dynamic auto | Trunk |
| dynamic auto | dynamic auto | Access |
| access | * | Access |

**Security Best Practice - Disable DTP:**
```
Switch(config)# interface gigabitethernet 0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport nonegotiate
Switch(config-if)# exit
```

## 5.5 Native VLANs

### 5.5.1 Understanding Native VLANs

The native VLAN is used for untagged traffic on a trunk link. By default, this is VLAN 1.

**How Native VLAN Works:**
1. Tagged frames from allowed VLANs traverse the trunk normally
2. Untagged frames are assumed to be in the native VLAN
3. Frames from the native VLAN are sent untagged
4. Both ends of a trunk must have the same native VLAN

### 5.5.2 Native VLAN Security Concerns

**VLAN Hopping Attack:**
An attacker can send double-tagged frames to access other VLANs:
```
[Outer 802.1Q Tag: VLAN 1] [Inner 802.1Q Tag: VLAN 30] [Malicious Payload]
```

The first switch strips the outer tag (native VLAN), forwarding the inner-tagged frame to VLAN 30.

### 5.5.3 Native VLAN Best Practices

**Change Native VLAN from Default:**
```
Switch(config)# vlan 999
Switch(config-vlan)# name NATIVE_UNUSED
Switch(config-vlan)# exit

Switch(config)# interface range gigabitethernet 0/23-24
Switch(config-if-range)# switchport trunk native vlan 999
Switch(config-if-range)# exit
```

**Verify Native VLAN Consistency:**
```
Switch# show interfaces trunk

Port        Mode         Encapsulation  Status        Native vlan
Gi0/23      on           802.1q         trunking      999
Gi0/24      on           802.1q         trunking      999

Port        Vlans allowed on trunk
Gi0/23      10,20,30,99
Gi0/24      10,20,30,99

Port        Vlans allowed and active in management domain
Gi0/23      10,20,30,99
Gi0/24      10,20,30,99

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/23      10,20,30,99
Gi0/24      10,20,30,99
```

**Native VLAN Mismatch Detection:**
```
Switch# show interfaces gigabitethernet 0/24 trunk

Port        Mode         Encapsulation  Status        Native vlan
Gi0/24      on           802.1q         trunking      999

! CDP will report native VLAN mismatches
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on GigabitEthernet0/24 (999), with Switch2 GigabitEthernet0/1 (1).
```

## 5.6 VLAN Trunking Protocol (VTP)

### 5.6.1 VTP Overview

VTP is a Cisco proprietary protocol that propagates VLAN configuration information across trunk links.

**VTP Modes:**
- **Server**: Can create, modify, delete VLANs; propagates changes
- **Client**: Cannot modify VLANs; accepts VTP updates
- **Transparent**: Can modify local VLANs; forwards VTP messages
- **Off**: Disables VTP entirely (VTPv3 only)

### 5.6.2 VTP Configuration

**VTP Server Configuration:**
```
Switch1(config)# vtp mode server
Switch1(config)# vtp domain COMPANY
Switch1(config)# vtp password SecurePass123
Switch1(config)# vtp version 2
```

**VTP Client Configuration:**
```
Switch2(config)# vtp mode client
Switch2(config)# vtp domain COMPANY
Switch2(config)# vtp password SecurePass123
Switch2(config)# vtp version 2
```

**VTP Transparent Configuration:**
```
Switch3(config)# vtp mode transparent
Switch3(config)# vtp domain COMPANY
```

**Verification:**
```
Switch# show vtp status

VTP Version capable             : 1 to 3
VTP version running             : 2
VTP Domain Name                 : COMPANY
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0024.9773.8400
Configuration last modified by 192.168.1.10 at 3-1-93 00:15:20

Feature VLAN:
--------------
VTP Operating Mode                : Server
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 8
Configuration Revision            : 12
MD5 digest                        : 0x3F 0x21 0xA4 0x5B 0x0C 0x9D
```

### 5.6.3 VTP Pitfalls and Best Practices

**Configuration Revision Number:**
VTP uses a revision number to determine which switch has the latest configuration. Higher revision = newer configuration.

**Dangerous Scenario:**
1. Old switch with high revision number (e.g., 50) is connected
2. Current switches have lower revision (e.g., 12)
3. Old switch overwrites current VLAN database
4. All VLANs are deleted or misconfigured

**Prevention:**
```
! Before connecting any switch, reset VTP revision:
Switch# vtp mode transparent
Switch# vtp mode server
! This resets revision to 0

! Or change VTP domain name temporarily:
Switch(config)# vtp domain TEMP
Switch(config)# vtp domain COMPANY
```

**Best Practices:**
1. **Use VTP Transparent or Off**: Avoid automatic VLAN propagation
2. **VTP Pruning**: Enable to reduce unnecessary broadcast traffic
3. **Strong Passwords**: Use VTP password authentication
4. **VTPv3**: Offers better security and features (extended VLANs)

### 5.6.4 VTP Pruning

VTP Pruning prevents unnecessary broadcast traffic across trunk links.

**Enable VTP Pruning:**
```
Switch(config)# vtp pruning
```

**Verify:**
```
Switch# show vtp status | include Pruning
VTP Pruning Mode                : Enabled
```

**Prune-Eligible VLANs:**
```
Switch(config)# interface gigabitethernet 0/24
Switch(config-if)# switchport trunk pruning vlan 10-20
Switch(config-if)# switchport trunk pruning vlan remove 15
```

## 5.7 Inter-VLAN Routing

VLANs create separate broadcast domains, requiring a Layer 3 device (router or Layer 3 switch) for inter-VLAN communication.

### 5.7.1 Router-on-a-Stick

Uses a single router interface with subinterfaces for each VLAN.

**Switch Configuration:**
```
Switch(config)# interface gigabitethernet 0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 99
Switch(config-if)# switchport trunk allowed vlan 10,20,30
Switch(config-if)# exit
```

**Router Configuration:**
```
Router(config)# interface gigabitethernet 0/0
Router(config-if)# no shutdown
Router(config-if)# exit

! VLAN 10 subinterface
Router(config)# interface gigabitethernet 0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit

! VLAN 20 subinterface
Router(config)# interface gigabitethernet 0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit

! VLAN 30 subinterface
Router(config)# interface gigabitethernet 0/0.30
Router(config-subif)# encapsulation dot1Q 30
Router(config-subif)# ip address 192.168.30.1 255.255.255.0
Router(config-subif)# exit
```

**Verification:**
```
Router# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up
GigabitEthernet0/0.10  192.168.10.1    YES manual up                    up
GigabitEthernet0/0.20  192.168.20.1    YES manual up                    up
GigabitEthernet0/0.30  192.168.30.1    YES manual up                    up

Router# show vlans

Virtual LAN ID:  10 (IEEE 802.1Q Encapsulation)
   vLAN Trunk Interface:   GigabitEthernet0/0.10
   Protocols Configured:   Address:              Received:        Transmitted:
           IP              192.168.10.1                324                 412
```

### 5.7.2 Layer 3 Switch (Switched Virtual Interface)

Modern approach using SVIs for inter-VLAN routing.

**Enable IP Routing:**
```
Switch(config)# ip routing
```

**Create SVIs:**
```
Switch(config)# interface vlan 10
Switch(config-if)# description SALES VLAN
Switch(config-if)# ip address 192.168.10.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

Switch(config)# interface vlan 20
Switch(config-if)# description ENGINEERING VLAN
Switch(config-if)# ip address 192.168.20.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

Switch(config)# interface vlan 30
Switch(config-if)# description HR VLAN
Switch(config-if)# ip address 192.168.30.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

**Verify Routing:**
```
Switch# show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

C    192.168.10.0/24 is directly connected, Vlan10
C    192.168.20.0/24 is directly connected, Vlan20
C    192.168.30.0/24 is directly connected, Vlan30
```

### 5.7.3 Access Control Between VLANs

**Basic ACL Example:**
```
! Permit HR VLAN to access all VLANs
! Deny other VLANs from accessing HR VLAN
Switch(config)# ip access-list extended PROTECT_HR
Switch(config-ext-nacl)# permit ip 192.168.30.0 0.0.0.255 any
Switch(config-ext-nacl)# deny ip any 192.168.30.0 0.0.0.255
Switch(config-ext-nacl)# permit ip any any
Switch(config-ext-nacl)# exit

Switch(config)# interface vlan 30
Switch(config-if)# ip access-group PROTECT_HR out
Switch(config-if)# exit
```

## 5.8 Troubleshooting VLANs

### 5.8.1 Common Issues and Solutions

**Issue 1: Devices in Same VLAN Cannot Communicate**

**Diagnosis:**
```
Switch# show vlan brief
Switch# show interface gigabitethernet 0/5 switchport
Switch# show mac address-table vlan 10
```

**Common Causes:**
- Incorrect VLAN assignment
- Port in wrong mode (trunk instead of access)
- Port administratively down
- VLAN not created on switch

**Solution:**
```
Switch(config)# interface gigabitethernet 0/5
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# no shutdown
```

**Issue 2: Trunk Not Passing VLAN Traffic**

**Diagnosis:**
```
Switch# show interfaces trunk
Switch# show interfaces gigabitethernet 0/24 switchport
Switch# show spanning-tree vlan 10
```

**Common Causes:**
- VLAN not in allowed list
- Native VLAN mismatch
- Trunk not formed (DTP issue)
- STP blocking port

**Solution:**
```
Switch(config)# interface gigabitethernet 0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan add 10
Switch(config-if)# switchport trunk native vlan 99
```

**Issue 3: Inter-VLAN Routing Not Working**

**Diagnosis:**
```
Switch# show ip interface brief
Switch# show ip route
Switch# show interface vlan 10
```

**Common Causes:**
- IP routing not enabled
- SVI shutdown
- Missing default gateway on clients
- ACL blocking traffic

**Solution:**
```
Switch(config)# ip routing
Switch(config)# interface vlan 10
Switch(config-if)# no shutdown
```

### 5.8.2 Verification Commands

```
! VLAN database and port assignments
Switch# show vlan brief
Switch# show vlan id 10

! Specific port VLAN configuration
Switch# show interfaces gigabitethernet 0/5 switchport

! Trunk status
Switch# show interfaces trunk

! MAC address table
Switch# show mac address-table
Switch# show mac address-table vlan 10

! VTP information
Switch# show vtp status
Switch# show vtp password

! Inter-VLAN routing (Layer 3 switches)
Switch# show ip interface brief
Switch# show ip route
```

### 5.8.3 Debug Commands

```
! VTP debugging
Switch# debug sw-vlan vtp events
Switch# debug sw-vlan vtp packets

! Trunk negotiation
Switch# debug sw-vlan trunk

! Turn off debugging
Switch# undebug all
```

## 5.9 VLAN Design Best Practices

### 5.9.1 Planning Guidelines

1. **VLAN Numbering Scheme**: Use consistent, logical numbering
   - 10-99: User VLANs
   - 100-199: Server VLANs
   - 200-299: Voice VLANs
   - 900-999: Management and infrastructure

2. **VLAN Size**: Keep broadcast domains reasonable
   - Typically 200-500 hosts per VLAN
   - Depends on application requirements and traffic patterns

3. **Security VLANs**:
   - Separate management VLAN (not VLAN 1)
   - Native VLAN unused (not VLAN 1)
   - Guest VLAN isolated from corporate network

### 5.9.2 Security Hardening

```
! Disable DTP on all access ports
Switch(config)# interface range gigabitethernet 0/1-20
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport nonegotiate
Switch(config-if-range)# exit

! Change native VLAN
Switch(config)# interface range gigabitethernet 0/23-24
Switch(config-if-range)# switchport trunk native vlan 999
Switch(config-if-range)# exit

! Shutdown unused ports and assign to parking VLAN
Switch(config)# vlan 666
Switch(config-vlan)# name PARKING
Switch(config-vlan)# exit

Switch(config)# interface range gigabitethernet 0/21-22
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 666
Switch(config-if-range)# shutdown
Switch(config-if-range)# exit

! Disable VTP or use transparent mode
Switch(config)# vtp mode transparent
```

### 5.9.3 Documentation Template

```
VLAN_ID | VLAN_Name     | Network         | Gateway      | Purpose
--------|---------------|-----------------|--------------|------------------
10      | SALES         | 192.168.10.0/24 | .1           | Sales department
20      | ENGINEERING   | 192.168.20.0/24 | .1           | Engineering
30      | HR            | 192.168.30.0/24 | .1           | Human Resources
99      | MANAGEMENT    | 192.168.99.0/24 | .1           | Switch management
150     | VOICE         | 10.10.150.0/24  | .1           | VoIP phones
666     | PARKING       | N/A             | N/A          | Unused ports
999     | NATIVE        | N/A             | N/A          | Trunk native VLAN
```

## 5.10 Real-World Scenario

### Small Office VLAN Deployment

**Requirements:**
- 3 departments: Sales (12 users), IT (5 users), Guest (WiFi)
- VoIP phones for all staff
- Secure management network
- 2 switches connected via trunk

**Design:**
```
VLAN 10: Sales (192.168.10.0/24)
VLAN 20: IT (192.168.20.0/24)
VLAN 30: Guest (192.168.30.0/24)
VLAN 100: Voice (10.10.100.0/24)
VLAN 99: Management (192.168.99.0/24)
VLAN 999: Native (unused)
```

**Core Switch Configuration:**
```
! Layer 3 switch with routing
CoreSwitch(config)# ip routing
CoreSwitch(config)# vtp mode transparent

! Create VLANs
CoreSwitch(config)# vlan 10
CoreSwitch(config-vlan)# name SALES
CoreSwitch(config-vlan)# vlan 20
CoreSwitch(config-vlan)# name IT
CoreSwitch(config-vlan)# vlan 30
CoreSwitch(config-vlan)# name GUEST
CoreSwitch(config-vlan)# vlan 100
CoreSwitch(config-vlan)# name VOICE
CoreSwitch(config-vlan)# vlan 99
CoreSwitch(config-vlan)# name MANAGEMENT
CoreSwitch(config-vlan)# vlan 999
CoreSwitch(config-vlan)# name NATIVE
CoreSwitch(config-vlan)# exit

! Configure SVIs
CoreSwitch(config)# interface vlan 10
CoreSwitch(config-if)# ip address 192.168.10.1 255.255.255.0
CoreSwitch(config-if)# no shutdown
CoreSwitch(config-if)# interface vlan 20
CoreSwitch(config-if)# ip address 192.168.20.1 255.255.255.0
CoreSwitch(config-if)# no shutdown
CoreSwitch(config-if)# interface vlan 30
CoreSwitch(config-if)# ip address 192.168.30.1 255.255.255.0
CoreSwitch(config-if)# no shutdown
CoreSwitch(config-if)# interface vlan 100
CoreSwitch(config-if)# ip address 10.10.100.1 255.255.255.0
CoreSwitch(config-if)# no shutdown
CoreSwitch(config-if)# interface vlan 99
CoreSwitch(config-if)# ip address 192.168.99.1 255.255.255.0
CoreSwitch(config-if)# no shutdown
CoreSwitch(config-if)# exit

! Trunk to access switch
CoreSwitch(config)# interface gigabitethernet 0/24
CoreSwitch(config-if)# switchport trunk encapsulation dot1q
CoreSwitch(config-if)# switchport mode trunk
CoreSwitch(config-if)# switchport trunk native vlan 999
CoreSwitch(config-if)# switchport trunk allowed vlan 10,20,30,99,100
CoreSwitch(config-if)# switchport nonegotiate
CoreSwitch(config-if)# exit
```

**Access Switch Configuration:**
```
AccessSwitch(config)# vtp mode transparent

! Sales ports with voice
AccessSwitch(config)# interface range gigabitethernet 0/1-12
AccessSwitch(config-if-range)# switchport mode access
AccessSwitch(config-if-range)# switchport access vlan 10
AccessSwitch(config-if-range)# switchport voice vlan 100
AccessSwitch(config-if-range)# spanning-tree portfast
AccessSwitch(config-if-range)# exit

! IT ports with voice
AccessSwitch(config)# interface range gigabitethernet 0/13-17
AccessSwitch(config-if-range)# switchport mode access
AccessSwitch(config-if-range)# switchport access vlan 20
AccessSwitch(config-if-range)# switchport voice vlan 100
AccessSwitch(config-if-range)# spanning-tree portfast
AccessSwitch(config-if-range)# exit

! Guest WiFi AP
AccessSwitch(config)# interface gigabitethernet 0/18
AccessSwitch(config-if)# switchport mode access
AccessSwitch(config-if)# switchport access vlan 30
AccessSwitch(config-if)# exit

! Trunk to core
AccessSwitch(config)# interface gigabitethernet 0/24
AccessSwitch(config-if)# switchport mode trunk
AccessSwitch(config-if)# switchport trunk native vlan 999
AccessSwitch(config-if)# switchport trunk allowed vlan 10,20,30,99,100
AccessSwitch(config-if)# switchport nonegotiate
AccessSwitch(config-if)# exit
```

This comprehensive VLAN configuration provides segmentation, security, and scalability for a small office environment.

## Summary

VLANs are essential for network segmentation, security, and efficient traffic management. Key takeaways:

- VLANs create logical broadcast domains
- Access ports for end devices, trunk ports for inter-switch links
- Change native VLAN from default for security
- Use VTP transparent mode or disable VTP
- Layer 3 switches provide efficient inter-VLAN routing
- Follow security best practices: disable DTP, secure management VLAN, document everything

In the next chapter, we'll explore Spanning Tree Protocol (STP), which prevents loops in redundant switched networks.
