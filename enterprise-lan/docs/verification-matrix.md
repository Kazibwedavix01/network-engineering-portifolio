# Enterprise LAN — Verification Matrix

This matrix records results reported during the Packet Tracer lab validation. It distinguishes command-output evidence from user-confirmed success.

| ID | Source | Destination / command | Expected | Result |
|---|---|---|---|---|
| T01 | PC-ADMIN | PC-FINANCE (192.168.20.21) | Denied by ACL | 100% loss; gateway returned destination unreachable |
| T02 | PC-FINANCE | PC-ADMIN (192.168.10.21) | Denied by ACL | 100% loss; gateway returned destination unreachable |
| T03 | PC-ADMIN | PC-IT-01 (192.168.30.21) | Reachable | 4/4 replies; 0% loss |
| T04 | PC-FINANCE | PC-IT-01 | Reachable | Success confirmed by operator |
| T05 | PC-IT-01 | R1 (192.168.30.1) | Reachable | Success confirmed by operator |
| T06 | PC-IT-01 | SW1 (192.168.30.2) | Reachable | 4/4 replies; 0% loss |
| T07 | PC-IT-01 | SW2 (192.168.30.3) | Reachable | 4/4 replies; 0% loss |
| T08 | PC-IT-01 | SSH to R1 | Login, privilege 15 | Successful; privilege 15 |
| T09 | PC-IT-01 | SSH to SW1 | Login, privilege 15 | Successful; privilege 15 |
| T10 | PC-IT-01 | SSH to SW2 | Login, privilege 15 | Successful; privilege 15 |
| T11 | R1 | `show ip interface brief` | Three subinterfaces up/up | G0/0.10, .20, .30 all up/up |
| T12 | SW1/SW2 | `show interfaces trunk` | 802.1Q trunks and required VLANs forwarding | Verified on all three trunk interfaces |
| T13 | SW1/SW2 | `show vlan brief` | Correct access/parking VLAN assignments | Verified |
| T14 | SW1/SW2 | `show spanning-tree` | Stable PVST topology | SW2 root for VLANs 1, 10, 20, 30; SW1 Gi0/2 root port |
| T15 | R1 | `show ip dhcp binding` | Four client leases | Four expected bindings observed |
| T16 | R1 | `show ip access-lists` | ACL rules and hit counters | 8 matches on each deny; 121 on permit at time of check |
| T17 | R1/SW1/SW2 | `copy running-config startup-config` | Save final config | Operator confirmed all three saved |

## Interpretation

- T01 and T02 demonstrate the intended bidirectional ADMIN/FINANCE isolation.
- T03–T07 show that intended connectivity to IT and management interfaces remained available.
- T08–T10 confirm authenticated SSH access and privilege level 15 on all network devices.
- T11–T17 provide configuration-state evidence from device CLI outputs and operator confirmation.

## Evidence still recommended for a public portfolio

Add screenshots from Packet Tracer showing:
1. Full topology with device names and links.
2. VLAN table and trunk status.
3. DHCP bindings.
4. ACL rules and counters.
5. Successful and blocked ping tests.
6. SSH privilege output for all three devices.
7. Unused-port status.

Before uploading screenshots, inspect them for passwords, password hashes, usernames that should remain private, or other sensitive information.
