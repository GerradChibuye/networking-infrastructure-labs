# LAWFIRM-UNSECURE

## 1. Project Overview

This project represents the initial, intentionally unsecured network design for a small law firm.

The network was designed to provide connectivity between different departments using VLAN segmentation, DHCP, and Router-on-a-Stick inter-VLAN routing.

The purpose of this version was to establish a functional baseline and identify security weaknesses before implementing the secured design.

The network contains four departmental VLANs:

* VLAN 10 — ADMIN
* VLAN 20 — LAWYERS
* VLAN 30 — FINANCE
* VLAN 40 — GUEST

A centralized server is connected to the ADMIN VLAN.

---

## 2. Network Topology

The topology consists of:

* 1 router — R1
* 1 Layer 2 switch — SW1
* 2 ADMIN PCs
* 2 LAWYERS PCs
* 2 FINANCE laptops
* 2 GUEST laptops
* 1 server

### Connections

* R1 connects to SW1 through Fa0/23.
* The R1–SW1 link operates as an 802.1Q trunk.
* The server connects to SW1 through Fa0/24.
* End devices connect to access ports assigned to their respective VLANs.

Topology diagram:

`topology/topology-unsecure.png`

---

## 3. VLAN Design

| VLAN | Department | Network         | Gateway      |
| ---- | ---------- | --------------- | ------------ |
| 10   | ADMIN      | 192.168.10.0/24 | 192.168.10.1 |
| 20   | LAWYERS    | 192.168.20.0/24 | 192.168.20.1 |
| 30   | FINANCE    | 192.168.30.0/24 | 192.168.30.1 |
| 40   | GUEST      | 192.168.40.0/24 | 192.168.40.1 |

Each VLAN represents a separate Layer 2 broadcast domain.

However, VLAN segmentation alone does not prevent communication between VLANs once traffic is routed at Layer 3.

---

## 4. IP Addressing and DHCP

R1 provides DHCP services for the four VLANs.

The DHCP pools correspond to:

* ADMIN — 192.168.10.0/24
* LAWYERS — 192.168.20.0/24
* FINANCE — 192.168.30.0/24
* GUEST — 192.168.40.0/24

The default gateway for each network is the corresponding R1 subinterface.

---

## 5. Inter-VLAN Routing

Router-on-a-Stick was implemented on R1.

The physical interface connects to SW1 through a trunk, while subinterfaces provide Layer 3 gateways for the VLANs.

The configured subinterfaces are:

```text
G0/0.10 → VLAN 10 → 192.168.10.1/24
G0/0.20 → VLAN 20 → 192.168.20.1/24
G0/0.30 → VLAN 30 → 192.168.30.1/24
G0/0.40 → VLAN 40 → 192.168.40.1/24
```

This allowed hosts in different VLANs to communicate through R1.

---

## 6. Baseline Security State

The network was intentionally left without additional security controls.

The baseline did not include:

* Inter-VLAN ACL restrictions
* Layer 2 Port Security
* SSH-only management
* Management-plane ACLs
* Dedicated unused-port VLAN
* Shutdown of unused switch ports

The purpose was not to create a production-ready network, but to establish a working network against which the secured version could be compared.

---

## 7. Baseline Testing

Connectivity testing was performed before security controls were implemented.

Results:

| Source  | Destination   | Result |
| ------- | ------------- | ------ |
| ADMIN   | ADMIN Gateway | PASS   |
| ADMIN   | LAWYERS       | PASS   |
| ADMIN   | FINANCE       | PASS   |
| ADMIN   | GUEST         | PASS   |
| ADMIN   | SERVER        | PASS   |
| LAWYERS | ADMIN         | PASS   |
| FINANCE | ADMIN         | PASS   |
| GUEST   | ADMIN         | PASS   |

Complete test results are available in:

`verification/baseline-tests.txt`

---

## 8. Security Findings

The baseline testing demonstrated that the network was operational but insufficiently restricted.

### Finding 1 — Unrestricted Inter-VLAN Communication

Hosts in different VLANs could communicate through R1.

For example:

```text
GUEST → ADMIN = ALLOWED
FINANCE → ADMIN = ALLOWED
```

This demonstrated that VLANs alone were not sufficient to enforce departmental security boundaries.

### Finding 2 — No Layer 2 Port Security

Access ports accepted devices without restricting the permitted MAC address.

An unauthorized device could therefore be physically connected to an employee-facing port.

### Finding 3 — Insecure Management Plane

The initial management configuration did not enforce secure SSH-only administration.

Telnet was subsequently demonstrated as an insecure management protocol because its traffic is transmitted without encryption.

### Finding 4 — Unused Ports

Unused switch ports were not isolated or administratively shut down.

This increased the number of interfaces through which an unauthorized device could potentially be connected.

---

## 9. Security Requirements Derived From the Baseline

The findings led to the following security requirements for the secured network:

1. Restrict unnecessary inter-VLAN communication using ACLs.
2. Prevent unauthorized devices from using protected employee access ports.
3. Replace Telnet with SSH version 2.
4. Restrict router management access to the ADMIN network.
5. Move unused switch ports to a dedicated unused VLAN.
6. Administratively shut down unused interfaces.
7. Verify that the implemented controls enforce the intended access policy.

---

## 10. Transition to LAWFIRM-SECURE

The baseline network was subsequently hardened to create the `LAWFIRM-SECURE` version.

The secured version introduced:

* Extended ACLs
* Layer 2 Port Security
* Sticky MAC learning
* Restrict violation mode
* SSH version 2
* Local user authentication
* Management-plane ACL
* VLAN 999 for unused ports
* Administrative shutdown of unused ports

The secured version was then tested against the defined security policy.

---

## 11. Skills Demonstrated

This baseline project demonstrates understanding of:

* VLAN segmentation
* IPv4 addressing
* DHCP
* 802.1Q trunking
* Router-on-a-Stick
* Inter-VLAN routing
* Network connectivity testing
* Security assessment
* Identification of network security weaknesses
* Security requirements analysis

---

## 12. Methodology

The project followed a simple security engineering process:

```text
Build
  ↓
Test
  ↓
Identify weaknesses
  ↓
Define security requirements
  ↓
Implement controls
  ↓
Retest
  ↓
Document
```

The UNSECURE version represents the baseline stage of this process.

---

## 13. Conclusion

The LAWFIRM-UNSECURE network successfully provided connectivity between all tested VLANs.

However, baseline testing demonstrated that functional connectivity did not automatically provide security.

The project therefore progressed from a functional network to a controlled network by introducing security mechanisms appropriate to the identified risks.

The resulting hardened implementation is documented separately as `LAWFIRM-SECURE`.
