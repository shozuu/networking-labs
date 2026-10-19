# Network Design

## 1. Scope

This document records the design, traffic flow, and the network structure of the VLAN Segmentation and Network Security Lab built in Cisco Packet Tracer.

The lab contains:
- One Cisco 1941 router (`R1` in the project documentation)
- One Cisco 2960 switch (`SW1`)
- Three client PCs
- One DNS server

The DNS server is placed in VLAN 10 for this lab. A dedicated server VLAN was considered but was not implemented.

## 2. Topology and Links

| Device/interface | Connected device/interface | Purpose |
|---|---|---|
| Router GigabitEthernet0/0 | SW1 GigabitEthernet0/1 | 802.1Q trunk carrying VLAN traffic |
| SW1 FastEthernet0/1 | PC1 (Staff) | Staff access port, VLAN 10 |
| SW1 FastEthernet0/2 | PC2 (Guest) | Guest access port, VLAN 20 |
| SW1 FastEthernet0/3 | PC3 (Staff) | Staff access port, VLAN 10 |
| SW1 FastEthernet0/4 | DNS server | Server access port, VLAN 10 |

The router's physical `GigabitEthernet0/0` interface carries VLAN-tagged traffic. Its VLAN subinterfaces provide the Layer 3 gateways for the two client networks.

## 3. VLAN and IP Addressing Plan

| VLAN | Name | Subnet | Default gateway | Purpose |
|---:|---|---|---|---|
| 10 | STAFF | `192.168.10.0/24` | `192.168.10.1` | Staff PCs, switch management SVI, and DNS server |
| 20 | GUEST | `192.168.20.0/24` | `192.168.20.1` | Guest client network |

The subnet mask for both networks is `255.255.255.0`.

### Known addresses

| Device or role | Address | Assignment |
|---|---|---|
| R1 VLAN 10 gateway (`G0/0.10`) | `192.168.10.1/24` | Static |
| R1 VLAN 20 gateway (`G0/0.20`) | `192.168.20.1/24` | Static |
| SW1 management SVI (`Vlan10`) | `192.168.10.4/24` | Static |
| DNS server | `192.168.10.10/24` | Static |
| PC1 (Staff) | `192.168.10.2/24` | DHCP lease observed |
| PC3 (Staff) | `192.168.10.3/24` | DHCP lease observed |
| PC2 (Guest) | `192.168.20.2/24` | DHCP lease observed |

The switch uses `192.168.10.1` as its default gateway for management traffic. The DNS server uses `192.168.10.1` as its default gateway.

The router excludes `192.168.10.1`, `192.168.10.4`, `192.168.10.10`, and `192.168.20.1` from DHCP assignment so these configured infrastructure addresses are not leased to clients.

## 4. Switching and Inter-VLAN Routing

Client-facing switch ports are configured as access ports in their respective VLANs. The link between SW1 and R1 is configured as an 802.1Q trunk so it can carry traffic for VLANs 10 and 20.

R1 uses router-on-a-stick:
- `GigabitEthernet0/0.10` is associated with VLAN 10 and has address `192.168.10.1/24`.
- `GigabitEthernet0/0.20` is associated with VLAN 20 and has address `192.168.20.1/24`.

Traffic between devices in the same VLAN can be switched locally. Traffic between VLANs is sent to the router gateway, where routing and the configured ACL determine whether it can proceed.

## 5. DHCP

R1 provides two DHCP pools:
- `STAFF` serves `192.168.10.0/24`, advertises gateway `192.168.10.1`, and advertises DNS server `192.168.10.10`.
- `GUEST` serves `192.168.20.0/24`, advertises gateway `192.168.20.1`, and advertises DNS server `192.168.10.10`.

The switch management address and DNS server address are manually assigned and excluded from the DHCP pool. This keeps those service addresses predictable and prevents the router from leasing them to clients.

## 6. DNS

The DNS server is configured with the static address `192.168.10.10/24` and a DNS record:

| Name | Address |
|---|---|
| `server.staff.local` | `192.168.10.10` |

Both DHCP pools advertise `192.168.10.10` as the DNS server. The Guest ACL permits DNS traffic to that host on UDP and TCP port 53 while continuing to block other Guest-to-Staff IP traffic under the configured deny rule.