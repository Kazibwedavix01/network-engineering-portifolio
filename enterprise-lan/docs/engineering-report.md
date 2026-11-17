# Enterprise LAN Design, Implementation and Security
## Engineering Project Report

**Environment:** Cisco Packet Tracer  
**Project classification:** Enterprise LAN simulation / learning portfolio  
**Status:** Implemented and technically verified  
**Date:** 18 November 2026

## 1. Executive summary

This project designed and implemented a small departmental enterprise LAN using one Cisco 2911 router, two Cisco 2960 switches, and four end-host PCs. The network separates Administration, Finance, and IT into dedicated VLANs, provides inter-VLAN routing through router-on-a-stick, allocates client addresses through DHCP, applies an extended ACL to restrict communication between Administration and Finance, and enables SSH-based device administration.

The implementation was validated through CLI inspection, endpoint connectivity tests, ACL hit counters, DHCP lease inspection, trunk and spanning-tree checks, and SSH login tests. All three network devices were saved to startup-config after the final changes.

This is a simulated Packet Tracer lab. It is not evidence of a live production deployment.

## 2. Objectives and scope

### Objectives
1. Segment departments using VLANs.
2. Carry required VLANs across 802.1Q trunk links.
3. Provide inter-VLAN routing using router subinterfaces.
4. Provide automatic IPv4 configuration through DHCP.
5. Prevent direct IP communication between ADMIN and FINANCE.
6. Permit intended communication between the departments and IT.
7. Secure remote administration with SSH v2 and local privileged accounts.
8. Apply basic Layer 2 hardening to edge and unused ports.
9. Verify behavior and save device configurations.

### In scope
- One router, two access switches, four PCs.
- VLANs 10, 20, 30 and parking VLAN 999.
- IPv4 addressing and DHCP.
- Extended IPv4 ACL.
- SSH v2, PVST, PortFast, BPDU Guard, unused-port shutdown.

### Out of scope
- Live production deployment, physical cabling certification, centralized AAA, stateful firewalling, IDS/IPS, dynamic routing, high availability, wireless, IPv6, and formal penetration testing.

## 3. Topology and inventory

| Device | Model | Hostname | Role |
|---|---|---|---|
| Router | Cisco 2911 | R1-HEAD-OFFICE | Default gateways, inter-VLAN routing, DHCP, ACL, SSH |
| Switch | Cisco 2960 | SW1-ADMIN | Access switching and trunk aggregation |
| Switch | Cisco 2960 | SW2-DEPARTMENTS | IT access switching and trunk extension |
| End host | PC | PC-ADMIN | Administration endpoint |
| End host | PC | PC-FINANCE | Finance endpoint |
| End host | PC | PC-IT-01 | IT endpoint |
| End host | PC | PC-IT-02 | IT endpoint |

### Physical/logical links
- R1 G0/0 ↔ SW1 G0/1: 802.1Q trunk.
- SW1 G0/2 ↔ SW2 G0/1: 802.1Q trunk.
- SW1 Fa0/1 ↔ PC-ADMIN.
- SW1 Fa0/2 ↔ PC-FINANCE.
- SW2 Fa0/1 and Fa0/2 ↔ IT PCs.

The editable topology source is in [topology.mmd](../topology.mmd).

## 4. VLAN and IP addressing plan

| VLAN | Name | Network | Gateway | Usage |
|---:|---|---|---|---|
| 10 | ADMIN | 192.168.10.0/24 | 192.168.10.1 | Administration clients |
| 20 | FINANCE | 192.168.20.0/24 | 192.168.20.1 | Finance clients |
| 30 | IT | 192.168.30.0/24 | 192.168.30.1 | IT clients and switch management |
| 999 | PARKING | No SVI/subnet configured | None | Unused switch ports |

Switch management:
- SW1 VLAN 30 SVI: 192.168.30.2/24; default gateway 192.168.30.1.
- SW2 VLAN 30 SVI: 192.168.30.3/24; default gateway 192.168.30.1.

DHCP exclusions: 192.168.10.1–192.168.10.20, 192.168.20.1–192.168.20.20, and 192.168.30.1–192.168.30.20.

Observed DHCP leases:
- ADMIN: 192.168.10.21
- FINANCE: 192.168.20.21
- IT: 192.168.30.21 and 192.168.30.22

## 5. Implementation summary

### 5.1 VLANs and access ports
VLANs 10 (ADMIN), 20 (FINANCE), 30 (IT), and 999 (PARKING) were created on both switches. SW1 Fa0/1 is in VLAN 10 and Fa0/2 in VLAN 20. SW2 Fa0/1–Fa0/2 are in VLAN 30.

Unused SW1 Fa0/3–Fa0/24 and SW2 Fa0/3–Fa0/24 plus Gi0/2 were assigned to VLAN 999 and administratively shut down. This accounts for 45 unused switch ports.

### 5.2 Trunking
SW1 Gi0/1 (router-facing), SW1 Gi0/2 (inter-switch), and SW2 Gi0/1 (inter-switch) were verified as 802.1Q trunks. Native VLAN is 1. VLANs 1, 10, 20, 30, and 999 were active and in spanning-tree forwarding state on the inspected trunks.

The allowed VLAN range remains 1–1005. A narrower allowed list is a future improvement, not part of the verified configuration.

### 5.3 Inter-VLAN routing
R1 uses subinterfaces on G0/0:
- G0/0.10 — 192.168.10.1/24
- G0/0.20 — 192.168.20.1/24
- G0/0.30 — 192.168.30.1/24

Final `show ip interface brief` output confirmed the parent G0/0 and all three subinterfaces were up/up.

### 5.4 DHCP
Three DHCP pools provide addressing to ADMIN, FINANCE, and IT. The first 20 addresses in each /24 subnet are excluded from dynamic allocation. The four observed client bindings match the expected subnets and gateway design.

### 5.5 Department access policy
An extended ACL named `DEPARTMENT-SECURITY` contains:
- Deny IP from 192.168.10.0/24 to 192.168.20.0/24.
- Deny IP from 192.168.20.0/24 to 192.168.10.0/24.
- Permit IP any any.

The ACL was applied inbound on R1 G0/0.10 and G0/0.20. This is a stateless packet-filtering policy. It is not a stateful firewall and does not provide application-layer inspection.

Observed ACL counters included 8 matches for each directional deny rule and 121 matches for the permit rule at the time of inspection.

### 5.6 Secure management
SSH version 2 was enabled on R1, SW1, and SW2. VTY lines use local login and SSH-only transport. A local `networkadmin` account with privilege 15 was verified on all three devices. Remote logins from the IT PC succeeded and `show privilege` returned level 15 on each device.

Temporary test accounts were removed after the permanent account was tested. Passwords, hashes, and private key material are intentionally excluded from this repository.

### 5.7 Layer 2 hardening
- PVST was active. SW2 was the root bridge for VLANs 1, 10, 20, and 30; SW1 used Gi0/2 as its root port.
- PortFast and BPDU Guard were configured on PC-facing access ports only: SW1 Fa0/1–Fa0/2 and SW2 Fa0/1–Fa0/2.
- Unused switch ports were placed in VLAN 999 and shut down.
- Port security (sticky MAC / maximum MAC addresses) was not configured.

## 6. Verification and test results

| ID | Test | Expected behavior | Recorded result |
|---|---|---|---|
| T01 | ADMIN to FINANCE | Blocked | Blocked; destination unreachable from gateway |
| T02 | FINANCE to ADMIN | Blocked | Blocked; destination unreachable from gateway |
| T03 | ADMIN to IT | Permitted | 4/4 replies, 0% loss |
| T04 | FINANCE to IT | Permitted | Successful; user confirmed |
| T05 | IT to R1 gateway | Permitted | Successful; user confirmed |
| T06 | IT to SW1 management | Permitted | 4/4 replies, 0% loss |
| T07 | IT to SW2 management | Permitted | 4/4 replies, 0% loss |
| T08 | SSH to R1 | Authenticated admin access | Successful; privilege 15 |
| T09 | SSH to SW1 | Authenticated admin access | Successful; privilege 15 |
| T10 | SSH to SW2 | Authenticated admin access | Successful; privilege 15 |
| T11 | Router subinterfaces | Up/up | All three up/up |
| T12 | Trunks | 802.1Q, required VLANs forwarding | Verified on both switches |
| T13 | Unused ports | VLAN 999 and disabled | Verified on both switches |
| T14 | Configuration persistence | Running config copied to startup config | User confirmed all three saved |

See [verification-matrix.md](verification-matrix.md) for the recorded detail.

## 7. Security review and limitations

Implemented controls:
- Department VLAN segmentation.
- Explicit interdepartment ACL denies between ADMIN and FINANCE.
- SSH v2 with local privileged accounts.
- PortFast and BPDU Guard on edge ports.
- Unused switch port shutdown and parking VLAN.
- Configuration persistence.

Limitations and future improvements:
1. Restrict trunk allowed VLANs to only the VLANs required, consistently at both ends.
2. Consider a dedicated unused native VLAN and configure it consistently across each trunk after planning and testing.
3. Configure switch port security (maximum MAC addresses, sticky learning, violation action) where supported and appropriate.
4. Consider centralized AAA for larger environments.
5. Add stateful firewall controls if application-aware or stateful policy is required.
6. Add monitoring (Syslog, SNMP, NTP) and configuration backup.
7. Validate all behavior on the target physical platform and IOS version before production use.
8. Re-run a complete test suite after any change.

## 8. Operational handover

The lab was saved as `Enterprise-LAN-Secure-Departmental-Network.pkt` in the user's Packet Tracer workspace. Device running configurations were copied to startup-config on R1, SW1, and SW2 after the final changes.

The Packet Tracer file was not attached to this GitHub publication. Screenshots and raw CLI transcript files should be added only after review and redaction.

## 9. Conclusion

The simulated network met its core objectives: departmental segmentation, DHCP-based addressing, router-on-a-stick routing, controlled ADMIN/FINANCE isolation, SSH administration, and basic Layer 2 hardening. Verification confirmed that intended paths remained reachable while the restricted departmental paths were blocked.

The project demonstrates a practical end-to-end enterprise LAN workflow: design, configuration, verification, security hardening, persistence, and technical documentation.
