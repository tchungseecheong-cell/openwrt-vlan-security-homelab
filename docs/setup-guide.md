# OpenWrt VLAN Security Homelab — Setup Guide

## Overview

This guide documents the high-level process used to build the OpenWrt VLAN Security Homelab.

The environment uses an **ASUS RT-N56U running OpenWrt** as the Layer 3 router/firewall and a **NETGEAR GS308E** as the managed Layer 2 switch.

The finished environment contains five segmented networks:

| VLAN | Name | Network | Gateway | Purpose |
|---|---|---|---|---|
| 10 | Management | `192.168.10.0/24` | `192.168.10.1` | Network administration |
| 20 | Servers | `192.168.20.0/24` | `192.168.20.1` | Server infrastructure |
| 30 | CyberLab | `192.168.30.0/24` | `192.168.30.1` | Cybersecurity testing |
| 40 | IoT/Test | `192.168.40.0/24` | `192.168.40.1` | IoT and test devices |
| 50 | Forensics | `192.168.50.0/24` | `192.168.50.1` | Forensics/malware analysis |

> **Important:** This guide intentionally excludes passwords, authentication keys, wireless credentials, and other sensitive information.

---

# 1. Final Architecture

The completed physical topology is:

```text
Internet
   |
   v
Rogers XB7 Gateway
10.0.0.1/24
   |
   | WAN
   v
ASUS RT-N56U
OpenWrt Router / Firewall
   |
   | IEEE 802.1Q Trunk
   | VLAN 10, 20, 30, 40, 50
   v
NETGEAR GS308E
Managed Switch
   |
   +-- Port 2 ------ VLAN 10 ------ Management
   |
   +-- Port 3 ------ VLAN 20 ------ Servers
   |
   +-- Ports 4-6 --- VLAN 30 ------ CyberLab
   |
   +-- Port 7 ------ VLAN 40 ------ IoT/Test
   |
   +-- Port 8 ------ VLAN 50 ------ Forensics
```

The graphical topology is available in the main repository:

[View Main Project README](https://github.com/tchungseecheong-cell/openwrt-vlan-security-homelab/blob/main/README.md)

---

# 2. Prerequisites

The lab requires:

- OpenWrt-compatible router
- Managed switch supporting IEEE 802.1Q VLANs
- Ethernet cables
- Administrative workstation
- Internet connection
- Test endpoints

The implementation documented in this project uses:

```text
Router: ASUS RT-N56U
Firmware: OpenWrt 25.12.1
Switch: NETGEAR GS308E
Upstream Gateway: Rogers XB7
```

---

# 3. Configure the OpenWrt WAN

Connect the OpenWrt WAN interface to the upstream Internet gateway.

The upstream network used in this lab is:

```text
Network: 10.0.0.0/24
Gateway: 10.0.0.1
WAN Protocol: DHCP Client
```

The OpenWrt WAN interface receives its address dynamically from the upstream gateway.

Verify that OpenWrt has Internet connectivity before proceeding with VLAN configuration.

---

# 4. Create the VLANs in OpenWrt

Configure the following VLAN IDs:

```text
VLAN 10
VLAN 20
VLAN 30
VLAN 40
VLAN 50
```

The OpenWrt CPU port must carry these VLANs as tagged traffic.

The LAN port connected to the GS308E must also carry VLANs 10–50 as tagged traffic.

Conceptually:

```text
               CPU        GS308E Uplink

VLAN 10       Tagged         Tagged
VLAN 20       Tagged         Tagged
VLAN 30       Tagged         Tagged
VLAN 40       Tagged         Tagged
VLAN 50       Tagged         Tagged
```

This creates the IEEE 802.1Q trunk between OpenWrt and the managed switch.

> OpenWrt interface names and physical switch-port numbering can vary by router model. Verify the correct physical uplink before modifying VLAN membership.

---

# 5. Create the Management Interface

Create a new OpenWrt interface for VLAN 10.

Configure:

```text
Name: Management
Device: eth0.10
Protocol: Static address
IPv4 Address: 192.168.10.1
Netmask: 255.255.255.0
```

Do not configure an additional gateway on the VLAN interface.

OpenWrt itself acts as the gateway for VLAN 10.

---

# 6. Create the Servers Interface

Create the VLAN 20 interface:

```text
Name: Servers
Device: eth0.20
Protocol: Static address
IPv4 Address: 192.168.20.1
Netmask: 255.255.255.0
```

This network is used for server and infrastructure systems.

---

# 7. Create the CyberLab Interface

Create the VLAN 30 interface:

```text
Name: CyberLab
Device: eth0.30
Protocol: Static address
IPv4 Address: 192.168.30.1
Netmask: 255.255.255.0
```

This network is used for cybersecurity testing systems.

Examples include:

- Kali Linux
- Raspberry Pi
- Vulnerability-testing systems
- Security tools
- Lab workstations

---

# 8. Create the IoT/Test Interface

Create the VLAN 40 interface:

```text
Name: IoT_Test
Device: eth0.40
Protocol: Static address
IPv4 Address: 192.168.40.1
Netmask: 255.255.255.0
```

This network is intended for less-trusted IoT and test systems.

---

# 9. Create the Forensics Interface

Create the VLAN 50 interface:

```text
Name: Forensics
Device: eth0.50
Protocol: Static address
IPv4 Address: 192.168.50.1
Netmask: 255.255.255.0
```

This VLAN is designed for:

- Digital forensics
- Malware analysis
- Suspicious-file analysis
- Isolated security testing

Unlike the other lab VLANs, VLAN 50 will not receive WAN forwarding permission.

---

# 10. Configure DHCP

Enable a DHCP server on each VLAN interface.

The lab uses:

```text
Start: 100
Limit: 100
Lease Time: 12h
```

This provides dynamic addresses beginning around:

```text
VLAN 10 -> 192.168.10.100
VLAN 20 -> 192.168.20.100
VLAN 30 -> 192.168.30.100
VLAN 40 -> 192.168.40.100
VLAN 50 -> 192.168.50.100
```

OpenWrt provides DHCP and DNS services to clients on each VLAN.

---

# 11. Create OpenWrt Firewall Zones

Create a separate firewall zone for every VLAN.

The initial lab configuration uses:

```text
Input: ACCEPT
Output: ACCEPT
Intra-Zone Forward: REJECT
Masquerading: Disabled
```

Create zones for:

```text
Management
Servers
CyberLab
IoT_Test
Forensics
```

Assign the corresponding network interface to each firewall zone.

---

# 12. Configure WAN Forwarding

Allow the following zones to forward traffic to the WAN:

```text
Management -> WAN
Servers    -> WAN
CyberLab   -> WAN
IoT_Test   -> WAN
```

Do not create WAN forwarding for:

```text
Forensics
```

The intended result is:

```text
VLAN 10 -> Internet     ALLOWED
VLAN 20 -> Internet     ALLOWED
VLAN 30 -> Internet     ALLOWED
VLAN 40 -> Internet     ALLOWED
VLAN 50 -> Internet     BLOCKED
```

NAT/masquerading is handled by the OpenWrt WAN zone.

---

# 13. Configure the GS308E Management Address

Configure the NETGEAR GS308E with a static management address:

```text
Management VLAN: 10
IP Address: 192.168.10.2
Subnet Mask: 255.255.255.0
Gateway: 192.168.10.1
DHCP: Disabled
```

This places switch administration on the dedicated Management network.

---

# 14. Create VLANs on the GS308E

Create the following IEEE 802.1Q VLANs:

```text
10 - Management
20 - Servers
30 - CyberLab
40 - IoT/Test
50 - Forensics
```

---

# 15. Configure the OpenWrt Trunk Port

Port 1 of the GS308E connects to OpenWrt.

Configure Port 1 as:

```text
VLAN 1  -> Untagged / Native
VLAN 10 -> Tagged
VLAN 20 -> Tagged
VLAN 30 -> Tagged
VLAN 40 -> Tagged
VLAN 50 -> Tagged

PVID: 1
```

This allows all five lab VLANs to traverse the OpenWrt-to-GS308E connection.

---

# 16. Configure Access Ports

Configure the endpoint-facing switch ports as follows:

| Port | VLAN | Membership | PVID |
|---|---:|---|---:|
| 2 | 10 | Untagged | 10 |
| 3 | 20 | Untagged | 20 |
| 4 | 30 | Untagged | 30 |
| 5 | 30 | Untagged | 30 |
| 6 | 30 | Untagged | 30 |
| 7 | 40 | Untagged | 40 |
| 8 | 50 | Untagged | 50 |

Remove access ports from VLAN 1 where appropriate so they belong only to their intended VLAN.

---

# 17. Verify DHCP Assignment

Connect a test system to each access VLAN and verify that it receives an address from the correct subnet.

Expected results:

```text
Port 2 -> 192.168.10.x
Port 3 -> 192.168.20.x
Port 4 -> 192.168.30.x
Port 5 -> 192.168.30.x
Port 6 -> 192.168.30.x
Port 7 -> 192.168.40.x
Port 8 -> 192.168.50.x
```

On Windows, verify using:

```powershell
ipconfig /all
```

On Linux, examples include:

```bash
ip addr
```

or:

```bash
ip route
```

---

# 18. Test the Local Gateway

From each VLAN, verify that the corresponding OpenWrt interface is reachable.

Examples:

```text
VLAN 10 -> ping 192.168.10.1
VLAN 20 -> ping 192.168.20.1
VLAN 30 -> ping 192.168.30.1
VLAN 40 -> ping 192.168.40.1
VLAN 50 -> ping 192.168.50.1
```

Successful gateway connectivity verifies several components at once:

- Physical connectivity
- Access-port assignment
- PVID configuration
- VLAN trunking
- OpenWrt VLAN interface
- Basic Layer 3 connectivity

---

# 19. Test Internet Connectivity

For VLANs permitted to use the WAN, test:

```text
ping 8.8.8.8
```

Then test DNS:

```text
ping google.com
```

Expected behavior:

| VLAN | Internet |
|---|---|
| Management | Allowed |
| Servers | Allowed |
| CyberLab | Allowed |
| IoT/Test | Allowed |
| Forensics | Blocked |

---

# 20. Test Inter-VLAN Isolation

Do not rely only on testing another VLAN's `.1` gateway address.

The `.1` addresses belong to OpenWrt itself, so they test traffic to the router rather than traffic being forwarded through the router.

Instead, test against an actual endpoint.

Example:

```text
CyberLab Client
192.168.30.x
       |
       X
       |
Server
192.168.20.182
```

Expected result:

```text
BLOCKED
```

This provides stronger evidence that forwarding between the VLANs is restricted.

---

# 21. Validate Forensics Isolation

Connect the forensic workstation to Port 8.

Verify DHCP:

```text
IP Address: 192.168.50.x
Gateway: 192.168.50.1
DNS: 192.168.50.1
```

Verify the local gateway:

```text
ping 192.168.50.1
```

Expected:

```text
SUCCESS
```

Test Internet access:

```text
ping 8.8.8.8
```

Expected:

```text
BLOCKED
```

Test an actual device on another VLAN.

Example:

```text
ping 192.168.40.140
```

Expected:

```text
BLOCKED
```

This confirms that VLAN 50 can communicate with its local OpenWrt interface while remaining isolated from the Internet and other VLAN endpoints.

---

# 22. Understanding Input vs Forwarding

OpenWrt treats traffic to the router differently from traffic routed between networks.

## Input

Traffic addressed directly to OpenWrt:

```text
Client
   |
   v
192.168.X.1
OpenWrt
```

This traffic is controlled by the firewall zone's **Input** policy.

## Forwarding

Traffic traveling through OpenWrt:

```text
VLAN 30 Client
      |
      v
   OpenWrt
      |
      X
      |
VLAN 20 Server
```

This traffic is controlled by firewall **Forwarding** policy.

Therefore, successfully pinging another VLAN's OpenWrt gateway does not prove that endpoints on that VLAN are reachable.

---

# 23. Troubleshooting Checklist

If a VLAN client does not receive an IP address, verify:

```text
1. Correct physical switch port
2. Correct VLAN membership
3. Correct PVID
4. Access port configured as untagged
5. Trunk VLAN configured as tagged
6. OpenWrt VLAN exists
7. OpenWrt interface exists
8. DHCP server is enabled
```

If DHCP works but Internet access does not, verify:

```text
1. Client default gateway
2. OpenWrt firewall-zone assignment
3. Forwarding from VLAN zone to WAN
4. WAN connectivity
5. DNS configuration
```

If inter-VLAN isolation appears not to work:

```text
1. Verify the destination is an actual endpoint.
2. Do not use only the OpenWrt .1 interface for testing.
3. Check firewall forwarding configuration.
4. Verify the endpoint's IP address and VLAN.
```

---

# 24. Recommended Configuration Order

A safe deployment sequence is:

```text
1. Verify OpenWrt WAN connectivity
        |
2. Create VLAN
        |
3. Create OpenWrt interface
        |
4. Configure DHCP
        |
5. Create firewall zone
        |
6. Configure trunk membership
        |
7. Configure switch access port
        |
8. Configure PVID
        |
9. Connect test endpoint
        |
10. Verify DHCP
        |
11. Verify gateway
        |
12. Verify Internet policy
        |
13. Verify inter-VLAN policy
```

Implementing and testing one VLAN at a time reduces the risk of accidentally losing management access.

---

# 25. Security Hardening Opportunities

The lab currently prioritizes functionality, segmentation, and hands-on learning.

Future hardening can include:

- Restrict router management to VLAN 10.
- Change Input to REJECT on less-trusted VLANs.
- Add explicit DHCP and DNS rules.
- Allow only required cross-VLAN services.
- Replace the native/default VLAN 1 on the trunk with an unused VLAN where practical.
- Disable or isolate unused switch ports.
- Enable centralized logging.
- Forward firewall/security events to a SIEM.
- Deploy IDS/IPS monitoring.
- Review IPv6 behavior before enabling it across all VLANs.
- Maintain sanitized configuration backups.

---

# 26. Configuration Documentation

Detailed sanitized configuration information is available in:

- [`openwrt-network.md`](../configs/openwrt-network.md)
- [`openwrt-firewall.md`](../configs/openwrt-firewall.md)
- [`gs308e-vlan-config.md`](../configs/gs308e-vlan-config.md)

These documents provide additional information about the OpenWrt interfaces, firewall zones, switch VLAN membership, and PVID configuration.

---

# 27. Final Validation

Before considering the deployment complete, verify:

- [ ] OpenWrt has Internet connectivity
- [ ] VLANs 10, 20, 30, 40, and 50 exist
- [ ] All VLAN interfaces use the correct gateway addresses
- [ ] DHCP operates on each VLAN
- [ ] Port 1 carries all required tagged VLANs
- [ ] Access ports use the correct VLAN membership
- [ ] PVIDs match access VLANs
- [ ] VLAN 10 can access required management resources
- [ ] VLAN 20 has expected server connectivity
- [ ] VLAN 30 has Internet access
- [ ] VLAN 30 cannot reach protected endpoints
- [ ] VLAN 40 has Internet access
- [ ] VLAN 40 cannot reach protected endpoints
- [ ] VLAN 50 can reach its local gateway
- [ ] VLAN 50 cannot access the Internet
- [ ] VLAN 50 cannot access endpoints on other VLANs

---

# Conclusion

This deployment creates a segmented networking and cybersecurity environment using OpenWrt and a managed Layer 2 switch.

The implementation demonstrates the relationship between:

```text
VLANs
  +
802.1Q Trunking
  +
Access Ports / PVIDs
  +
Layer 3 Interfaces
  +
DHCP
  +
Firewall Zones
  +
Routing Policies
  =
Segmented Cybersecurity Homelab
```

The resulting environment provides a reusable platform for networking, cybersecurity testing, monitoring, digital forensics, virtualization, and future security projects.

---

## Related Documentation

For the complete project overview, topology diagram, screenshots, and validation evidence:

[Return to main README](https://github.com/tchungseecheong-cell/openwrt-vlan-security-homelab/blob/main/README.md)
