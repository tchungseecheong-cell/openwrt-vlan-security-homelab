# OpenWrt Firewall Configuration

## Overview

OpenWrt provides the Layer 3 firewall and routing controls for the segmented cybersecurity homelab.

Each VLAN is assigned to its own firewall zone. This allows traffic to be controlled according to the purpose and trust level of each network.

The firewall design follows a segmentation approach:

- Management systems are separated from lab systems.
- Servers are placed in a dedicated network.
- Cybersecurity testing systems are isolated from protected endpoints.
- IoT/test devices are separated from trusted systems.
- The Forensics VLAN is isolated from both the Internet and other internal VLANs.

> **Note:** This document is a sanitized representation of the firewall configuration. Credentials, keys, and other sensitive information are intentionally excluded.

---

# Firewall Zones

The following zones are used in the lab:

| Zone | VLAN | Network | Input | Output | Intra-Zone Forward | WAN Forwarding |
|---|---:|---|---|---|---|---|
| Management | 10 | 192.168.10.0/24 | ACCEPT | ACCEPT | REJECT | Allowed |
| Servers | 20 | 192.168.20.0/24 | ACCEPT | ACCEPT | REJECT | Allowed |
| CyberLab | 30 | 192.168.30.0/24 | ACCEPT | ACCEPT | REJECT | Allowed |
| IoT_Test | 40 | 192.168.40.0/24 | ACCEPT | ACCEPT | REJECT | Allowed |
| Forensics | 50 | 192.168.50.0/24 | ACCEPT | ACCEPT | REJECT | Blocked |

The WAN zone performs NAT/masquerading for networks that are permitted to access the Internet.

Masquerading is not enabled on the individual internal VLAN zones.

---

# VLAN 10 — Management

The Management zone is associated with:

```text
Interface: Management
Network: 192.168.10.0/24
Gateway: 192.168.10.1
```

Firewall behavior:

```text
Input: ACCEPT
Output: ACCEPT
Intra-Zone Forward: REJECT
Forward to WAN: ALLOWED
Masquerading: DISABLED
```

The Management VLAN is intended for trusted administrative systems and network infrastructure.

Examples include:

- OpenWrt administration
- NETGEAR GS308E management
- Administrative workstations
- Network-management systems

The GS308E management interface uses:

```text
192.168.10.2
```

---

# VLAN 20 — Servers

The Servers zone is associated with:

```text
Interface: Servers
Network: 192.168.20.0/24
Gateway: 192.168.20.1
```

Firewall behavior:

```text
Input: ACCEPT
Output: ACCEPT
Intra-Zone Forward: REJECT
Forward to WAN: ALLOWED
Masquerading: DISABLED
```

This VLAN provides a dedicated network for:

- Servers
- Virtual machines
- Infrastructure services
- Future security-monitoring services

Direct access from less-trusted VLANs is restricted unless an explicit firewall rule is created.

---

# VLAN 30 — CyberLab

The CyberLab zone is associated with:

```text
Interface: CyberLab
Network: 192.168.30.0/24
Gateway: 192.168.30.1
```

Firewall behavior:

```text
Input: ACCEPT
Output: ACCEPT
Intra-Zone Forward: REJECT
Forward to WAN: ALLOWED
Masquerading: DISABLED
```

The CyberLab VLAN is intended for:

- Kali Linux
- Raspberry Pi security projects
- Vulnerability testing
- Security tools
- Cybersecurity experimentation

CyberLab systems are permitted to reach the Internet but are restricted from directly reaching protected endpoints on other VLANs.

---

# VLAN 40 — IoT/Test

The IoT/Test zone is associated with:

```text
Interface: IoT_Test
Network: 192.168.40.0/24
Gateway: 192.168.40.1
```

Firewall behavior:

```text
Input: ACCEPT
Output: ACCEPT
Intra-Zone Forward: REJECT
Forward to WAN: ALLOWED
Masquerading: DISABLED
```

This network is intended for:

- IoT devices
- Test systems
- Less-trusted endpoints

These devices can access the Internet while remaining segmented from protected internal endpoints.

---

# VLAN 50 — Forensics

The Forensics zone is associated with:

```text
Interface: Forensics
Network: 192.168.50.0/24
Gateway: 192.168.50.1
```

Firewall behavior:

```text
Input: ACCEPT
Output: ACCEPT
Intra-Zone Forward: REJECT
Forward to WAN: BLOCKED
Masquerading: DISABLED
```

No destination forwarding zone is configured for the Forensics network.

This produces the following policy:

```text
Forensics -> Internet       BLOCKED
Forensics -> Management     BLOCKED
Forensics -> Servers        BLOCKED
Forensics -> CyberLab       BLOCKED
Forensics -> IoT/Test       BLOCKED
```

VLAN 50 is therefore suitable for controlled activities such as:

- Digital forensics
- Malware analysis
- Suspicious-file analysis
- Isolated virtual machines
- High-risk security testing

---

# Firewall Policy Matrix

The high-level forwarding policy is:

| Source | Internet | Management | Servers | CyberLab | IoT/Test |
|---|---|---|---|---|---|
| Management | Allowed | — | Restricted | Restricted | Restricted |
| Servers | Allowed | Restricted | — | Restricted | Restricted |
| CyberLab | Allowed | Blocked | Blocked | — | Blocked |
| IoT/Test | Allowed | Blocked | Blocked | Blocked | — |
| Forensics | Blocked | Blocked | Blocked | Blocked | Blocked |

> "Restricted" indicates that access is not intended to be broadly permitted and should be controlled using explicit firewall rules when a legitimate service requires communication.

---

# Input vs Forwarding

One of the most important concepts demonstrated during this project was the difference between **Input** and **Forwarding** in OpenWrt.

## Input Traffic

Input traffic is traffic whose destination is the OpenWrt router itself.

Examples include:

```text
192.168.10.1
192.168.20.1
192.168.30.1
192.168.40.1
192.168.50.1
```

These are OpenWrt interface addresses.

Because the lab firewall zones currently use:

```text
Input: ACCEPT
```

a client may be able to ping another VLAN's OpenWrt gateway address.

This does **not** automatically mean the client can communicate with endpoints located on that VLAN.

---

## Forwarded Traffic

Forwarded traffic passes through OpenWrt from one network to another.

Example:

```text
CyberLab Client
192.168.30.x
       |
       v
    OpenWrt
       |
       X
       |
Server
192.168.20.x
```

The firewall forwarding policy determines whether this traffic is allowed.

Testing against actual endpoints therefore provides better evidence of network isolation than testing only OpenWrt gateway addresses.

---

# Isolation Validation

Firewall behavior was validated using actual devices.

## CyberLab to Servers

A CyberLab system attempted to reach an actual VLAN 20 endpoint:

```text
CyberLab
192.168.30.x

        X

Server
192.168.20.182
```

Result:

```text
BLOCKED
```

This confirmed that traffic was not being forwarded from CyberLab to the Servers VLAN.

---

## IoT/Test Isolation

A Raspberry Pi was connected to VLAN 40.

The device successfully reached:

```text
192.168.40.1
8.8.8.8
google.com
```

while access to protected internal endpoints was blocked.

This confirmed that VLAN 40 had Internet connectivity without unrestricted access to trusted networks.

---

## Forensics Internet Isolation

A forensic Windows workstation on VLAN 50 attempted to reach:

```text
8.8.8.8
```

OpenWrt returned:

```text
Destination port unreachable
```

This confirmed that VLAN 50 did not have WAN forwarding permission.

---

## Forensics Inter-VLAN Isolation

The forensic workstation was also tested against an actual Raspberry Pi located on VLAN 40:

```text
Forensic Workstation
192.168.50.182

        X

Raspberry Pi
192.168.40.140
```

Result:

```text
BLOCKED
```

This demonstrated that the forensic network could not forward traffic into another internal VLAN.

---

# Understanding ICMP Rejection Messages

During blocked connectivity tests, OpenWrt may return an ICMP error such as:

```text
Destination port unreachable
```

On Windows, this response can sometimes appear in ping statistics as a received packet.

The packet was received from the router as an **error response**.

It does not indicate that the intended destination responded successfully.

---

# NAT and Masquerading

NAT is performed by the OpenWrt WAN firewall zone.

The internal VLAN zones do not require masquerading.

The traffic flow for an Internet-enabled VLAN is:

```text
VLAN Client
     |
     v
OpenWrt VLAN Interface
     |
     v
Firewall
     |
     v
WAN Zone / NAT
     |
     v
Rogers XB7
     |
     v
Internet
```

VLAN 50 does not have forwarding permission to the WAN zone and therefore cannot follow this path.

---

# Security Design

The firewall architecture follows the principle of least privilege by limiting unnecessary communication between network segments.

The design separates systems based on function and risk:

```text
Management
   |
   +-- Trusted administration

Servers
   |
   +-- Infrastructure services

CyberLab
   |
   +-- Security testing

IoT/Test
   |
   +-- Less-trusted devices

Forensics
   |
   +-- High-risk isolated analysis
```

Segmentation helps reduce the potential impact of a compromised or untrusted endpoint.

---

# Future Firewall Hardening

The current configuration was designed as a functional learning environment.

Potential future hardening includes:

- Change Input from ACCEPT to REJECT on untrusted VLANs.
- Add explicit DHCP and DNS rules where required.
- Restrict OpenWrt administration to the Management VLAN.
- Add explicit allow rules for required cross-VLAN services.
- Enable centralized firewall logging.
- Forward security events to a SIEM.
- Add IDS/IPS monitoring.
- Log and analyze rejected inter-VLAN traffic.
- Review IPv6 firewall behavior before enabling IPv6 throughout the lab.

These changes can be implemented incrementally while preserving management access and lab functionality.

---

# Configuration Safety

Raw OpenWrt firewall exports are intentionally not published in this repository.

Configuration exports may contain information that should not be exposed publicly.

Examples include:

- Passwords
- Authentication keys
- Private keys
- VPN credentials
- Wireless credentials
- API tokens
- Device-specific identifiers

This document provides a sanitized representation of the firewall architecture instead.

---

## Related Documentation

The complete project architecture, topology, screenshots, testing evidence, and implementation details are available in the main project README.

[Return to main README](https://github.com/tchungseecheong-cell/openwrt-vlan-security-homelab/blob/main/README.md)
