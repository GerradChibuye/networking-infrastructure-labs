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
192.168.10.x
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
192.168.20.x
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

