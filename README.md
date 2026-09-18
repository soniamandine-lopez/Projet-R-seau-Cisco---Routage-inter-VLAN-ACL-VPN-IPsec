# Cisco Network Security Lab : Inter-VLAN Routing, ACL & IPsec VPN



## 📌 Project Overview

This project is a Cisco networking and cybersecurity lab completed as part of my Bachelor's degree in Cybersecurity.

The objective was to design and configure a small network infrastructure using Cisco Packet Tracer, then progressively implement network segmentation, inter-VLAN routing, access control, and a site-to-site VPN.

The project covers :

* Layer 2 VLAN segmentation
* Switch interface configuration
* IP addressing
* Inter-VLAN routing
* Access Control Lists (ACLs)
* Routing between two networks
* Site-to-site VPN
* ISAKMP configuration
* IPsec encryption
* Network connectivity and security testing

---

## 🎯 Objectives

The main objectives of this lab were to :

1. Create and configure multiple VLANs.
2. Assign switch ports to the appropriate VLANs.
3. Configure IP addressing on the workstations.
4. Implement inter-VLAN routing using a Cisco router.
5. Control traffic between VLANs using an ACL.
6. Connect two routers through a point-to-point network.
7. Configure routing between the two remote networks.
8. Establish a site-to-site IPsec VPN.
9. Verify that traffic is correctly encrypted and transmitted through the VPN tunnel.

---

## 🏗️ Network Architecture

The initial topology consists of :

* 1 Cisco router
* 1 Cisco switch
* 4 workstations

The network is divided into two VLANs.

The project is then extended with a second router to create a site-to-site VPN between two networks.

### Main concepts

```text
                 ┌─────────────────┐
                 │   Cisco Router  │
                 │   Inter-VLAN    │
                 │    Routing      │
                 └────────┬────────┘
                          │
                    Trunk connection
                          │
                 ┌────────┴────────┐
                 │  Cisco Switch   │
                 └───────┬─────────┘
                         │
              ┌──────────┴──────────┐
              │                     │
           VLAN 10               VLAN 20
              │                     │
         ┌────┴────┐           ┌────┴────┐
         │   PC0   │           │   PC2   │
         │   PC1   │           │   PC3   │
         └─────────┘           └─────────┘
```

The initial network is configured so that devices within the same VLAN can communicate, while communication between VLANs requires routing through the router.

---

## 🔹 Part 1 : VLAN Configuration

### 1.1 VLAN Creation

The first step was to segment the network at Layer 2 using VLANs.

The VLANs were created on the Cisco switch before enabling communication between them.

The switch ports were then assigned to their corresponding VLANs.

### 1.2 IP Addressing

Each workstation was configured with a different IP address using a `/24` subnet mask.

The workstations were divided between two networks, with separate gateways for the two VLANs.

### 1.3 Connectivity Testing

Connectivity was tested using Cisco Packet Tracer's simulation tools and ICMP traffic.

Before inter-VLAN routing was configured, devices belonging to different VLANs could not communicate.

This confirmed that the Layer 2 segmentation was working as expected.

---

## 🔀 Part 2 : Inter-VLAN Routing

After configuring the VLANs, the router was configured to allow communication between the different networks.

The router uses separate subinterfaces for each VLAN.

### Configuration concept

```text
Router
│
├── GigabitEthernet0/0.10 → VLAN 10
│
└── GigabitEthernet0/0.20 → VLAN 20
```

This configuration allows the router to act as the gateway for each VLAN and route traffic between the two networks.

The configuration was verified using :

```bash
show ip interface brief
```

### Connectivity Validation

Connectivity tests confirmed that :

* PC0 could reach its gateway.
* PC2 could reach its gateway.
* PC0 could communicate with PC2.
* PC2 could communicate with PC0.

This validated the inter-VLAN routing configuration.

---

## 🛡️ Part 3 : Access Control List (ACL)

Once inter-VLAN routing was operational, an ACL was introduced to control traffic between the two VLANs.

### Security Objective

The ACL was configured with the following policy :

| Traffic             | Expected result                       |
| ------------------- | ------------------------------------- |
| VLAN 10 → VLAN 20   | ❌ Blocked                            |
| VLAN 20 → VLAN 10   | ✅ Allowed                            |
| Other IP traffic    | ✅ Allowed unless explicitly denied   |

The objective was to restrict communication from VLAN 10 to VLAN 20 without blocking communication in the opposite direction.

### ACL Logic

The ACL uses :

```text
deny
permit
```

The `deny` rule blocks the traffic matching the specified source and destination networks.

A subsequent `permit` rule allows IP traffic that has not previously been denied.

### Testing

The ACL was tested using connectivity tests between the workstations.

The results confirmed that :

* PC0 could no longer send traffic to PC2.
* Traffic from PC2 to PC0 was permitted at the request level, but the return traffic was affected by the ACL configuration.
* The ACL counters increased when matching traffic was blocked.

The ACL statistics were also checked to confirm that the deny rule was actually matching packets.

---

## 🔐 Part 4 : Site-to-Site VPN

The final part of the project consisted of creating a site-to-site VPN between two routers.

A second router was added to the topology.

```text
   Local Network                         Remote Network

┌─────────────────┐                  ┌─────────────────┐
│     VLANs       │                  │   Remote LAN    │
│  192.168.x.0/24 │                  │ 192.168.3.0/24  │
└────────┬────────┘                  └────────┬────────┘
         │                                    │
    ┌────▼────┐                          ┌────▼────┐
    │ Router0 │==========================│ Router1 │
    └─────────┘       IPsec VPN          └─────────┘
```

The routers were connected using a point-to-point `/30` network.

The `/30` network provides four addresses, including two usable host addresses, which is appropriate for a point-to-point connection.

---

## 🌐 Part 5 : Routing Between the Two Networks

Static routes were configured so that each router knew how to reach the remote network.

Router0 was configured to reach :

```text
192.168.3.0/24
```

through Router1.

Router1 was configured to route traffic toward the main network through Router0.

Connectivity between the routers and the remote network was then tested before configuring the VPN.

---

## 🔑 Part 6 — IPsec VPN Configuration

The VPN configuration uses IPsec to protect traffic exchanged between the two networks.

An ACL was created specifically to identify the traffic that should be protected by the VPN.

> Note: This ACL has a different purpose from the previous traffic-filtering ACL. It identifies **interesting traffic for the VPN** rather than simply blocking traffic.

---

## 🔒 ISAKMP Configuration

ISAKMP was configured on both routers to establish and manage the secure VPN connection.

The configuration included :

* ISAKMP policy
* Pre-shared key
* Security parameters required to establish the VPN

The required Security Technology Package was also enabled on the Cisco 2911 routers used in the lab.

---

## 🔐 IPsec Configuration

The IPsec configuration was divided into two main components:

### Transform Set

The transform set defines how traffic is protected and encrypted.

### Crypto Map

The crypto map defines :

* Which traffic should use the VPN
* The remote VPN peer
* The IPsec transform set
* The conditions under which the VPN is activated

The crypto map was then applied to the appropriate WAN interface.

The same general configuration was performed on both routers so that they could establish the IPsec tunnel.

---

## 🧪 VPN Testing

To trigger the VPN tunnel, traffic matching the VPN ACL was generated from the local network.

The IPsec statistics were then checked on Router0.

The packet counters increased, confirming that traffic was being processed by IPsec and passing through the VPN tunnel.

This provided a practical validation of the VPN configuration.

---

## 🧰 Technologies & Tools

| Technology / Tool       | Purpose                              |
| ----------------------- | ------------------------------------ |
| Cisco Packet Tracer     | Network simulation                   |
| Cisco IOS               | Router and switch configuration      |
| VLAN                    | Network segmentation                 |
| Inter-VLAN Routing      | Communication between VLANs          |
| ACL                     | Traffic filtering and access control |
| ISAKMP                  | VPN security association management  |
| IPsec                   | Traffic encryption                   |
| ICMP / Ping             | Connectivity testing                 |

---

## 📚 Skills Demonstrated

Through this project, I practiced the following technical skills :

### Networking

* VLAN creation and configuration
* Switch port configuration
* IP addressing
* Subnetting
* Default gateways
* Router configuration
* Inter-VLAN routing
* Static routing
* Point-to-point network configuration

### Network Security

* Access Control Lists
* Traffic filtering
* Network segmentation
* VPN architecture
* ISAKMP
* IPsec
* Pre-shared key authentication
* Encrypted network communication

### Troubleshooting & Validation

* ICMP connectivity testing
* Cisco Packet Tracer simulation mode
* `show ip interface brief`
* ACL verification
* ACL hit-counter analysis
* IPsec statistics verification

---

## 📁 Repository Structure

```text
.
├── README.md
├── packet-tracer/
│   └── network-security-lab.pkt
├── documentation/
│   └── report.pdf
└── screenshots/
    ├── topology.png
    ├── vlan-configuration.png
    ├── inter-vlan-routing.png
    ├── acl-configuration.png
    └── ipsec-vpn.png
```

> The `.pkt` file contains the Cisco Packet Tracer network configuration used during the lab.

---

## ✅ Results

The different stages of the project were successfully tested :

* VLAN segmentation was implemented.
* Inter-VLAN routing was configured and tested.
* Traffic filtering was implemented using an ACL.
* Two routers were connected through a point-to-point network.
* Routing between the two networks was configured.
* An IPsec VPN was established between the routers.
* IPsec traffic counters confirmed that traffic was being processed through the VPN tunnel.

---

## 🎓 Context

Academic project : Bachelor 3 Cybersecurity

This project was completed during the 2026–2027 academic year as part of an advanced Cisco networking lab.

It allowed me to put networking and cybersecurity concepts into practice in a simulated enterprise network environment.

---

## 👤 Author

Sonia LOPEZ

Bachelor's Degree : Cybersecurity

---

## 📌 Key Takeaways

This project helped me understand how different networking and security mechanisms work together :

```text
VLAN
  ↓
Network Segmentation
  ↓
Inter-VLAN Routing
  ↓
Traffic Control with ACL
  ↓
Routing Between Sites
  ↓
IPsec VPN
  ↓
Encrypted Communication
```

The project demonstrates the practical implementation of network segmentation, traffic control, routing, and secure communication using Cisco technologies.

