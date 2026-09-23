# OpenWrt VLAN Security Homelab

## Overview

This project documents the design, implementation, and validation of a segmented networking and cybersecurity homelab using OpenWrt and a NETGEAR managed switch.

The lab was built to provide hands-on experience with VLANs, IEEE 802.1Q tagging, trunk and access ports, DHCP, routing, firewall policies, network segmentation, virtualization, cybersecurity testing, and forensic network isolation.

The environment separates management systems, servers, cybersecurity testing systems, IoT/test devices, and forensic/malware-analysis systems into dedicated VLANs.

---

## Lab Objectives

The primary objectives of this project were to:

- Configure OpenWrt as the lab router and firewall
- Configure a managed switch for VLAN segmentation
- Implement IEEE 802.1Q VLAN tagging
- Configure trunk and access ports
- Configure dedicated DHCP scopes for each VLAN
- Implement firewall zones and inter-VLAN isolation
- Provide Internet access only where required
- Create dedicated server and cybersecurity networks
- Build an isolated network for forensic and malware analysis
- Validate segmentation through connectivity testing
- Integrate physical and virtualized lab systems
- Document the implementation as a cybersecurity portfolio project

---

## Core Network Hardware

| Device | Role |
|---|---|
| ASUS RT-N56U | OpenWrt Router / Firewall |
| NETGEAR GS308E | Managed VLAN Switch |
| Rogers XB7 Gateway | Upstream Internet Gateway |
| Raspberry Pi 4 (4 GB) | Lab / Test Endpoint |
| Windows Laptop | Forensics / Malware Analysis Endpoint |

---

## Lab Systems

The homelab contains three dedicated physical systems used for server infrastructure, virtualization, and cybersecurity testing.

| Specification | Desktop 1 | Desktop 2 | Desktop 3 |
|---|---|---|---|
| Model | HP EliteDesk 800 G1 SFF | HP EliteDesk 800 G1 DM | HP Compaq Elite 8300 SFF |
| Operating System | Ubuntu 26.04 LTS | VMware ESXi 8.0.3 | Parrot OS |
| CPU | Intel Core i7-4770 | Intel Core i7-4785T @ 2.20 GHz | Intel Core i5-3470 @ 3.20 GHz |
| CPU Generation | 4th Gen | 4th Gen | 3rd Gen |
| Cores / Threads | 4 / 8 | 4 / 8 | 4 / 4 |
| RAM | 16 GB | 12 GB | 8 GB |
| Storage | 1 TB | 1 TB | 256 GB |
| Graphics | Intel HD Graphics 4600 | Intel HD Graphics 4600 | Intel HD Graphics 2500 |
| Lab Role | Linux Server & Services | VMware Virtualization Host | Cybersecurity Workstation |

### Desktop 1 — Linux Server & Services

The HP EliteDesk 800 G1 SFF running Ubuntu is used for Linux-based server infrastructure and services within the homelab.

Potential workloads include:

- Network monitoring
- Logging
- Security monitoring
- Docker containers
- DNS services
- Web services
- Administrative tools

### Desktop 2 — VMware Virtualization Host

The HP EliteDesk 800 G1 DM runs VMware ESXi 8.0.3 and provides virtualization capabilities for the lab.

Virtual machines can be used for:

- Windows Server
- Windows endpoints
- Linux servers
- Security monitoring systems
- Test environments
- Cybersecurity exercises

### Desktop 3 — Cybersecurity Workstation

The HP Compaq Elite 8300 SFF runs Parrot OS and serves as a dedicated cybersecurity workstation.

It can be used for:

- Network analysis
- Packet inspection
- Vulnerability assessment
- Security testing
- Network scanning
- Cybersecurity exercises
- Traffic analysis

---

## Network Architecture

The high-level network architecture is:

```text
Internet
   |
   v
Rogers XB7 Gateway
   |
   v
OpenWrt Router / Firewall
   |
   | 802.1Q VLAN Trunk
   v
NETGEAR GS308E Managed Switch
   |
   +-- VLAN 10 - Management
   |
   +-- VLAN 20 - Servers
   |
   +-- VLAN 30 - CyberLab
   |
   +-- VLAN 40 - IoT / Test
   |
   +-- VLAN 50 - Forensics
