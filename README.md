# Enterprise Network Architecture & Configuration Project

## 📌 Executive Summary

This project demonstrates the design, deployment, and security implementation of an Enterprise Network infrastructure using **Cisco Packet Tracer**. It simulates a multi-department enterprise connected to an ISP with strict security policies, segmented broadcast domains, dynamic routing, centralized DHCP services, and public IP translation (PAT) for internet connectivity.

---

## 🌐 Network Topology Diagram

```text
                                +---------------------------+
                                |        ISP Router         |
                                |   Loopback0: 8.8.8.8/32   |
                                +-------------+-------------+
                                              |
                                              | WAN Link: 10.1.1.0/30
                                              v
                                +-------------+-------------+
                                |    HQ-R1 Border Router    |
                                |  (OSPF, ACL, NAT/PAT,     |
                                |    Inter-VLAN, DHCP)      |
                                +-------------+-------------+
                                              |
                                              | GigabitEthernet 0/0 (Trunk / 802.1Q)
                                              |
                                +-------------+-------------+
                                |      SW1 Core Switch      |
                                |     (Layer 2 Access)      |
                                +-------+-------------+-----+
                                        |             |
                 +----------------------+             +----------------------+
                 | FastEthernet 0/1-5                 | FastEthernet 0/6-10
                 | VLAN 10 (HR)                       | VLAN 20 (IT)
                 v                                    v
     +-----------+-----------+            +-----------+-----------+
     |     HR Department     |            |     IT Department     |
     |  IP: 192.168.10.0/24  |            |  IP: 192.168.20.0/24  |
     +-----------------------+            +-----------------------+
```

---

## 📊 IP Addressing Plan & VLAN Schema

| Device / Host | Interface / Sub-interface | IP Address / Subnet | Assigned VLAN | Gateway | Description / Function |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **ISP Router** | `Loopback 0` | `8.8.8.8/32` | N/A | N/A | Simulated Public Internet Host |
| **ISP Router** | `g0/0` | `10.1.1.2/30` | N/A | N/A | ISP WAN Gateway Interface |
| **HQ-R1 Border Router** | `g0/1` | `10.1.1.1/30` | N/A | N/A | HQ Edge WAN Interface (Outside NAT) |
| **HQ-R1 Border Router** | Sub-if `g0/0.10` | `192.168.10.1/24` | VLAN 10 | `192.168.10.1` | HR Default Gateway (Inside NAT) |
| **HQ-R1 Border Router** | Sub-if `g0/0.20` | `192.168.20.1/24` | VLAN 20 | `192.168.20.1` | IT Default Gateway (Inside NAT) |
| **SW1 Switch** | `g0/1` | Trunk Mode | 10, 20 | N/A | IEEE 802.1Q Trunk Link to Router |
| **HR Host Pool** | `f0/1 - f0/5` | `192.168.10.11-253/24` | VLAN 10 | `192.168.10.1` | Dynamic IP via Centralized DHCP |
| **IT Host Pool** | `f0/6 - f0/10` | `192.168.20.11-253/24` | VLAN 20 | `192.168.20.1` | Dynamic IP via Centralized DHCP |

---

## 🚀 Key Implemented Features

1. **VLAN Segmentation & 802.1Q Trunking**: Segmented HR and IT departments into isolated broadcast domains using VLAN 10 and VLAN 20.
2. **Router-on-a-Stick Inter-VLAN Routing**: Configured sub-interfaces on HQ-R1 to handle routing between VLANs.
3. **Centralized DHCP Server**: Configured DHCP pools directly on HQ-R1 to dynamically issue IP addresses, default gateways, and DNS configuration.
4. **OSPF Dynamic Routing**: Implemented single-area OSPF (Area 0) across HQ and ISP routers for dynamic route advertisement.
5. **Perimeter Security via Extended ACL**: Enforced security policies to deny HR hosts access to IT infrastructure while preserving outbound internet access.
6. **PAT / NAT Overload**: Mapped private enterprise addresses (`192.168.x.x`) to a single public IP (`10.1.1.1`) for outbound internet sessions.

---

## 🛠️ Complete Configuration CLI Script

### 1. Switch SW1 Configuration (VLANs, Access, and Trunk)

```text
SW1> enable
SW1# configure terminal
SW1(config)# hostname SW1

! Create VLANs
SW1(config)# vlan 10
SW1(config-vlan)# name HR_Dept
SW1(config-vlan)# exit
SW1(config)# vlan 20
SW1(config-vlan)# name IT_Dept
SW1(config-vlan)# exit

! Assign Access Ports to VLAN 10 (HR)
SW1(config)# interface range fastEthernet 0/1 - 5
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
SW1(config-if-range)# no shutdown
SW1(config-if-range)# exit

! Assign Access Ports to VLAN 20 (IT)
SW1(config)# interface range fastEthernet 0/6 - 10
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 20
SW1(config-if-range)# no shutdown
SW1(config-if-range)# exit

! Uplink 802.1Q Trunk Port
SW1(config)# interface gigabitEthernet 0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# no shutdown
SW1(config-if)# end
SW1# write memory
```

### 2. HQ-R1 Border Router (Inter-VLAN, DHCP Pools, OSPF Routing)

```text
HQ-R1> enable
HQ-R1# configure terminal
HQ-R1(config)# hostname HQ-R1

! Sub-interfaces for Inter-VLAN Routing
HQ-R1(config)# interface gigabitEthernet 0/0
HQ-R1(config-if)# no shutdown
HQ-R1(config-if)# exit

HQ-R1(config)# interface gigabitEthernet 0/0.10
HQ-R1(config-subif)# encapsulation dot1Q 10
HQ-R1(config-subif)# ip address 192.168.10.1 255.255.255.0

HQ-R1(config)# interface gigabitEthernet 0/0.20
HQ-R1(config-subif)# encapsulation dot1Q 20
HQ-R1(config-subif)# ip address 192.168.20.1 255.255.255.0
HQ-R1(config-subif)# exit

! Centralized DHCP Pools Configuration
HQ-R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.10
HQ-R1(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.10

HQ-R1(config)# ip dhcp pool VLAN10_HR
HQ-R1(dhcp-config)# network 192.168.10.0 255.255.255.0
HQ-R1(dhcp-config)# default-router 192.168.10.1
HQ-R1(dhcp-config)# dns-server 8.8.8.8
HQ-R1(dhcp-config)# exit

HQ-R1(config)# ip dhcp pool VLAN20_IT
HQ-R1(dhcp-config)# network 192.168.20.0 255.255.255.0
HQ-R1(dhcp-config)# default-router 192.168.20.1
HQ-R1(dhcp-config)# dns-server 8.8.8.8
HQ-R1(dhcp-config)# exit

! WAN Interface Setup
HQ-R1(config)# interface gigabitEthernet 0/1
HQ-R1(config-if)# ip address 10.1.1.1 255.255.255.252
HQ-R1(config-if)# no shutdown
HQ-R1(config-if)# exit

! Dynamic Routing (OSPF Area 0)
HQ-R1(config)# router ospf 1
HQ-R1(config-router)# router-id 1.1.1.1
HQ-R1(config-router)# network 192.168.10.0 0.0.0.255 area 0
HQ-R1(config-router)# network 192.168.20.0 0.0.0.255 area 0
HQ-R1(config-router)# network 10.1.1.0 0.0.0.3 area 0
HQ-R1(config-router)# default-information originate
HQ-R1(config-router)# exit
```

### 3. Security Extended ACL & PAT / NAT Overload Configuration

```text
! Extended Access Control List (Deny HR access to IT network)
HQ-R1(config)# ip access-list extended SECURE_ACL
HQ-R1(config-ext-nacl)# deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
HQ-R1(config-ext-nacl)# permit ip any any
HQ-R1(config-ext-nacl)# exit

! Apply Extended ACL on VLAN 10 Inbound
HQ-R1(config)# interface gigabitEthernet 0/0.10
HQ-R1(config-subif)# ip access-group SECURE_ACL in
HQ-R1(config-subif)# exit

! Enable NAT/PAT (Port Address Translation)
HQ-R1(config)# interface gigabitEthernet 0/0.10
HQ-R1(config-subif)# ip nat inside
HQ-R1(config)# interface gigabitEthernet 0/0.20
HQ-R1(config-subif)# ip nat inside
HQ-R1(config)# interface gigabitEthernet 0/1
HQ-R1(config-if)# ip nat outside
HQ-R1(config-if)# exit

HQ-R1(config)# access-list 1 permit 192.168.10.0 0.0.0.255
HQ-R1(config)# access-list 1 permit 192.168.20.0 0.0.0.255
HQ-R1(config)# ip nat inside source list 1 interface gigabitEthernet 0/1 overload
HQ-R1# write memory
```

### 4. ISP Router Setup

```text
ISP> enable
ISP# configure terminal
ISP(config)# hostname ISP

ISP(config)# interface loopback 0
ISP(config-if)# ip address 8.8.8.8 255.255.255.255
ISP(config-if)# exit

ISP(config)# interface gigabitEthernet 0/0
ISP(config-if)# ip address 10.1.1.2 255.255.255.252
ISP(config-if)# no shutdown
ISP(config-if)# exit

ISP(config)# router ospf 1
ISP(config-router)# router-id 2.2.2.2
ISP(config-router)# network 10.1.1.0 0.0.0.3 area 0
ISP(config-router)# network 8.8.8.8 0.0.0.0 area 0
ISP(config-router)# exit
ISP# write memory
```

---

## 🧪 Verification & Troubleshooting Commands

Use the following operational Cisco IOS commands to test and verify connectivity:

* **Verify VLANs & Trunk Ports**:
  ```text
  SW1# show vlan brief
  SW1# show interfaces trunk
  ```
* **Verify Centralized DHCP Leases**:
  ```text
  HQ-R1# show ip dhcp binding
  ```
* **Verify OSPF Adjacency & Routing Table**:
  ```text
  HQ-R1# show ip ospf neighbor
  HQ-R1# show ip route ospf
  ```
* **Inspect Active PAT Session Table**:
  ```text
  HQ-R1# show ip nat translations
  ```
* **Traffic Isolation & ICMP Ping Verification**:
  * Outbound Ping to ISP (`ping 8.8.8.8`) from HR/IT hosts: **Successful**
  * Inter-VLAN Ping from HR (`192.168.10.x`) to IT (`192.168.20.x`): **Blocked by ACL**

---

## 👤 Author Information

* **Role**: Network / NOC Engineer
* **Environment**: Cisco Packet Tracer v8.x (ISR 4331, Catalyst 2960)
