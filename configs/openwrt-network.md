# OpenWrt Network Configuration

## Overview

OpenWrt is used as the Layer 3 router, DHCP server, and firewall for the homelab.

The router provides separate interfaces for each VLAN and connects to the NETGEAR GS308E using an IEEE 802.1Q trunk.

> **Note:** This document contains a sanitized representation of the lab configuration. Passwords, authentication information, and other sensitive values are intentionally excluded.

---

## VLAN Configuration

| VLAN ID | Interface | Device | IPv4 Network | Gateway |
|---|---|---|---|---|
| 10 | Management | `eth0.10` | `192.168.10.0/24` | `192.168.10.1` |
| 20 | Servers | `eth0.20` | `192.168.20.0/24` | `192.168.20.1` |
| 30 | CyberLab | `eth0.30` | `192.168.30.0/24` | `192.168.30.1` |
| 40 | IoT_Test | `eth0.40` | `192.168.40.0/24` | `192.168.40.1` |
| 50 | Forensics | `eth0.50` | `192.168.50.0/24` | `192.168.50.1` |

---

## OpenWrt Switch VLAN Configuration

The OpenWrt switch carries VLAN traffic toward the NETGEAR GS308E.

The CPU port is tagged so OpenWrt can process traffic belonging to each VLAN.

The downstream LAN port connected to the GS308E is also tagged for VLANs 10 through 50.

| VLAN | CPU (`eth0`) | GS308E Uplink |
|---|---|---|
| 10 | Tagged | Tagged |
| 20 | Tagged | Tagged |
| 30 | Tagged | Tagged |
| 40 | Tagged | Tagged |
| 50 | Tagged | Tagged |

This creates the 802.1Q trunk between OpenWrt and the managed switch.

---

## Management Interface

```text
Interface: Management
Device: eth0.10
Protocol: Static address
IPv4 Address: 192.168.10.1
Netmask: 255.255.255.0
```

Purpose:

- Network administration
- OpenWrt management
- Managed-switch administration
- Trusted management systems

---

## Servers Interface

```text
Interface: Servers
Device: eth0.20
Protocol: Static address
IPv4 Address: 192.168.20.1
Netmask: 255.255.255.0
```

Purpose:

- Servers
- Virtual machines
- Infrastructure services

---

## CyberLab Interface

```text
Interface: CyberLab
Device: eth0.30
Protocol: Static address
IPv4 Address: 192.168.30.1
Netmask: 255.255.255.0
```

Purpose:

- Kali Linux
- Raspberry Pi security projects
- Vulnerability testing
- Security monitoring
- Cybersecurity experimentation

---

## IoT/Test Interface

```text
Interface: IoT_Test
Device: eth0.40
Protocol: Static address
IPv4 Address: 192.168.40.1
Netmask: 255.255.255.0
```

Purpose:

- IoT devices
- Test systems
- Less-trusted endpoints

---

## Forensics Interface

```text
Interface: Forensics
Device: eth0.50
Protocol: Static address
IPv4 Address: 192.168.50.1
Netmask: 255.255.255.0
```

Purpose:

- Digital forensics
- Malware analysis
- Suspicious file analysis
- Isolated security testing

VLAN 50 is intentionally configured as the most restricted network in the environment.

---

# DHCP Configuration

OpenWrt provides an independent DHCP scope for each VLAN.

The lab uses a DHCP starting offset of `100` with up to `100` addresses and a 12-hour lease.

Example:

```text
Start: 100
Limit: 100
Lease Time: 12h
```

This results in dynamic client addressing beginning around:

```text
192.168.X.100
```

for each VLAN subnet.

---

# WAN Configuration

The OpenWrt WAN interface connects upstream to the Rogers XB7 gateway.

```text
Upstream Network: 10.0.0.0/24
Upstream Gateway: 10.0.0.1
WAN Protocol: DHCP Client
```

The WAN address is dynamically assigned by the upstream gateway and therefore is not documented as a permanent address.

---

# Traffic Flow

The logical network path is:

```text
Internet
   |
Rogers XB7
   |
OpenWrt WAN
   |
OpenWrt Routing / Firewall
   |
802.1Q VLAN Trunk
   |
NETGEAR GS308E
   |
VLAN Access Ports
```

OpenWrt performs Layer 3 routing and firewall enforcement while the GS308E provides Layer 2 VLAN segmentation.

---

# Security Considerations

Sensitive configuration information has intentionally been excluded from this repository.

The following should never be published in a public configuration export:

- Administrative passwords
- Authentication keys
- Private keys
- VPN credentials
- API tokens
- Wireless passwords
- Public-facing credentials
- Other secrets

The configuration documented here therefore represents the networking architecture without exposing authentication material.

---

## Related Documentation

See the main project README for the complete network architecture, firewall design, connectivity testing, screenshots, and topology diagram.
[Return to main README](https://github.com/tchungseecheong-cell/openwrt-vlan-security-homelab/blob/main/README.md)
