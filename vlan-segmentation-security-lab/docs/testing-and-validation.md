# Testing and Validation

## 1. Purpose

This document records the checks performed during the lab build. Results below reflect the tests and observations recorded in the project braindump and subsequent project work. It is not a substitute for re-running the tests after making configuration changes.

## 2. Validation Results

| Area | Test or evidence | Expected result | Recorded result |
|---|---|---|---|
| Physical/link status | Check router and switch interfaces after cabling | Router uplink is operational | Initially the router interface was administratively down; enabling `GigabitEthernet0/0` with `no shutdown` brought it up |
| VLAN assignment | Inspect switch VLANs and port assignments | Staff ports in VLAN 10; Guest port in VLAN 20 | VLAN and access-port assignments were configured and checked |
| Trunk | Inspect SW1 `GigabitEthernet0/1` | Link carries VLAN-tagged traffic for VLANs 10 and 20 | Trunk operation and 802.1Q encapsulation were verified |
| Inter-VLAN routing | Inspect R1 subinterfaces | VLAN 10 and VLAN 20 gateways are configured | `G0/0.10` and `G0/0.20` were configured with the planned gateway addresses |
| DHCP | Check client addresses and router DHCP bindings | Clients receive addresses from their assigned VLAN subnet | PC1 received `192.168.10.2`; PC3 received `192.168.10.3`; PC2 received `192.168.20.2` |
| Staff to Guest ICMP | Ping PC2 (`192.168.20.2`) from PC1 | Ping succeeds | Passed after the ACL was updated to permit the required ICMP echo replies |
| Guest to Staff ICMP | Ping PC1 (`192.168.10.2`) from PC2 | Guest-initiated traffic is blocked | Blocked as intended by the ACL |
| DNS from Staff | Resolve `server.staff.local` from a Staff PC | Name resolves to `192.168.10.10` | DNS resolution was tested successfully |
| DNS from Guest | Run `nslookup server.staff.local` from the Guest PC | DNS query resolves despite the Guest-to-Staff restriction | Resolution succeeded after permitting UDP and TCP DNS traffic to `192.168.10.10` |
| SSH to SW1 | Connect from PC1 using SSH | Local credentials authenticate and CLI access is granted | Successful SSH login was recorded |
| Port security | Connect a different PC to SW1 `Fa0/1` | A violation occurs under the configured shutdown policy | Violation was observed; the original PC was reconnected and the port restored |
| Port-security recovery | Check `show port-security interface fa0/1` after restoration | Port is secure and the original sticky MAC is present | Reported as `Secure-up`, with one sticky MAC address and zero violations after restoration |

## 3. Useful Verification Commands

Run these commands on the relevant device when repeating the validation.

### On SW1

```text
show vlan brief
show interfaces trunk
show ip interface brief
show port-security interface fa0/1
show port-security address
show running-config
```

### On R1

```text
show ip interface brief
show ip route
show ip dhcp binding
show running-config
show access-lists
```

### From client PCs

- Check the assigned IP address, subnet mask, gateway, and DNS server in the PC's IP configuration.
- Ping the intended destination IP for connectivity tests.
- Use `nslookup server.staff.local` to test DNS name resolution.
- Use the Packet Tracer PC command prompt to test SSH access to SW1.

## 4. Interpreting the ACL Tests

A successful Staff-to-Guest ping requires the Guest echo reply to pass through the inbound ACL on the Guest router subinterface. The ACL therefore includes a permit for ICMP echo-reply from the Guest subnet to the Staff subnet before the broader Guest-to-Staff deny.

A Guest-to-Staff ping is blocked because its initial ICMP echo request matches the Guest-to-Staff deny rule. The explicit DNS permits allow Guest clients to query the DNS server at `192.168.10.10` on port 53, but do not grant unrestricted access to the Staff subnet.
