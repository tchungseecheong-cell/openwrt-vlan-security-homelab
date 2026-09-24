# NETGEAR GS308E VLAN Configuration

## Overview

The NETGEAR GS308E is used as the managed Layer 2 switch in the OpenWrt cybersecurity homelab.

The switch provides:

- IEEE 802.1Q VLAN segmentation
- Tagged trunk connectivity to OpenWrt
- Untagged access ports for endpoint devices
- Port VLAN ID (PVID) assignment
- Management access through VLAN 10

OpenWrt performs Layer 3 routing, DHCP, NAT, and firewall enforcement, while the GS308E provides Layer 2 network segmentation.

> **Note:** This document is a sanitized representation of the switch configuration. Passwords and device-specific sensitive information are intentionally excluded.

---

# Switch Management

The GS308E management interface is assigned to the Management VLAN.

```text
Management VLAN: 10
Management IP: 192.168.10.2
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
DHCP: Disabled
```

Using a static management address ensures that the switch remains available at a predictable IP address.

---

# VLAN Configuration

The following VLANs are configured on the switch:

| VLAN ID | Name | Network | Purpose |
|---|---|---|---|
| 10 | Management | 192.168.10.0/24 | Network administration |
| 20 | Servers | 192.168.20.0/24 | Server infrastructure |
| 30 | CyberLab | 192.168.30.0/24 | Cybersecurity testing |
| 40 | IoT/Test | 192.168.40.0/24 | IoT and test devices |
| 50 | Forensics | 192.168.50.0/24 | Forensics and malware analysis |

VLAN 1 remains present as the native/default VLAN on the OpenWrt uplink.

---

# Port Assignment

The switch ports are assigned as follows:

| Port | VLAN | Port Type | PVID | Purpose |
|---|---|---|---:|---|
| 1 | VLAN 1, 10, 20, 30, 40, 50 | Trunk/Hybrid | 1 | OpenWrt uplink |
| 2 | VLAN 10 | Access | 10 | Management |
| 3 | VLAN 20 | Access | 20 | Servers |
| 4 | VLAN 30 | Access | 30 | CyberLab |
| 5 | VLAN 30 | Access | 30 | CyberLab |
| 6 | VLAN 30 | Access | 30 | CyberLab |
| 7 | VLAN 40 | Access | 40 | IoT/Test |
| 8 | VLAN 50 | Access | 50 | Forensics |

---

# Port 1 — OpenWrt Trunk

Port 1 connects the GS308E to the OpenWrt router.

The port carries multiple VLANs over a single physical Ethernet connection.

```text
Port 1

VLAN 1  -> Untagged / Native
VLAN 10 -> Tagged
VLAN 20 -> Tagged
VLAN 30 -> Tagged
VLAN 40 -> Tagged
VLAN 50 -> Tagged

PVID -> 1
```

This creates an IEEE 802.1Q trunk between:

```text
OpenWrt
   |
   | VLAN 10
   | VLAN 20
   | VLAN 30
   | VLAN 40
   | VLAN 50
   |
GS308E Port 1
```

OpenWrt receives the tagged traffic and processes it using the corresponding VLAN interfaces.

---

# Port 2 — Management

Port 2 is assigned to VLAN 10.

```text
Port: 2
VLAN: 10
Membership: Untagged
PVID: 10
```

Devices connected to this port become members of:

```text
192.168.10.0/24
```

The default gateway is:

```text
192.168.10.1
```

This port is intended for trusted administrative systems.

---

# Port 3 — Servers

Port 3 is assigned to VLAN 20.

```text
Port: 3
VLAN: 20
Membership: Untagged
PVID: 20
```

Connected systems become members of:

```text
192.168.20.0/24
```

Gateway:

```text
192.168.20.1
```

This network is intended for server and infrastructure workloads.

---

# Ports 4–6 — CyberLab

Ports 4, 5, and 6 are assigned to VLAN 30.

```text
Ports: 4, 5, 6
VLAN: 30
Membership: Untagged
PVID: 30
```

Connected devices become members of:

```text
192.168.30.0/24
```

Gateway:

```text
192.168.30.1
```

These ports are intended for cybersecurity testing systems.

Examples include:

- Kali Linux
- Raspberry Pi
- Security testing systems
- Lab workstations

---

# Port 7 — IoT/Test

Port 7 is assigned to VLAN 40.

```text
Port: 7
VLAN: 40
Membership: Untagged
PVID: 40
```

Connected systems become members of:

```text
192.168.40.0/24
```

Gateway:

```text
192.168.40.1
```

This port is intended for IoT and test devices.

---

# Port 8 — Forensics

Port 8 is assigned to VLAN 50.

```text
Port: 8
VLAN: 50
Membership: Untagged
PVID: 50
```

Connected systems become members of:

```text
192.168.50.0/24
```

Gateway:

```text
192.168.50.1
```

This port is intended for forensic and malware-analysis systems.

OpenWrt prevents VLAN 50 from forwarding traffic to the Internet or other internal VLAN endpoints.

---

# VLAN Membership Matrix

The final VLAN membership can be summarized as:

| VLAN | Port 1 | Port 2 | Port 3 | Port 4 | Port 5 | Port 6 | Port 7 | Port 8 |
|---|---|---|---|---|---|---|---|---|
| VLAN 1 | Untagged | — | — | — | — | — | — | — |
| VLAN 10 | Tagged | Untagged | — | — | — | — | — | — |
| VLAN 20 | Tagged | — | Untagged | — | — | — | — | — |
| VLAN 30 | Tagged | — | — | Untagged | Untagged | Untagged | — | — |
| VLAN 40 | Tagged | — | — | — | — | — | Untagged | — |
| VLAN 50 | Tagged | — | — | — | — | — | — | Untagged |

---

# PVID Matrix

| Port | PVID |
|---|---:|
| Port 1 | 1 |
| Port 2 | 10 |
| Port 3 | 20 |
| Port 4 | 30 |
| Port 5 | 30 |
| Port 6 | 30 |
| Port 7 | 40 |
| Port 8 | 50 |

The PVID determines which VLAN is assigned to untagged frames entering a switch port.

For example:

```text
PC
 |
 | Untagged Ethernet
 |
Port 2
PVID 10
 |
 v
VLAN 10
```

The endpoint does not need to understand VLAN tagging.

---

# Tagged vs Untagged Traffic

## Tagged

Tagged traffic contains an IEEE 802.1Q VLAN identifier.

Tagged traffic is used on Port 1 because the OpenWrt trunk must carry several VLANs simultaneously.

Example:

```text
GS308E Port 1
     |
     +---- VLAN 10 TAGGED
     +---- VLAN 20 TAGGED
     +---- VLAN 30 TAGGED
     +---- VLAN 40 TAGGED
     +---- VLAN 50 TAGGED
```

## Untagged

Endpoint devices normally send and receive standard untagged Ethernet frames.

The GS308E assigns these frames to the appropriate VLAN using the port's PVID.

---

# Traffic Example

A CyberLab device connected to Port 4 follows this path:

```text
CyberLab Device
192.168.30.x
       |
       | Untagged
       v
GS308E Port 4
PVID 30
       |
       | VLAN 30
       v
GS308E Port 1
       |
       | VLAN 30 Tagged
       v
OpenWrt
eth0.30
       |
       v
Firewall / Routing
```

This demonstrates the relationship between an access port, VLAN tagging, the trunk, and the OpenWrt VLAN interface.

---

# Management Security

The GS308E management interface was moved to VLAN 10:

```text
192.168.10.2
```

This keeps switch administration associated with the trusted Management network rather than a cybersecurity-testing or IoT network.

Future hardening can further restrict which management hosts are permitted to reach the switch.

---

# Validation

The switch configuration was validated by connecting devices to different access ports and confirming that they received addresses from the expected DHCP scopes.

Examples:

```text
Port 2 -> 192.168.10.x
Port 3 -> 192.168.20.x
Ports 4-6 -> 192.168.30.x
Port 7 -> 192.168.40.x
Port 8 -> 192.168.50.x
```

Additional testing confirmed that tagged VLAN traffic successfully traversed Port 1 to OpenWrt.

---

# Future Hardening

Potential improvements include:

- Replace VLAN 1 as the native/default trunk VLAN with an unused VLAN where practical.
- Restrict switch management access to approved Management VLAN systems.
- Document switch firmware updates.
- Maintain sanitized configuration backups.
- Periodically review VLAN membership and PVID assignments.
- Disable or isolate unused switch ports when appropriate.

---

# Configuration Safety

A raw switch configuration backup is not published in this repository.

Configuration backups may contain device-specific or sensitive information.

Instead, this document records the relevant VLAN architecture and switch configuration in a sanitized, reproducible format.

---

## Related Documentation

For the complete topology, OpenWrt configuration, firewall design, screenshots, and validation results, see the main project README.

[Return to main README](https://github.com/tchungseecheong-cell/openwrt-vlan-security-homelab/blob/main/README.md)
