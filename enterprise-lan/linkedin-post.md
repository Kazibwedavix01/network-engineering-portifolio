# LinkedIn Project Post

I’m pleased to share a hands-on Enterprise LAN Engineering project I completed using Cisco Packet Tracer.

The project involved designing, configuring, securing, and testing a small departmental network built with a Cisco 2911 router and two Cisco 2960 switches.

**What I implemented:**
- Departmental VLAN segmentation for ADMIN, FINANCE, and IT.
- 802.1Q trunking between the router and switches.
- Router-on-a-stick inter-VLAN routing.
- DHCP services for automatic IPv4 addressing.
- Extended ACL rules to block communication between ADMIN and FINANCE while preserving permitted connectivity to IT.
- SSH v2 remote administration with privilege-level-15 access.
- PVST verification, PortFast, and BPDU Guard on end-device ports.
- Unused switch-port shutdown and assignment to a parking VLAN.

**Verification:** I tested permitted and denied traffic paths, checked DHCP leases and ACL counters, verified trunk and VLAN states, confirmed router subinterfaces were up/up, and successfully authenticated to all three network devices over SSH.

This project strengthened my practical understanding of enterprise switching, routing, access control, device hardening, and network troubleshooting.

The work was completed in a **Cisco Packet Tracer simulation**, not a live production environment.

Project documentation and sanitized configuration references:
https://github.com/Kazibwedavix01/network-engineering-portifolio/tree/main/enterprise-lan

I’m continuing to build practical skills across enterprise networking, routing, security, and ISP/WAN technologies.

#Networking #Cisco #NetworkEngineering #CCNA #PacketTracer #EnterpriseNetworking #ITInfrastructure
