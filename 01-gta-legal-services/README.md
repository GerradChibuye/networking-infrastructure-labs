# Lab 1 — GTA Legal Services Network

## 1. Project Overview

**Project:** GTA Legal Services Network Infrastructure & Security Lab

**Type:** Networking & Cybersecurity Practical Lab

**Platform:** Cisco Packet Tracer

**Primary Technologies:** Cisco IOS, VLANs, Router-on-a-Stick, DHCP, ACLs, SSH and Port Security

### 1.1 Scenario

GTA Legal Services is a fictional law firm requiring a structured network infrastructure for its different departments.

The organisation has four primary network segments:

* Administration
* Lawyers
* Finance
* Guests

A dedicated server is also deployed within the internal network.

The initial objective was to build a functional network that provides connectivity between the required departments. Once the baseline network was operational, security controls were introduced to restrict unauthorised communication and protect the infrastructure.

The project therefore follows a two-stage approach:

```text
UNSECURED NETWORK
       ↓
BASELINE TESTING
       ↓
IDENTIFY SECURITY REQUIREMENTS
       ↓
SECURITY IMPLEMENTATION
       ↓
SECURED NETWORK
       ↓
SECURITY TESTING & COMPARISON
```

### 1.2 Objectives

The lab aims to demonstrate the ability to:

1. Design a segmented network for a small organisation.
2. Implement VLAN-based network segmentation.
3. Configure inter-VLAN routing using Router-on-a-Stick.
4. Provide dynamic IP addressing using DHCP.
5. Establish and test baseline connectivity.
6. Secure network-device management using SSH.
7. Implement ACLs to enforce departmental communication policies.
8. Implement switch port security.
9. Harden unused switch interfaces.
10. Implement PortFast and BPDU Guard on appropriate access ports.
11. Test security controls and verify expected behaviour.
12. Troubleshoot configuration and connectivity problems.
13. Compare an unsecured network against its secured state.

### 1.3 Network Devices

| Device | Model       | Quantity | Purpose                                        |
| ------ | ----------- | -------: | ---------------------------------------------- |
| Router | Cisco 2911  |        1 | Inter-VLAN routing, DHCP and security controls |
| Switch | Cisco 2960  |        1 | VLAN segmentation and Layer 2 security         |
| PCs    | End devices |        8 | Departmental clients                           |
| Server | Server      |        1 | Internal server resource                       |
