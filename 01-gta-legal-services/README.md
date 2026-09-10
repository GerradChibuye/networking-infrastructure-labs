# GTA Legal Services Network

## 1. Project Overview

### 1.1 Scenario

GTA Legal Services is a fictional law firm requiring a structured network infrastructure to support its Administration, Lawyers, Finance and Guest users.

The network was designed and implemented in Cisco Packet Tracer as a practical networking and cybersecurity project.

The project follows two stages:

1. **Unsecured Network** — a functional baseline network used to establish and test connectivity.
2. **Secured Network** — the same network after security controls are implemented and tested.

The purpose of this approach is to demonstrate the difference between a network that is simply functional and a network that has been deliberately secured according to organisational requirements.

### 1.2 Objectives

The objectives of this project are to:

* Design a small organisational network using VLAN segmentation.
* Configure IPv4 addressing and DHCP.
* Implement inter-VLAN routing using Router-on-a-Stick.
* Establish and verify baseline connectivity.
* Identify communication requirements between departments.
* Secure network-device management using SSH.
* Implement ACLs to control inter-VLAN communication.
* Implement Layer 2 security using Port Security.
* Secure unused switch interfaces.
* Implement PortFast and BPDU Guard where appropriate.
* Harden the network devices.
* Test and verify the implemented security controls.
* Compare the unsecured and secured network environments.

### 1.3 Technologies

* Cisco Packet Tracer
* Cisco IOS
* VLANs
* 802.1Q Trunking
* Router-on-a-Stick
* IPv4
* DHCP
* Extended ACLs
* SSH
* Port Security
* PortFast
* BPDU Guard
* Cisco device hardening

  2. Network Topology

The following diagram represents the baseline unsecured network implemented in Cisco Packet Tracer.

<img width="1370" height="399" alt="LAWFIRM_UNSECURE" src="https://github.com/user-attachments/assets/c649bf00-1f5a-4d24-914f-0b1e4ca4625b" />

## 3. VLAN & IP Addressing Design

### 3.1 VLAN Design

The network is divided into four VLANs to separate users according to their organisational roles.

| VLAN ID | VLAN Name | Network         | Default Gateway | Users/Devices        |
| ------: | --------- | --------------- | --------------- | -------------------- |
|      10 | ADMIN     | 192.168.10.0/24 | 192.168.10.1    | 2 Admin PCs + Server |
|      20 | LAWYERS   | 192.168.20.0/24 | 192.168.20.1    | 2 Lawyer PCs         |
|      30 | FINANCE   | 192.168.30.0/24 | 192.168.30.1    | 2 Finance Laptops    |
|      40 | GUEST     | 192.168.40.0/24 | 192.168.40.1    | 2 Guest Laptops      |

### 3.2 IP Addressing

Each VLAN uses a `/24` IPv4 subnet, providing a separate Layer 3 network for each department.

The router provides the default gateway for each VLAN:

* VLAN 10: `192.168.10.1`
* VLAN 20: `192.168.20.1`
* VLAN 30: `192.168.30.1`
* VLAN 40: `192.168.40.1`

The client devices use DHCP to obtain their IPv4 configuration automatically.

### 3.3 Address Allocation

The router was configured to provide DHCP services for all four VLANs.

Reserved addresses at the beginning of each subnet are excluded from the DHCP pools so that they can be used for infrastructure and statically addressed resources where required.

The client devices therefore receive their IP addresses dynamically while the VLAN gateway addresses remain fixed.

### 3.4 Network Segmentation

VLAN segmentation creates separate broadcast domains for Administration, Lawyers, Finance and Guests.

At this stage, VLAN segmentation provides logical separation, but it does not by itself enforce the complete communication policy between departments.

Inter-VLAN communication is handled by the router and will be controlled later using Layer 3 security mechanisms.


## 4. Inter-VLAN Routing and DHCP

### 4.1 Inter-VLAN Routing

The network uses **Router-on-a-Stick** to provide communication between the four VLANs.

A single physical link connects the router to the switch:

```text
R1 G0/0
   |
   | 802.1Q Trunk
   |
SW1 Fa0/23
```

The router's `G0/0` interface uses VLAN-specific subinterfaces. Each subinterface acts as the default gateway for its corresponding VLAN.

| Router Interface | VLAN | IP Address      | Function        |
| ---------------- | ---: | --------------- | --------------- |
| G0/0.10          |   10 | 192.168.10.1/24 | ADMIN gateway   |
| G0/0.20          |   20 | 192.168.20.1/24 | LAWYERS gateway |
| G0/0.30          |   30 | 192.168.30.1/24 | FINANCE gateway |
| G0/0.40          |   40 | 192.168.40.1/24 | GUEST gateway   |

The switch port connected to R1, `Fa0/23`, carries traffic for the VLANs over the trunk link.

### 4.2 DHCP

R1 also provides DHCP services for the client devices across the four VLANs.

The DHCP configuration provides addresses from the corresponding departmental networks:

| VLAN | DHCP Network    | Default Gateway |
| ---: | --------------- | --------------- |
|   10 | 192.168.10.0/24 | 192.168.10.1    |
|   20 | 192.168.20.0/24 | 192.168.20.1    |
|   30 | 192.168.30.0/24 | 192.168.30.1    |
|   40 | 192.168.40.0/24 | 192.168.40.1    |

Addresses reserved for infrastructure and other statically configured resources are excluded from the DHCP pools.

The client devices were configured to use DHCP. The following devices successfully obtained IPv4 addresses dynamically:

* PC1
* PC2
* PC3
* PC4
* Laptop1
* Laptop2
* Laptop3
* Laptop4

### 4.3 Connectivity Through the Router

Because each VLAN has a Layer 3 gateway on R1, traffic destined for another VLAN is sent to the router.

For example:

```text
ADMIN PC
192.168.10.x (DHCP)
     |
     | VLAN 10
     ↓
SW1
     |
     | Trunk
     ↓
R1 G0/0.10
192.168.10.1
     |
     | Routing
     ↓
R1 G0/0.20
192.168.20.1
     |
     | VLAN 20
     ↓
LAWYER PC
192.168.20.(DHCP)
```

At the unsecured baseline stage, inter-VLAN routing allows the different departmental networks to communicate according to the routing configuration.

The communication restrictions required by the organisation are addressed later through security controls.

### 4.4 Baseline Configuration State

At this stage of the project, the following functionality has been implemented and verified:

* VLAN segmentation
* 802.1Q trunking
* Router-on-a-Stick
* Inter-VLAN routing
* DHCP
* Dynamic IPv4 addressing for client devices
* Connectivity between the network segments
* Internal server connectivity

No ACL-based traffic restrictions, SSH management controls, Layer 2 port-security controls, or additional device-hardening measures are considered part of this baseline configuration.

## 5. Baseline Connectivity Testing

### 5.1 Test Objective

Connectivity testing was performed to establish the behaviour of the network before security controls were introduced.

The objective was to verify that VLAN connectivity, inter-VLAN routing, DHCP and internal server communication were functioning correctly.

### 5.2 Test Results

The following connectivity tests were successfully completed:

| Test                      | Result |
| ------------------------- | ------ |
| Admin PC → Admin gateway  | PASS   |
| Admin PC → Lawyer PC      | PASS   |
| Admin PC → Finance laptop | PASS   |
| Admin PC → Guest laptop   | PASS   |
| Admin PC → Server         | PASS   |
| Lawyer PC → Admin PC      | PASS   |
| Finance laptop → Admin PC | PASS   |
| Guest laptop → Admin PC   | PASS   |

### 5.3 Baseline Findings

The tests confirmed that the four VLANs were operational and that R1 was successfully providing Layer 3 connectivity between the VLANs.

DHCP was also functioning correctly, allowing client devices to obtain IPv4 addresses automatically.

The baseline network therefore permits communication between the different departmental networks.

This represents the **unsecured state** of the network.

### 5.4 Security Observation

Although VLANs provide logical segmentation and separate broadcast domains, the baseline configuration does not sufficiently restrict communication between the departments.

For example, a device in the Guest VLAN can currently communicate with devices in the Admin VLAN when permitted by the routing configuration.

This is a security concern because different departments have different levels of trust and different access requirements.

The next stage of the project will therefore introduce security controls to enforce the organisation's intended communication policy.

### 5.5 Baseline Status

**Baseline network: OPERATIONAL**

The following functionality was verified:

* VLAN segmentation
* DHCP
* 802.1Q trunking
* Router-on-a-Stick
* Inter-VLAN routing
* Internal server connectivity
* Cross-VLAN communication


## 6. Security Policy Definition

Before implementing Access Control Lists (ACLs), a security policy was defined to determine which VLANs should be permitted or denied communication with one another.

The objective was to move beyond basic VLAN segmentation and establish controlled communication based on the responsibilities of each department.

### 6.1 Security Objectives

The security policy follows the principle of least privilege:

* Administration requires broad access to the organization's internal network and server.
* Lawyers require access to Administration and Finance but should not access the Guest network.
* Finance should not initiate communication with other departmental networks.
* Guests should not access internal departmental networks or the internal server.

### 6.2 Access Control Matrix

| Source VLAN | Destination | Action |
| ----------- | ----------- | ------ |
| ADMIN       | LAWYERS     | ALLOW  |
| ADMIN       | FINANCE     | ALLOW  |
| ADMIN       | GUEST       | ALLOW  |
| ADMIN       | SERVER      | ALLOW  |
| LAWYERS     | ADMIN       | ALLOW  |
| LAWYERS     | FINANCE     | ALLOW  |
| LAWYERS     | GUEST       | DENY   |
| FINANCE     | ADMIN       | DENY   |
| FINANCE     | LAWYERS     | DENY   |
| FINANCE     | GUEST       | DENY   |
| GUEST       | ADMIN       | DENY   |
| GUEST       | LAWYERS     | DENY   |
| GUEST       | FINANCE     | DENY   |
| GUEST       | SERVER      | DENY   |

### 6.3 Policy Rationale

VLANs provide logical network segmentation, but segmentation alone does not determine which networks are allowed to communicate.

Because Router-on-a-Stick provides Layer 3 connectivity between the VLANs, traffic can currently travel between departments unless additional access controls are applied.

ACLs will therefore be used to enforce the defined security policy.

The intended security model is:

**VLANs → provide segmentation**

**Router-on-a-Stick → provides inter-VLAN connectivity**

**ACLs → control permitted and denied traffic**

This approach allows necessary business communication while restricting unnecessary or unauthorized access.

### 6.4 Baseline vs. Secured State

The current `LAWFIRM-UNSECURE` network intentionally has no ACL-based traffic restrictions. This provides a baseline against which the secured implementation can later be tested.

For example, in the baseline state, a Guest device can communicate with an Administration device because:

1. The devices belong to different VLANs.
2. R1 provides routing between VLAN 40 and VLAN 10.
3. No ACL currently denies the traffic.

The secured version of the lab will introduce ACLs to enforce the access control matrix defined above.

### 6.5 Security Transformation

The project therefore follows this progression:

**Initial state**

`VLAN Segmentation → Inter-VLAN Routing → unrestricted communication`

**Secured state**

`VLAN Segmentation → Inter-VLAN Routing → ACL Enforcement → controlled communication`

The purpose of the secure implementation is not to eliminate all inter-VLAN communication, but to ensure that communication occurs according to the organization's defined security requirements.

## 7. Guest Network Access Control

The first security control implemented was an Extended ACL restricting traffic originating from the Guest VLAN.

### 7.1 Security Requirement

Guest users must not be able to access:

* Administration
* Lawyers
* Finance
* Internal servers

The Guest network is:

`192.168.40.0/24`

### 7.2 ACL Configuration

An Extended ACL named `GUEST_RESTRICTIONS` was created on R1.

```cisco
ip access-list extended GUEST_RESTRICTIONS
 deny ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip any any
```

The ACL was applied inbound on the VLAN 40 router subinterface:

```cisco
interface g0/0.40
 ip access-group GUEST-RESTRICTIONS in
```

### 7.3 Configuration Rationale

The ACL is applied **inbound** on `G0/0.40` because this is the Layer 3 interface through which traffic originating from the Guest VLAN enters the router.

This allows R1 to inspect Guest-originated traffic before routing it toward another VLAN.

The three deny statements prevent Guest traffic from reaching the three internal departmental networks.

The final `permit ip any any` allows other traffic not matching those restrictions to continue. This also prevents the ACL's implicit deny from unnecessarily blocking all remaining Guest traffic.

### 7.4 Verification

The following connectivity tests were performed from a Guest device:

| Test                    | Result      |
| ----------------------- | ----------- |
| Guest → Admin           | **BLOCKED** |
| Guest → Lawyers         | **BLOCKED** |
| Guest → Finance         | **BLOCKED** |
| Guest → Internal Server | **BLOCKED** |
| Guest → VLAN 40 Gateway | **PASS**    |

### 7.5 Result

The Guest network is now restricted from accessing the organization's internal departmental networks and server.

This confirms that the ACL is actively enforcing the security policy.

### 7.6 Security Improvement

**Before ACL:**

`Guest → Router → Internal VLAN → ALLOWED`

**After ACL:**

`Guest → Router → ACL → DENIED`

The implementation demonstrates the difference between network segmentation and access control: VLAN 40 remains a separate network, while the ACL now determines what traffic originating from that network is permitted to reach.


## 8. Finance Network Access Control

### 8.1 Security Requirement

The Finance VLAN should not initiate communication with other departmental networks.

The following traffic originating from VLAN 30 must be denied:

* Finance → Administration
* Finance → Lawyers
* Finance → Guest

Finance must still be able to communicate with its own default gateway.

### 8.2 ACL Configuration

An extended ACL named `FINANCE_RESTRICTIONS` was created on R1:

```cisco
ip access-list extended FINANCE_RESTRICTIONS
 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.40.0 0.0.0.255
 permit ip any any
```

The ACL was applied inbound to the Finance subinterface:

```cisco
interface g0/0.30
 ip access-group FINANCE_RESTRICTIONS in
```

### 8.3 Rationale

The ACL filters traffic entering the router from the Finance VLAN.

Because the ACL is applied inbound on `G0/0.30`, traffic originating from `192.168.30.0/24` is inspected before the router forwards it toward another VLAN.

The `permit ip any any` statement allows traffic that does not match the three deny statements to continue through the router.

### 8.4 Verification

The following tests were performed:

| Test                      | Expected Result | Result |
| ------------------------- | --------------- | ------ |
| Finance → Admin           | DENY            | PASS   |
| Finance → Lawyers         | DENY            | PASS   |
| Finance → Guest           | DENY            | PASS   |
| Finance → Finance Gateway | ALLOW           | PASS   |

### 8.5 Result

The Finance VLAN's outbound access to the other departmental networks is now restricted according to the defined security policy.

**Status: SECURED**

### 8.6 Security Improvement

Before the ACL was implemented, Finance hosts could communicate with all routed VLANs.

After implementation, traffic originating from the Finance network is filtered by R1 before being forwarded to the other departmental VLANs.

This demonstrates the transition from basic VLAN segmentation to **policy-based Layer 3 access control**.
