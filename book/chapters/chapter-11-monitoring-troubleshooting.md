# Chapter 11: Switch Monitoring and Troubleshooting

## Introduction

Even the most carefully configured network switch will eventually require troubleshooting. Port failures, performance degradation, configuration errors, and software bugs are inevitable in production environments. The difference between a five-minute outage and a five-hour disaster often comes down to monitoring, diagnostic tools, and systematic troubleshooting methodology.

This chapter covers essential monitoring techniques, powerful diagnostic commands, log analysis, and a structured approach to resolving common switch problems.

---

## 11.1 Proactive Monitoring Strategies

### SNMP (Simple Network Management Protocol)

**What SNMP Monitors:**
- Port status (up/down, errors, utilization)
- CPU and memory usage
- Temperature sensors
- Power supply status
- Configuration changes
- Security violations

**Basic SNMP Configuration:**

```cisco
! SNMPv2c (community-based, less secure)
Switch(config)# snmp-server community PublicRO RO
Switch(config)# snmp-server community PrivateRW RW
Switch(config)# snmp-server location "Building A, Floor 2, Rack 5"
Switch(config)# snmp-server contact "network-team@company.com"

! Enable traps
Switch(config)# snmp-server enable traps snmp linkdown linkup
Switch(config)# snmp-server enable traps port-security
Switch(config)# snmp-server enable traps config
Switch(config)# snmp-server host 192.168.100.50 version 2c PublicRO
```

**SNMPv3 (Recommended - Encrypted):**

```cisco
! Create SNMPv3 user with authentication and encryption
Switch(config)# snmp-server group ADMIN-GROUP v3 priv
Switch(config)# snmp-server user admin-user ADMIN-GROUP v3 auth sha AuthPass123 priv aes 128 PrivPass456

! Configure trap receiver
Switch(config)# snmp-server host 192.168.100.50 version 3 priv admin-user

! Enable specific traps
Switch(config)# snmp-server enable traps cpu threshold
Switch(config)# snmp-server enable traps vlan-membership
Switch(config)# snmp-server enable traps errdisable
```

**Verification:**

```cisco
Switch# show snmp
Chassis: JAE12345ABC
Contact: network-team@company.com
Location: Building A, Floor 2, Rack 5
19472 SNMP packets input
    0 Bad SNMP version errors
    8 Unknown community name
    0 Illegal operation for community name supplied

Switch# show snmp user
User name: admin-user
Engine ID: 800000090300001A2B3C4D5E
storage-type: nonvolatile        active
Authentication Protocol: SHA
Privacy Protocol: AES128
Group-name: ADMIN-GROUP
```

### NetFlow / sFlow: Traffic Analysis

**NetFlow** exports flow records (source/dest IP, ports, protocol, byte counts).

**Configuration:**

```cisco
! Define NetFlow exporter
Switch(config)# flow exporter NETFLOW-EXPORT
Switch(config-flow-exporter)# destination 192.168.100.60
Switch(config-flow-exporter)# transport udp 9996
Switch(config-flow-exporter)# exit

! Define flow monitor
Switch(config)# flow monitor NETFLOW-MON
Switch(config-flow-monitor)# exporter NETFLOW-EXPORT
Switch(config-flow-monitor)# record netflow ipv4 original-input
Switch(config-flow-monitor)# exit

! Apply to interface
Switch(config)# interface gigabitethernet0/1
Switch(config-if)# ip flow monitor NETFLOW-MON input
```

**What NetFlow Reveals:**
- Top talkers (bandwidth hogs)
- Application usage patterns
- Unusual traffic spikes
- Security anomalies (port scans, DDoS)

### Syslog: Centralized Logging

**Configure Syslog Server:**

```cisco
! Set logging level (0=emergencies, 7=debugging)
Switch(config)# logging trap informational

! Configure syslog server
Switch(config)# logging host 192.168.100.70
Switch(config)# logging source-interface vlan 1

! Buffer logs locally
Switch(config)# logging buffered 64000

! Console logging (can impact performance)
Switch(config)# logging console warnings

! Timestamp logs
Switch(config)# service timestamps log datetime msec
Switch(config)# service timestamps debug datetime msec
```

**Syslog Severity Levels:**

| Level | Keyword | Description | Example |
|-------|---------|-------------|---------|
| 0 | emergencies | System unusable | Hardware failure |
| 1 | alerts | Immediate action needed | Temperature critical |
| 2 | critical | Critical condition | Memory allocation failure |
| 3 | errors | Error messages | Interface down |
| 4 | warnings | Warning messages | Duplex mismatch |
| 5 | notifications | Normal but significant | Config change |
| 6 | informational | Informational | Interface up |
| 7 | debugging | Debug messages | Detailed packet info |

**Useful Syslog Filtering:**

```cisco
! View recent logs
Switch# show logging | include LINK-3-UPDOWN

! Count specific events
Switch# show logging | count STP

! Filter by interface
Switch# show logging | include Gi0/5
```

### SPAN (Switch Port Analyzer): Packet Capture

**Local SPAN - Mirror traffic to local port:**

```cisco
! Monitor port Gi0/5, send copy to Gi0/24 (connected to packet analyzer)
Switch(config)# monitor session 1 source interface gigabitethernet0/5
Switch(config)# monitor session 1 destination interface gigabitethernet0/24

! Monitor entire VLAN
Switch(config)# monitor session 2 source vlan 10
Switch(config)# monitor session 2 destination interface gigabitethernet0/24

! Monitor both directions
Switch(config)# monitor session 3 source interface gi0/5 both
Switch(config)# monitor session 3 destination interface gi0/24
```

**RSPAN (Remote SPAN) - Mirror traffic to remote switch:**

```cisco
! Create RSPAN VLAN (reserved for mirrored traffic)
Switch(config)# vlan 999
Switch(config-vlan)# remote-span
Switch(config-vlan)# exit

! Source switch configuration
Switch(config)# monitor session 1 source interface gi0/5
Switch(config)# monitor session 1 destination remote vlan 999

! Destination switch configuration
Switch(config)# monitor session 1 source remote vlan 999
Switch(config)# monitor session 1 destination interface gi0/24
```

**Verification:**

```cisco
Switch# show monitor session 1
Session 1
---------
Type                   : Local Session
Source Ports           :
    Both               : Gi0/5
Destination Ports      : Gi0/24
    Encapsulation      : Native
```

---

## 11.2 Essential Diagnostic Commands

### Interface Statistics

**Basic Interface Status:**

```cisco
Switch# show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Gi0/1     Workstation-001    connected    10         a-full  a-100 10/100/1000BaseTX
Gi0/2     Server-DB01        connected    20         a-full a-1000 10/100/1000BaseTX
Gi0/3                        notconnect   1            auto   auto 10/100/1000BaseTX
Gi0/4                        err-disabled 10           auto   auto 10/100/1000BaseTX
```

**Detailed Interface Information:**

```cisco
Switch# show interfaces gigabitethernet0/1
GigabitEthernet0/1 is up, line protocol is up (connected)
  Hardware is Gigabit Ethernet, address is 0050.56be.0001
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  Full-duplex, 1000Mb/s, media type is 10/100/1000BaseTX
  Input flow-control is off, output flow-control is unsupported
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:00:00, output 00:00:02, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  5 minute input rate 2000 bits/sec, 3 packets/sec
  5 minute output rate 1000 bits/sec, 2 packets/sec
     1234567 packets input, 89012345 bytes, 0 no buffer
     Received 12345 broadcasts (10 multicasts)
     0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 watchdog, 10 multicast, 0 pause input
     0 input packets with dribble condition detected
     2345678 packets output, 123456789 bytes, 0 underruns
     0 output errors, 0 collisions, 1 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier, 0 pause output
     0 output buffer failures, 0 output buffers swapped out
```

**Key Metrics to Monitor:**

- **Input errors:** Damaged frames (CRC errors, runts, giants)
- **Output errors:** Transmission problems (collisions, late collisions)
- **Drops:** Buffer overflows (input queue drops, output drops)
- **Collisions:** Half-duplex issues (should be 0 in full-duplex)
- **CRC errors:** Physical layer problems (bad cable, EMI)
- **Runts:** Frames < 64 bytes (collision fragments)
- **Giants:** Frames > 1518 bytes (misconfiguration)

### Error Analysis

**Check for Errors:**

```cisco
Switch# show interfaces counters errors
Port        Align-Err    FCS-Err   Xmit-Err    Rcv-Err  UnderSize OutDiscards
Gi0/1               0          0          0          0          0           0
Gi0/2               0         15          0          0          0           0  ← CRC errors
Gi0/3               0          0          0          0          0           0
```

**Common Error Patterns:**

1. **High CRC Errors:**
   - Bad cable or connector
   - EMI (electromagnetic interference)
   - Duplex mismatch
   - Action: Replace cable, check for EMI sources

2. **Collisions (Half-Duplex):**
   - Normal up to 10% utilization
   - Excessive collisions indicate network congestion
   - Action: Upgrade to full-duplex or segment network

3. **Late Collisions (Full-Duplex):**
   - Duplex mismatch (one side auto, other hard-coded)
   - Cable too long (>100m for Ethernet)
   - Action: Fix duplex settings on both ends

4. **Runts:**
   - Collision fragments
   - Faulty NIC
   - Action: Check for duplex mismatch, replace NIC

### MAC Address Table Troubleshooting

```cisco
! Show all learned MAC addresses
Switch# show mac address-table
          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
  10    0050.56be.0001    DYNAMIC     Gi0/1
  10    0050.56be.0002    DYNAMIC     Gi0/2
  20    0050.56be.1000    DYNAMIC     Gi0/5
Total Mac Addresses for this criterion: 3

! Show MAC addresses on specific interface
Switch# show mac address-table interface gigabitethernet0/1
          Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
  10    0050.56be.0001    DYNAMIC     Gi0/1

! Show MAC addresses for specific VLAN
Switch# show mac address-table vlan 10

! Check for MAC flapping (sign of loops or switch misconfiguration)
Switch# show mac address-table notification mac-move
```

### CPU and Memory Monitoring

**Check CPU Usage:**

```cisco
Switch# show processes cpu sorted
CPU utilization for five seconds: 15%/2%; one minute: 12%; five minutes: 10%
 PID Runtime(ms)   Invoked      uSecs   5Sec   1Min   5Min TTY Process
  95    1234567   8901234        138  12.00%  10.50%  9.00%   0 IP Input
  23      567890   1234567        460   2.00%   1.50%  1.00%   0 Hulc LED Process
```

**Check Memory:**

```cisco
Switch# show memory statistics
                Head    Total(b)     Used(b)     Free(b)   Lowest(b)  Largest(b)
Processor   65D1B000    524288000   123456000   400832000  380000000  390000000
      I/O   12000000     67108864    12345678    54763186   50000000   45000000

Switch# show processes memory sorted
PID  TTY  Allocated      Freed    Holding    Getbufs    Retbufs Process
  0    0   12345678   9876543    2469135          0          0 *Init*
  1    0    5678901   4567890    1111011          0          0 *Sched*
```

**Warning Signs:**
- CPU > 80% sustained
- Memory free < 20%
- Frequent memory allocation failures

---

## 11.3 Structured Troubleshooting Methodology

### The OSI Model Approach (Bottom-Up)

**Layer 1 (Physical):**
```cisco
! Check physical status
Switch# show interfaces gigabitethernet0/5 status
Port      Name               Status       Vlan       Duplex  Speed Type
Gi0/5                        notconnect   10           auto   auto 10/100/1000BaseTX

! Verify cable
- LED indicators (link light on?)
- Cable tester results
- Try different cable
- Check for physical damage

! Check transceiver
Switch# show interfaces gigabitethernet0/24 transceiver
ITU Channel not available (Wavelength not available),
Transceiver is internally calibrated.
If device is externally calibrated, only calibrated values are printed.
++ : high alarm, +  : high warning, -  : low warning, -- : low alarm.
NA or N/A: not applicable, Rx power: received power, Tx power: transmitted power.

                              Optical   Optical
          Temperature  Voltage  TX Power  RX Power
Port      (Celsius)    (Volts)  (dBm)     (dBm)
--------- -----------  -------  --------  --------
Gi0/24      34.5        3.30     -2.5      -3.1
```

**Layer 2 (Data Link):**
```cisco
! Check VLAN assignment
Switch# show vlan brief | include Gi0/5
10   Engineering                   active    Gi0/5, Gi0/6

! Verify spanning tree
Switch# show spanning-tree interface gigabitethernet0/5
Vlan             Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
VLAN0010         Desg FWD 4         128.5    P2p

! Check for err-disabled
Switch# show interfaces status err-disabled
Port      Name               Status       Reason               Err-disabled Vlans
Gi0/4                        err-disabled psecure-violation
```

**Layer 3 (Network):**
```cisco
! Verify IP configuration
Switch# show ip interface vlan 10
Vlan10 is up, line protocol is up
  Internet address is 10.1.10.1/24
  Broadcast address is 255.255.255.255

! Check routing
Switch# show ip route 10.2.20.0
Routing entry for 10.2.20.0/24
  Known via "ospf 1", distance 110, metric 2
  Last update from 192.168.100.2 on GigabitEthernet0/24, 00:15:23 ago

! Test connectivity
Switch# ping 10.2.20.1 source vlan 10
```

### Common Problems and Solutions

**Problem 1: Port Flapping (Constant Up/Down)**

```cisco
! Check logs
Switch# show logging | include LINK-3-UPDOWN.*Gi0/5
%LINK-3-UPDOWN: Interface GigabitEthernet0/5, changed state to down
%LINK-3-UPDOWN: Interface GigabitEthernet0/5, changed state to up
%LINK-3-UPDOWN: Interface GigabitEthernet0/5, changed state to down

! Possible causes:
1. Bad cable/connector
2. Duplex mismatch
3. EtherChannel misconfiguration
4. Power issues (PoE budget exceeded)
5. Faulty NIC

! Troubleshooting steps:
! Check interface details
Switch# show interfaces gi0/5
  Full-duplex, 100Mb/s  ← Check if auto-negotiation failed

! Check duplex on both ends
Switch(config)# interface gi0/5
Switch(config-if)# duplex full
Switch(config-if)# speed 1000

! Replace cable and test
```

**Problem 2: Slow Performance / High Latency**

```cisco
! Check interface utilization
Switch# show interfaces gi0/5 | include rate
  5 minute input rate 950000000 bits/sec, 12000 packets/sec  ← Near 1Gbps limit
  5 minute output rate 920000000 bits/sec, 11500 packets/sec

! Check for errors
Switch# show interfaces gi0/5 | include errors
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 output errors, 0 collisions, 0 interface resets

! Check for queue drops
Switch# show interfaces gi0/5 | include drops
  Input queue: 0/75/12345/0 (size/max/drops/flushes)  ← Input drops!
  Total output drops: 8901  ← Output drops!

! Solutions:
1. Upgrade link bandwidth (1G → 10G)
2. Implement QoS to prioritize critical traffic
3. Use link aggregation (EtherChannel)
```

**Problem 3: VLAN Connectivity Issues**

```cisco
! Verify VLAN exists
Switch# show vlan id 10
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
10   Engineering                      active    Gi0/1, Gi0/2, Gi0/3

! Check port VLAN assignment
Switch# show interfaces gi0/5 switchport
Name: Gi0/5
Switchport: Enabled
Administrative Mode: access
Operational Mode: access
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (Engineering)

! Verify trunk allows VLAN
Switch# show interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Gi0/24      on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/24      1-4094  ← Verify VLAN 10 is allowed

! Check spanning tree isn't blocking
Switch# show spanning-tree vlan 10 | include Gi0/5
Gi0/5               Desg FWD 19        128.5    P2p  ← Should be FWD, not BLK
```

**Problem 4: err-disabled Port**

```cisco
! Identify reason
Switch# show interfaces status err-disabled
Port      Name               Status       Reason               Err-disabled Vlans
Gi0/4                        err-disabled bpduguard

! Common causes:
- Port security violation
- BPDU guard triggered (STP BPDU received on PortFast port)
- Link aggregation misconfiguration
- Duplex mismatch
- UDLD (UniDirectional Link Detection)

! Re-enable port
Switch(config)# interface gi0/4
Switch(config-if)# shutdown
Switch(config-if)# no shutdown

! Enable automatic recovery (optional)
Switch(config)# errdisable recovery cause bpduguard
Switch(config)# errdisable recovery interval 300
```

---

## 11.4 Advanced Troubleshooting Tools

### Debugging (Use with Caution!)

**⚠️ WARNING:** Debug commands generate massive CPU load. Use only in controlled environments.

```cisco
! Debug specific interface
Switch# debug interface gigabitethernet0/5

! Debug spanning tree
Switch# debug spanning-tree events

! Debug VLAN
Switch# debug vlan events

! Disable all debugging
Switch# undebug all

! Show active debugs
Switch# show debugging
```

**Best Practices:**
1. Use during maintenance windows
2. Apply to specific interfaces/VLANs, not globally
3. Monitor CPU: `show processes cpu | include CPU`
4. Always have `undebug all` ready
5. Log to buffer, not console: `no logging console`

### Conditional Debugging

```cisco
! Debug only traffic from specific MAC
Switch# debug condition mac 0050.56be.0001

! Debug only specific interface
Switch# debug condition interface gigabitethernet0/5

! Show active conditions
Switch# show debug condition

! Remove condition
Switch# undebug condition <condition-id>
```

### Embedded Event Manager (EEM)

Automate responses to network events.

**Example: Alert on high CPU**

```cisco
! Define event and action
Switch(config)# event manager applet HIGH-CPU
Switch(config-applet)# event snmp oid 1.3.6.1.4.1.9.9.109.1.1.1.1.5.1 get-type next entry-op gt entry-val 80 poll-interval 60
Switch(config-applet)# action 1.0 syslog msg "CPU exceeded 80%"
Switch(config-applet)# action 2.0 cli command "show processes cpu sorted"
Switch(config-applet)# action 3.0 mail server "192.168.100.80" to "admin@company.com" subject "Switch CPU Alert"
```

**Example: Auto-recover err-disabled ports**

```cisco
Switch(config)# event manager applet RECOVER-ERRDISABLE
Switch(config-applet)# event syslog pattern "%PM-4-ERR_DISABLE"
Switch(config-applet)# action 1.0 cli command "enable"
Switch(config-applet)# action 2.0 cli command "config t"
Switch(config-applet)# action 3.0 cli command "interface $interface"
Switch(config-applet)# action 4.0 cli command "shutdown"
Switch(config-applet)# action 5.0 wait 10
Switch(config-applet)# action 6.0 cli command "no shutdown"
```

---

## 11.5 Performance Baselines and Capacity Planning

### Establishing Baselines

**Collect Baseline Data:**

```cisco
! Interface statistics
Switch# show interfaces gigabitethernet0/1 | include rate
  5 minute input rate 5000 bits/sec, 10 packets/sec
  5 minute output rate 3000 bits/sec, 8 packets/sec

! CPU utilization
Switch# show processes cpu history
    60 seconds period
        80
        70
        60  *
        50  *  *
        40  *  *  *
        30  *  *  *  *
        20  *  *  *  *  *
        10  *  *  *  *  *  *
```

**What to Baseline:**
- Port utilization (5-minute averages)
- CPU usage (daily peaks)
- Memory utilization
- Error rates
- Temperature
- Power consumption

**Tools for Baseline Collection:**
- SNMP polling every 5 minutes
- NetFlow for traffic patterns
- Syslog for events
- Export to graphing tools (Grafana, PRTG, SolarWinds)

### Capacity Planning

**Signs You're Reaching Capacity:**

1. **Port utilization > 70% sustained**
   ```
   Solution: Upgrade bandwidth or add ports
   ```

2. **Input/output queue drops increasing**
   ```cisco
   Switch# show interfaces gi0/1 | include queue
   Input queue: 75/75/12345/0 (size/max/drops/flushes)  ← Queue full
   ```

3. **CPU > 80% during normal operation**
   ```
   Solution: Reduce features (ACLs, QoS policies), upgrade switch
   ```

4. **Memory free < 20%**
   ```cisco
   Switch# show memory statistics
   Processor   65D1B000   524288000   419430400   104857600  ← 20% free
   ```

---

## Key Takeaways

1. **Proactive monitoring** (SNMP, NetFlow, Syslog) prevents problems before users notice
2. **Interface errors** indicate physical layer issues; analyze counters systematically
3. **Structured troubleshooting** using the OSI model isolates problems quickly
4. **Common issues** (err-disabled, flapping, slow performance) have known solutions
5. **Debug commands** are powerful but dangerous; use conditionally and during maintenance
6. **Baselines** establish normal behavior, enabling anomaly detection
7. **EEM scripts** automate remediation for common problems

Effective monitoring and troubleshooting transform reactive firefighting into proactive network management, reducing downtime and improving user experience.

---

**Chapter 11 Complete** | Next: Chapter 12 - Advanced Switching Features
