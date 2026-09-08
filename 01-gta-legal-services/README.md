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


2.3 Switch Port Assignments
Switch Port	Device/Connection	VLAN	Purpose
Fa0/1	R1 G0/0	Trunk	Router-on-a-Stick
Fa0/2	Admin PC 1	10	Administration
Fa0/3	Admin PC 2	10	Administration
Fa0/4	Lawyer PC 1	20	Lawyers
Fa0/5	Lawyer PC 2	20	Lawyers
Fa0/6	Finance PC 1	30	Finance
Fa0/7	Finance PC 2	30	Finance
Fa0/8	Guest PC 1	40	Guests
Fa0/9	Guest PC 2	40	Guests
Fa0/10	Internal Server	10	Server
Fa0/11–Fa0/24	Unused	—	Reserved/unused interfaces
2.4 Logical Network Architecture

The network uses four VLANs to logically separate the organisation's departments.

                         R1
                    Cisco 2911
                       G0/0
                        |
                     TRUNK
                        |
                      Fa0/1
                        |
                 +-------------+
                 |     SW1     |
                 | Cisco 2960  |
                 +-------------+
                  |    |    |    |
                 VLAN  VLAN VLAN VLAN
                  10    20   30   40
                  |     |    |    |
               Admin  Law  Finance Guest

VLAN 10 also contains the internal server.

2.5 VLAN Architecture
VLAN	Name	Network	Default Gateway	Purpose
10	ADMIN	192.168.10.0/24	192.168.10.1	Administration and internal server
20	LAWYERS	192.168.20.0/24	192.168.20.1	Lawyers
30	FINANCE	192.168.30.0/24	192.168.30.1	Finance
40	GUEST	192.168.40.0/24	192.168.40.1	Guest users
2.6 Router-on-a-Stick Design

The physical connection between R1 and SW1 is configured as a trunk.

R1 uses subinterfaces to provide a Layer 3 gateway for each VLAN:

Interface	VLAN	IP Address
G0/0.10	10	192.168.10.1/24
G0/0.20	20	192.168.20.1/24
G0/0.30	30	192.168.30.1/24
G0/0.40	40	192.168.40.1/24

This architecture allows the single physical router interface to route traffic between multiple VLANs while maintaining logical network segmentation.
