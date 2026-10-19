# Security Controls

## 1. Purpose

This document describes the security controls implemented in the Packet Tracer lab and the protection each control is intended to provide.

## 2. VLAN Segmentation

The switch separates endpoint traffic into two VLANs:

- **VLAN 10 - STAFF:** Staff PCs, the switch management SVI, and the DNS server.
- **VLAN 20 - GUEST:** Guest PC.

VLANs create separate Layer 2 broadcast domains. Communication between the two subnets must pass through R1, where the ACL can filter routed traffic. VLAN separation by itself is not a complete security policy; the router ACL provides the inter-VLAN filtering in this lab.

## 3. Guest Traffic ACL

The extended ACL `GUEST-FILTER` is applied inbound on router subinterface `GigabitEthernet0/0.20`, the gateway interface for VLAN 20.

The configured rules are evaluated from top to bottom:

| Rule order | Traffic match | Action | Purpose |
|---:|---|---|---|
| 1 | ICMP echo-reply from `192.168.20.0/24` to `192.168.10.0/24` | Permit | Allows replies to Staff-initiated pings to return from Guest |
| 2 | UDP from Guest subnet to `192.168.10.10`, destination port 53 | Permit | Allows Guest DNS queries over UDP |
| 3 | TCP from Guest subnet to `192.168.10.10`, destination port 53 | Permit | Allows Guest DNS traffic over TCP |
| 4 | IP from Guest subnet to Staff subnet | Deny | Blocks other Guest-initiated traffic to Staff |
| 5 | Any IP traffic not matched earlier | Permit | Allows other traffic not matched by the previous rules |

The final `permit ip any any` does not override the earlier Guest-to-Staff deny because ACL processing stops at the first matching rule.

### Important behavior

The ACL is stateless. The ICMP echo-reply rule allows a specific type of response, not all return traffic from sessions initiated by Staff. The DNS rules permit traffic to the DNS server on port 53; they do not provide Guest clients with general access to Staff devices.

## 4. Secure Device Management

SSH was configured for remote administration of SW1, and SSH access from PC1 in VLAN 10 was tested successfully.

The switch configuration includes:
- A local user account for authentication
- SSH version 2
- A domain name used for SSH key generation
- VTY lines configured for local login and SSH transport only
- An enable secret for privileged EXEC access
- Console login protection

The switch management SVI is `Vlan10` with address `192.168.10.4/24`. The switch default gateway is `192.168.10.1`.

## 5. Switch Port Security

Port security is configured on SW1 `FastEthernet0/1`, the access port connected to PC1 in VLAN 10.

The intended configuration:
- Enables port security on the access port
- Uses sticky MAC learning to learn and retain the connected PC's MAC address
- Allows a maximum of one secure MAC address
- Uses the `shutdown` violation mode

A test was performed by disconnecting PC1 and connecting PC2 to that port. The switch recorded a port-security violation and the port was subsequently restored by reconnecting the original PC to its intended port.

Port security is a Layer 2 access-port control. It should not be treated as strong user authentication or as a replacement for other network access controls.
