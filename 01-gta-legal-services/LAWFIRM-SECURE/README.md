# LAWFIRM-SECURE

## Secure VLAN-Based Network Design and Implementation

**Platform:** Cisco Packet Tracer
**Network Type:** Small Enterprise / Law Firm
**Security Focus:** VLAN Segmentation, ACLs, Layer 2 Security, SSH Management

---

## 1. Project Overview

LAWFIRM-SECURE is a secure network implementation for a small law firm consisting of four logical departments:

* Administration
* Lawyers
* Finance
* Guest Users

The project began with a functional but insecure network, `LAWFIRM-UNSECURE`, in which VLAN segmentation and inter-VLAN routing were implemented but no meaningful traffic restrictions or management-plane security controls were present.

The objective of this project was to transform that network into a more secure enterprise network using practical Cisco security controls while maintaining required business connectivity.

The project follows the principle:

> **Segment → Restrict → Harden → Verify**

---

## 2. Business Requirements

The law firm requires:

* Departmental network segmentation.
* Communication between departments where business operations require it.
* Restricted access from the Guest network to internal resources.
* Restricted access from Finance to other departmental networks.
* Restricted access from Lawyers to the Guest network.
* Secure remote management of the router.
* Protection against unauthorized devices being connected to employee access ports.
* Shutdown of unused switch ports.
* Continued DHCP and inter-VLAN connectivity where permitted.

---

## 3. Network Topology

The network uses a single Cisco router and switch.

* `R1` provides Layer 3 routing and DHCP.
* `SW1` provides Layer 2 switching and VLAN segmentation.
* `Fa0/23` is the 802.1Q trunk between SW1 and R1.
* `Fa0/24` connects the internal server.
* Router-on-a-Stick is used for inter-VLAN routing.

<img width="1366" height="768" alt="lawfirm-secure" src="https://github.com/user-attachments/assets/fb93b9a0-a049-4483-b8db-098c86896773" />



---

## 4. VLAN and IP Addressing Design

| VLAN | Department | Network         | Default Gateway |
| ---- | ---------- | --------------- | --------------- |
| 10   | ADMIN      | 192.168.10.0/24 | 192.168.10.1    |
| 20   | LAWYERS    | 192.168.20.0/24 | 192.168.20.1    |
| 30   | FINANCE    | 192.168.30.0/24 | 192.168.30.1    |
| 40   | GUEST      | 192.168.40.0/24 | 192.168.40.1    |
| 999  | UNUSED     | No user hosts   | N/A             |

### Switch Port Assignment

| Ports        |  VLAN | Purpose           |
| ------------ | ----: | ----------------- |
| Fa0/1–Fa0/2  |    10 | ADMIN             |
| Fa0/3–Fa0/4  |    20 | LAWYERS           |
| Fa0/5–Fa0/6  |    30 | FINANCE           |
| Fa0/7–Fa0/8  |    40 | GUEST             |
| Fa0/9–Fa0/22 |   999 | Unused / shutdown |
| Fa0/23       | Trunk | R1                |
| Fa0/24       |    10 | Internal Server   |
| Gi0/1–Gi0/2  |   999 | Unused / shutdown |

---

## 5. Initial Network — LAWFIRM-UNSECURE

The original network provided:

* VLAN segmentation.
* DHCP.
* Router-on-a-Stick.
* Inter-VLAN routing.
* Server connectivity.

However, the network had no effective restrictions between the routed VLANs.

For example, before security controls were implemented:

```text
GUEST → ADMIN       ALLOW
FINANCE → ADMIN     ALLOW
LAWYERS → GUEST     ALLOW
```

This demonstrated an important networking principle:

> **VLANs separate Layer 2 broadcast domains; they do not automatically prevent Layer 3 communication between those VLANs.**

Because R1 was routing between the VLANs, additional Layer 3 access control was required.

---

# 6. Security Policy

The security policy was defined before implementing the ACLs.

| Source  | Destination | Policy |
| ------- | ----------- | ------ |
| ADMIN   | LAWYERS     | ALLOW  |
| ADMIN   | FINANCE     | ALLOW  |
| ADMIN   | GUEST       | ALLOW  |
| ADMIN   | SERVER      | ALLOW  |
| LAWYERS | ADMIN       | ALLOW  |
| LAWYERS | FINANCE     | ALLOW  |
| LAWYERS | GUEST       | DENY   |
| FINANCE | ADMIN       | DENY   |
| FINANCE | LAWYERS     | DENY   |
| FINANCE | GUEST       | DENY   |
| GUEST   | ADMIN       | DENY   |
| GUEST   | LAWYERS     | DENY   |
| GUEST   | FINANCE     | DENY   |
| GUEST   | SERVER      | DENY   |

The implementation follows a least-privilege approach: traffic is permitted where business requirements require it and restricted where access is unnecessary.

---

# 7. Guest Network Security

An extended ACL named `GUEST_RESTRICTIONS` was created.

```cisco
ip access-list extended GUEST_RESTRICTIONS
 deny ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip any any
```

The ACL was applied inbound on the Guest subinterface:

```cisco
interface g0/0.40
 ip access-group GUEST_RESTRICTIONS in
```

This prevents Guest clients from initiating IP communication with internal departmental networks.

---

# 8. Lawyers Network Security

Lawyers require access to Administration and Finance but should not access the Guest network.

```cisco
ip access-list extended LAWYERS_RESTRICTIONS
 deny ip 192.168.20.0 0.0.0.255 192.168.40.0 0.0.0.255
 permit ip any any
```

Applied to:

```cisco
interface g0/0.20
 ip access-group LAWYERS_RESTRICTIONS in
```

---

# 9. Finance Network Security

Finance was isolated from the other departmental networks according to the security policy.

```cisco
ip access-list extended FINANCE_RESTRICTIONS
 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.40.0 0.0.0.255
 permit ip any any
```

Applied to:

```cisco
interface g0/0.30
 ip access-group FINANCE_RESTRICTIONS in
```

---

# 10. Layer 2 Port Security

Port Security was implemented on employee-facing access ports.

Protected ports:

```text
Fa0/1
Fa0/2
Fa0/3
Fa0/4
Fa0/5
Fa0/6
```

The configuration uses:

* Maximum 1 secure MAC address.
* Sticky MAC learning.
* Violation mode `restrict`.

Example:

```cisco
interface fa0/1
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```

### Why `restrict`?

Three violation modes were investigated:

| Mode     | Unauthorized Traffic | Port State   | Violation Recorded |
| -------- | -------------------- | ------------ | ------------------ |
| Protect  | Dropped              | Remains up   | No                 |
| Restrict | Dropped              | Remains up   | Yes                |
| Shutdown | Dropped              | Err-disabled | Yes                |

`restrict` was selected because it provides a balance between security and availability. Unauthorized traffic is blocked and violations are recorded without automatically disabling the entire switch port.
see linkedin post here: https://www.linkedin.com/posts/gerrad-chibuye-a9298b256_networking-cybersecurity-cisco-activity-7503758906763161601-Gm1o

The behavior was experimentally verified by connecting an unauthorized device to a protected port.

---

# 11. Secure Router Management with SSH

Telnet was deliberately tested during the project to understand its security weakness.

Telnet successfully provided remote CLI access, but its management traffic is transmitted without encryption.

The network was subsequently migrated to SSH.

### SSH Configuration

```cisco
hostname R1
ip domain-name lawfirm.local
crypto key generate rsa
username admin privilege 15 secret <configured-secret>

line vty 0 4
 login local
 transport input ssh

ip ssh version 2
```

Telnet was removed from the VTY lines.

SSH version 2 was verified using:

```cisco
show ip ssh
```

---

# 12. Management-Plane Access Control

Only the ADMIN subnet is permitted to remotely manage R1.

```cisco
ip access-list standard MGMT-ADMIN-ONLY
 permit 192.168.10.0 0.0.0.255
 deny any
```

Applied to the VTY lines:

```cisco
line vty 0 4
 access-class MGMT-ADMIN-ONLY in
```

### Result

```text
ADMIN → R1 SSH       ALLOW
LAWYERS → R1 SSH     DENY
FINANCE → R1 SSH     DENY
GUEST → R1 SSH       DENY
```

This separates normal network access from administrative access to the infrastructure itself.

---

# 13. Unused Port Hardening

Unused switch ports were moved to an isolated VLAN and administratively disabled.

```cisco
vlan 999
 name UNUSED

interface range fa0/9 - 22
 switchport mode access
 switchport access vlan 999
 shutdown

interface range gigabitEthernet0/1 - 2
 switchport mode access
 switchport access vlan 999
 shutdown
```

This reduces the risk of an unauthorized device being connected to an unused physical interface.

Ports `Fa0/23` and `Fa0/24` were deliberately excluded because they are active infrastructure/server connections.

---

# 14. DHCP

R1 provides DHCP services for the four operational VLANs.

Verified DHCP pools:

```text
ADMIN
LAWYERS
FINANCE
GUEST
```

DHCP bindings were verified for client devices across the network.

The DHCP configuration was retained after security implementation to ensure that security controls did not interfere with normal address assignment.

---

# 15. Verification

The network was tested after implementation from multiple VLANs.

### Administration

```text
ADMIN → LAWYERS      PASS
ADMIN → FINANCE      PASS
ADMIN → GUEST        PASS
ADMIN → SERVER       PASS
ADMIN → R1 SSH       PASS
```

### Lawyers

```text
LAWYERS → ADMIN      PASS
LAWYERS → FINANCE    PASS
LAWYERS → GUEST      DENIED
LAWYERS → R1 SSH     DENIED
```

### Finance

```text
FINANCE → ADMIN      DENIED
FINANCE → LAWYERS    DENIED
FINANCE → GUEST      DENIED
FINANCE → R1 SSH     DENIED
```

### Guest

```text
GUEST → ADMIN        DENIED
GUEST → LAWYERS      DENIED
GUEST → FINANCE      DENIED
GUEST → SERVER       DENIED
GUEST → R1 SSH       DENIED
```

All required security policies behaved as expected.

---

# 16. Configuration Verification

The following Cisco commands were used during the audit:

```cisco
show vlan brief
show interfaces status
show interfaces trunk
show interfaces fa0/23 switchport
show port-security
show port-security address
show ip interface brief
show ip dhcp binding
show ip dhcp pool
show access-lists
show ip interface g0/0.20
show ip interface g0/0.30
show ip interface g0/0.40
show ip ssh
show running-config | section line vty
```

These commands were used to verify VLAN membership, trunk operation, port security, DHCP, ACL attachment, SSH configuration, and interface state.

---

# 17. LAWFIRM-UNSECURE vs LAWFIRM-SECURE

| Capability            | UNSECURE | SECURE        |
| --------------------- | -------- | ------------- |
| VLAN segmentation     | Yes      | Yes           |
| Inter-VLAN routing    | Yes      | Yes           |
| DHCP                  | Yes      | Yes           |
| Guest restrictions    | No       | Yes           |
| Finance restrictions  | No       | Yes           |
| Lawyers restrictions  | No       | Yes           |
| SSH management        | No       | Yes           |
| Telnet management     | Yes      | No            |
| Admin-only management | No       | Yes           |
| Port Security         | No       | Yes           |
| Sticky MAC            | No       | Yes           |
| Unused port hardening | No       | Yes           |
| Dedicated unused VLAN | No       | Yes           |
| Security verification | Limited  | Comprehensive |

The key improvement was not simply adding more configuration. The network was changed according to an explicit security policy and then tested against that policy.

---

# 18. Troubleshooting and Lessons Learned

### VLANs are not ACLs

VLANs provide Layer 2 segmentation, but routing between VLANs can still allow communication. Layer 3 controls such as ACLs are required when communication must be restricted.

### Port Security is device-based

Port Security identifies devices using MAC addresses. It does not identify the human using the device.

### `shutdown` vs `restrict`

The project deliberately experimented with both violation behaviors.

`shutdown` provides stronger immediate enforcement but can place a port into an err-disabled state and require recovery.

`restrict` blocks unauthorized traffic while keeping the port operational and recording violations.

### Telnet vs SSH

Telnet demonstrated that remote management can function while still being insecure. SSH provides encrypted management communication.

### Security must be verified

A configuration that looks correct is not enough. The project used actual connectivity tests from multiple VLANs to confirm that the implemented policy worked in practice.

---

# 19. Limitations

This is a Packet Tracer simulation and therefore does not represent every capability or behavior of production Cisco hardware.

The project also does not implement:

* 802.1X authentication
* Centralized AAA
* RADIUS/TACACS+
* DHCP snooping
* Dynamic ARP Inspection
* IP Source Guard
* Spanning Tree security
* Network monitoring/SIEM
* Redundant switching
* Redundant routing
* High-availability firewall infrastructure

These were intentionally outside the scope of this project.

---

# 20. Potential Future Improvements

A production implementation could be extended with:

1. 802.1X for identity-based network access.
2. Centralized AAA using RADIUS or TACACS+.
3. DHCP Snooping.
4. Dynamic ARP Inspection.
5. IP Source Guard.
6. BPDU Guard and Root Guard.
7. Dedicated management VLAN.
8. More restrictive trunk configuration.
9. Redundant switches and routers.
10. Centralized network monitoring and logging.

---

# 21. Skills Demonstrated

This project demonstrates practical experience with:

* Cisco IOS
* VLAN configuration
* 802.1Q trunking
* Router-on-a-Stick
* Inter-VLAN routing
* DHCP
* Extended ACLs
* Standard ACLs
* VTY access control
* SSH
* Telnet security analysis
* Layer 2 Port Security
* Sticky MAC learning
* Security violation handling
* Switch port hardening
* Network troubleshooting
* Security policy implementation
* Connectivity verification
* Network documentation

---

# 22. Project Methodology

The project followed a repeatable engineering workflow:

```text
Understand the business requirements
            ↓
Build the functional network
            ↓
Test baseline connectivity
            ↓
Identify security weaknesses
            ↓
Define security policy
            ↓
Implement controls
            ↓
Test each control
            ↓
Audit configuration
            ↓
Perform end-to-end verification
            ↓
Document the final architecture
```

This approach is intended to demonstrate not only configuration ability, but also the ability to reason about network security, test assumptions, troubleshoot failures, and document engineering decisions.

---

## 23. Conclusion

LAWFIRM-SECURE demonstrates the transformation of a functional but overly permissive network into a segmented and access-controlled enterprise network.

The final design combines:

**VLANs + ACLs + Port Security + SSH + Management Access Control + Unused-Port Hardening**

to enforce the defined security policy while preserving required business communication.

The project also served as a practical exercise in moving from:

**Configuration → Security Design → Testing → Troubleshooting → Verification → Documentation**

rather than treating networking as simply a collection of Cisco commands.
