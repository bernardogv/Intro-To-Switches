# Glossary of Networking and Switching Terms

## A

**AAA (Authentication, Authorization, Accounting)** - Security framework that controls access to network resources and tracks user activity.

**Access Port** - A switch port that carries traffic for only one VLAN (untagged).

**ACL (Access Control List)** - A set of rules that filter network traffic based on source/destination IP, port, or protocol.

**Active-Active** - A redundancy configuration where multiple devices simultaneously forward traffic (e.g., GLBP).

**Active-Standby** - A redundancy configuration where one device forwards traffic while another waits in standby mode (e.g., HSRP).

**Administrative Distance (AD)** - A numerical value (0-255) that indicates the trustworthiness of a routing protocol. Lower values are preferred.

**ARP (Address Resolution Protocol)** - Protocol that maps IP addresses to MAC addresses on a local network.

**ASIC (Application-Specific Integrated Circuit)** - Specialized hardware chip designed for high-speed packet forwarding in switches.

**Auto-MDIX** - Automatic Medium-Dependent Interface Crossover - feature that automatically detects and corrects cable type (straight-through vs crossover).

**Auto-Negotiation** - Process where two network devices automatically negotiate speed and duplex settings.

## B

**Backbone** - The core network infrastructure that interconnects multiple network segments or buildings.

**Bandwidth** - The maximum data transfer rate of a network link, typically measured in bits per second (bps).

**BFD (Bidirectional Forwarding Detection)** - Protocol for rapid failure detection on network links (sub-second).

**BPDU (Bridge Protocol Data Unit)** - Frames sent by switches running Spanning Tree Protocol to detect and prevent loops.

**BPDU Guard** - STP feature that disables a port if it receives a BPDU (prevents accidental loops from rogue switches).

**Broadcast Domain** - A logical network segment where any broadcast packet sent by a device reaches all other devices.

**Broadcast Storm** - Network condition where excessive broadcast traffic saturates the network, causing performance degradation.

## C

**CAM (Content Addressable Memory) Table** - Hardware table in switches that stores MAC address-to-port mappings for fast forwarding.

**CEF (Cisco Express Forwarding)** - Hardware-based packet forwarding mechanism that uses FIB and adjacency tables for high-speed routing.

**CDP (Cisco Discovery Protocol)** - Cisco proprietary protocol for discovering directly connected Cisco devices.

**Collision Domain** - Network segment where packet collisions can occur (relevant for half-duplex Ethernet).

**CoS (Class of Service)** - 3-bit field in the 802.1Q VLAN tag used for Layer 2 QoS marking (values 0-7).

**CRC (Cyclic Redundancy Check)** - Error-detection code used in Ethernet frames to detect transmission errors.

**Cut-Through Switching** - Forwarding method where a switch begins forwarding a frame before receiving it entirely (low latency).

## D

**DAI (Dynamic ARP Inspection)** - Security feature that validates ARP packets against the DHCP snooping binding table to prevent ARP poisoning.

**DCB (Data Center Bridging)** - Set of enhancements to Ethernet for lossless data center networks (includes PFC, ETS, DCBX).

**Default Gateway** - The router IP address used by hosts to send traffic destined for other networks.

**DHCP (Dynamic Host Configuration Protocol)** - Protocol that automatically assigns IP addresses and network configuration to devices.

**DHCP Snooping** - Security feature that filters untrusted DHCP messages and builds a binding table of IP-MAC-port mappings.

**DTP (Dynamic Trunking Protocol)** - Cisco proprietary protocol for automatic trunk negotiation between switches.

**DSCP (Differentiated Services Code Point)** - 6-bit field in the IP header used for Layer 3 QoS marking (64 possible values).

**Duplex** - Communication mode: half-duplex (one direction at a time) or full-duplex (simultaneous bidirectional).

## E

**EAP (Extensible Authentication Protocol)** - Authentication framework used in 802.1X for network access control.

**ECMP (Equal-Cost Multi-Path)** - Routing technique that distributes traffic across multiple equal-cost paths.

**ECN (Explicit Congestion Notification)** - Mechanism that marks packets to indicate network congestion instead of dropping them.

**EtherChannel** - Link aggregation technology that combines multiple physical links into one logical link (Cisco term for LACP/PAgP).

**EVPN (Ethernet VPN)** - BGP-based control plane for VXLAN that advertises MAC addresses and provides multi-homing.

**err-disabled** - Port state where a switch automatically disables a port due to a security or configuration violation.

## F

**Fabric** - Network architecture where all devices are interconnected (e.g., spine-leaf fabric in data centers).

**FCoE (Fibre Channel over Ethernet)** - Protocol that encapsulates Fibre Channel frames over Ethernet networks.

**FHRP (First-Hop Redundancy Protocol)** - Generic term for gateway redundancy protocols (HSRP, VRRP, GLBP).

**FIB (Forwarding Information Base)** - Hardware table used by CEF containing IP destination prefixes and next-hop information.

**FlexLink** - Cisco feature providing simple link redundancy without Spanning Tree Protocol.

## G

**GLBP (Gateway Load Balancing Protocol)** - Cisco proprietary FHRP that provides both redundancy and load balancing.

**gNMI (gRPC Network Management Interface)** - Modern network management protocol using gRPC for configuration and telemetry.

## H

**Hot-Swappable** - Components (modules, power supplies, fans) that can be replaced without powering down the device.

**HSRP (Hot Standby Router Protocol)** - Cisco proprietary FHRP providing gateway redundancy.

**Hub** - Legacy Layer 1 device that broadcasts all traffic to all ports (replaced by switches).

## I

**IGMP (Internet Group Management Protocol)** - Protocol used by hosts to join/leave multicast groups.

**IGMP Snooping** - Switch feature that forwards multicast traffic only to ports with interested receivers.

**iSCSI (Internet Small Computer Systems Interface)** - IP-based storage protocol that encapsulates SCSI commands over Ethernet.

**Inter-VLAN Routing** - Routing traffic between different VLANs (performed by Layer 3 switch or router).

## J

**Jitter** - Variation in packet arrival times (critical for real-time applications like VoIP).

**Jumbo Frames** - Ethernet frames larger than standard 1500-byte MTU (typically 9000 bytes).

## L

**LACP (Link Aggregation Control Protocol)** - IEEE 802.3ad standard protocol for dynamically negotiating link aggregation.

**Latency** - Time delay for a packet to travel from source to destination.

**Layer 2 Switch** - Switch that forwards frames based on MAC addresses (Data Link Layer).

**Layer 3 Switch** - Multilayer switch capable of both Layer 2 switching and Layer 3 routing.

**Link Aggregation** - Combining multiple physical links into one logical link for increased bandwidth and redundancy.

**LLDP (Link Layer Discovery Protocol)** - IEEE 802.1AB standard protocol for discovering directly connected devices.

**Loopback Interface** - Virtual interface used for management and routing protocols (always up/up).

## M

**MAB (MAC Authentication Bypass)** - Authentication method using MAC addresses for devices that don't support 802.1X.

**MAC Address** - 48-bit hardware address uniquely identifying network interface cards (format: XX:XX:XX:XX:XX:XX).

**MAC Flooding** - Attack that overflows a switch's CAM table by sending frames with numerous fake source MAC addresses.

**MEC (Multi-Access Edge Computing)** - Distributed computing architecture that brings compute resources closer to network edge.

**MST (Multiple Spanning Tree)** - IEEE 802.1s standard that maps multiple VLANs to spanning tree instances for efficiency.

**MTU (Maximum Transmission Unit)** - Largest packet size (in bytes) that can be transmitted without fragmentation.

**Multicast** - One-to-many communication where a single packet is delivered to multiple destinations.

## N

**NAT (Network Address Translation)** - Translates private IP addresses to public IP addresses (router function, not switch).

**Native VLAN** - Untagged VLAN on an 802.1Q trunk port (default VLAN 1).

**NetFlow** - Cisco network protocol for collecting IP traffic information and monitoring network traffic.

**NTP (Network Time Protocol)** - Protocol for synchronizing clocks across network devices.

## O

**OpenFlow** - Protocol for SDN that separates control plane (controller) from data plane (switches).

**OSPF (Open Shortest Path First)** - Link-state routing protocol (RFC 2328) commonly used in enterprise networks.

**Oversubscription** - Ratio of bandwidth available to downstream devices versus uplink bandwidth (e.g., 20:1).

## P

**P4** - Programming language for defining custom packet processing behavior in programmable switches.

**PAgP (Port Aggregation Protocol)** - Cisco proprietary protocol for negotiating link aggregation.

**PAM4 (Pulse Amplitude Modulation)** - Modulation technique using 4 signal levels to double data rates (used in 400G Ethernet).

**PBR (Policy-Based Routing)** - Routing based on criteria other than destination IP (source, protocol, port).

**PVST+ (Per-VLAN Spanning Tree Plus)** - Cisco proprietary STP variant running separate spanning tree instances per VLAN.

**PFC (Priority Flow Control)** - Per-class pause mechanism for lossless Ethernet (part of DCB).

**PoE (Power over Ethernet)** - Technology that delivers electrical power over Ethernet cables (IEEE 802.3af/at/bt).

**Port Security** - Feature that limits MAC addresses learned on a switch port to prevent MAC flooding attacks.

**PortFast** - STP feature that bypasses listening/learning states on access ports (instant transition to forwarding).

**Private VLAN** - Subdivides a VLAN into isolated sub-domains (isolated, community, promiscuous).

**Promiscuous Mode** - Port mode that receives all network traffic (used with port mirroring/SPAN).

## Q

**QoS (Quality of Service)** - Techniques for prioritizing certain types of network traffic over others.

**Queuing** - Process of buffering packets and determining transmission order based on priority.

## R

**RADIUS (Remote Authentication Dial-In User Service)** - Protocol for centralized authentication, authorization, and accounting.

**RSPAN (Remote SPAN)** - Port mirroring technique that spans traffic across multiple switches using a dedicated VLAN.

**RSTP (Rapid Spanning Tree Protocol)** - IEEE 802.1w - faster convergence version of STP (1-2 seconds vs 30-50 seconds).

**Root Bridge** - Switch elected as the root in Spanning Tree Protocol topology (all paths calculated from root).

**Root Guard** - STP protection feature preventing unauthorized switches from becoming root bridge.

**Router-on-a-Stick** - Inter-VLAN routing method using a single router interface with subinterfaces for each VLAN.

## S

**SAI (Switch Abstraction Interface)** - Hardware abstraction layer for programming switch ASICs (used in SONiC).

**SASE (Secure Access Service Edge)** - Cloud-delivered service converging networking and security.

**SDN (Software-Defined Networking)** - Network architecture decoupling control plane from data plane.

**SD-WAN (Software-Defined Wide Area Network)** - SDN application for WAN connectivity with dynamic path selection.

**sFlow** - Industry-standard traffic monitoring technology (similar to NetFlow).

**SGT (Security Group Tag)** - 16-bit tag assigned to packets for policy enforcement (Cisco TrustSec).

**SNMP (Simple Network Management Protocol)** - Protocol for monitoring and managing network devices.

**SONiC (Software for Open Networking in the Cloud)** - Open-source network operating system developed by Microsoft.

**SPAN (Switch Port Analyzer)** - Cisco feature for mirroring traffic from source port(s) to destination port for monitoring.

**Spine-Leaf** - Data center network topology where leaf switches connect to all spine switches in full mesh.

**STP (Spanning Tree Protocol)** - IEEE 802.1D protocol preventing Layer 2 loops by blocking redundant paths.

**Store-and-Forward** - Switching method where entire frame is received and checked before forwarding (higher latency, error-free).

**SVI (Switched Virtual Interface)** - Virtual Layer 3 interface associated with a VLAN.

**Stacking** - Technology that combines multiple physical switches into single logical unit with unified management.

## T

**TACACS+ (Terminal Access Controller Access-Control System Plus)** - Cisco protocol for AAA (alternative to RADIUS).

**Telnet** - Insecure protocol for remote CLI access (port 23). Replaced by SSH in modern networks.

**Throughput** - Actual data transfer rate achieved in practice (may be lower than bandwidth due to overhead).

**ToR (Top-of-Rack)** - Switch placement architecture where each server rack has dedicated switch at top.

**Trunk Port** - Switch port carrying traffic for multiple VLANs (tagged with 802.1Q).

**TrustSec** - Cisco technology using Security Group Tags (SGT) for identity-based network segmentation.

## U

**UDLD (UniDirectional Link Detection)** - Protocol detecting unidirectional fiber links to prevent STP loops.

**Underlay** - Physical network infrastructure in overlay network architectures (e.g., IP network under VXLAN).

**Uplink** - Connection from lower-tier switch to higher-tier switch (e.g., access to distribution).

## V

**VACL (VLAN Access Control List)** - ACL applied to all traffic within or between VLANs on a switch.

**Virtual MAC** - MAC address used by FHRP protocols (HSRP, VRRP, GLBP) for the virtual gateway.

**VLAN (Virtual Local Area Network)** - Logical network segmentation creating separate broadcast domains on a physical switch.

**VLAN Hopping** - Attack exploiting DTP or double-tagging to access VLANs without authorization.

**VRF (Virtual Routing and Forwarding)** - Separate routing table instances for multi-tenancy on single device.

**VRRP (Virtual Router Redundancy Protocol)** - RFC 5798 standard FHRP providing gateway redundancy.

**VSS (Virtual Switching System)** - Cisco technology combining two physical switches into one logical switch.

**VXLAN (Virtual Extensible LAN)** - Encapsulation protocol extending Layer 2 networks over Layer 3 infrastructure (16M networks vs 4096 VLANs).

## W

**WAN (Wide Area Network)** - Network spanning large geographic areas (cities, countries, continents).

**White-Box Switch** - Generic switch hardware from ODMs, separate from network operating system (disaggregated networking).

**Wire Speed** - Maximum rate at which a switch can forward packets without drops (full line-rate performance).

## Z

**Zero-Touch Provisioning (ZTP)** - Automated device configuration without manual intervention (DHCP + TFTP/HTTP).

**Zero Trust** - Security model requiring verification for every access request regardless of network location.

---

## Common Acronyms Quick Reference

| Acronym | Full Name |
|---------|-----------|
| BGP | Border Gateway Protocol |
| BPD | Bits Per Day |
| CCNA | Cisco Certified Network Associate |
| CCNP | Cisco Certified Network Professional |
| CLI | Command Line Interface |
| DCB | Data Center Bridging |
| DNS | Domain Name System |
| EoR | End-of-Row |
| FTP | File Transfer Protocol |
| HTTP | Hypertext Transfer Protocol |
| HTTPS | HTTP Secure |
| IoT | Internet of Things |
| IP | Internet Protocol |
| IPv4 | Internet Protocol version 4 |
| IPv6 | Internet Protocol version 6 |
| ISP | Internet Service Provider |
| LAN | Local Area Network |
| MAC | Media Access Control |
| MPLS | Multiprotocol Label Switching |
| NIC | Network Interface Card |
| OSI | Open Systems Interconnection |
| QoS | Quality of Service |
| RFC | Request for Comments |
| SAN | Storage Area Network |
| SDN | Software-Defined Networking |
| SSH | Secure Shell |
| SSL | Secure Sockets Layer |
| TCP | Transmission Control Protocol |
| TFTP | Trivial File Transfer Protocol |
| TLS | Transport Layer Security |
| UDP | User Datagram Protocol |
| VLAN | Virtual Local Area Network |
| VoIP | Voice over IP |
| VPN | Virtual Private Network |
| WAN | Wide Area Network |

---

**Total Terms:** 150+ networking and switching terms defined
**Categories:** Protocols, Technologies, Security, Routing, Switching, Data Center, Emerging Technologies
