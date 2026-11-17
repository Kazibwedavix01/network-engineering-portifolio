# Enterprise LAN — VLAN and IP Addressing Plan

| VLAN | Name | Subnet | Mask | Default gateway | Use |
|---:|---|---|---|---|---|
| 10 | ADMIN | 192.168.10.0/24 | 255.255.255.0 | 192.168.10.1 | Administration |
| 20 | FINANCE | 192.168.20.0/24 | 255.255.255.0 | 192.168.20.1 | Finance |
| 30 | IT | 192.168.30.0/24 | 255.255.255.0 | 192.168.30.1 | IT and switch management |
| 999 | PARKING | No IP subnet | — | — | Unused switch ports |

## Infrastructure addresses

| Device | Interface | Address |
|---|---|---|
| R1-HEAD-OFFICE | G0/0.10 | 192.168.10.1/24 |
| R1-HEAD-OFFICE | G0/0.20 | 192.168.20.1/24 |
| R1-HEAD-OFFICE | G0/0.30 | 192.168.30.1/24 |
| SW1-ADMIN | VLAN 30 SVI | 192.168.30.2/24 |
| SW2-DEPARTMENTS | VLAN 30 SVI | 192.168.30.3/24 |

Switch default gateway: 192.168.30.1.

## DHCP plan

The router provides DHCP for VLANs 10, 20, and 30. Addresses .1 through .20 are excluded in each subnet to reserve infrastructure/static assignments.

Observed leases during validation:
- PC-ADMIN: 192.168.10.21
- PC-FINANCE: 192.168.20.21
- PC-IT-01: 192.168.30.21
- PC-IT-02: 192.168.30.22

DNS option configured in the DHCP pools: 8.8.8.8.

## Layer 2 port map

| Switch | Port | Assignment |
|---|---|---|
| SW1 | Fa0/1 | ADMIN, VLAN 10 |
| SW1 | Fa0/2 | FINANCE, VLAN 20 |
| SW1 | Gi0/1 | Trunk to R1 G0/0 |
| SW1 | Gi0/2 | Trunk to SW2 Gi0/1 |
| SW2 | Fa0/1–Fa0/2 | IT, VLAN 30 |
| SW2 | Gi0/1 | Trunk to SW1 Gi0/2 |
| SW1 | Fa0/3–Fa0/24 | VLAN 999, shutdown |
| SW2 | Fa0/3–Fa0/24 | VLAN 999, shutdown |
| SW2 | Gi0/2 | VLAN 999, shutdown |

## Trunk parameters

- Encapsulation: IEEE 802.1Q
- Native VLAN: 1
- Allowed VLAN configuration: 1–1005 (current verified state)
- Active/forwarding VLANs observed: 1, 10, 20, 30, 999

The broad allowed VLAN list is documented as-is. Narrowing it is a future hardening task and must be performed consistently on both ends of each trunk.
