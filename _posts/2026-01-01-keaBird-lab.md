---
type: pages

title: " Enterprise Edge Lab: Dual-Router Network Infrastructure"
excerpt: "A dual-router CentOS environment implementing BIRD for dynamic routing and Kea for advanced DHCP services across four virtualized clients."

tags:
  - writeup
  - networking
  - centos
  - bird
  - kea
  - nftables
classes: "wide"
---


## Project Overview

This project involves the design and implementation of a robust, segmented network environment using **CentOS**-based routers. The architecture features a dual-router setup providing infrastructure services to four virtualized clients across multiple subnets. Key implementations include **OSPF dynamic routing** via **BIRD**, advanced **DHCP services** using **Kea**, and a multi-layered security approach with **nftables** for packet filtering and Network Address Translation (NAT).

---

## Technical Architecture

The environment is structured into three primary segments connected via a **VirtualBox Host-Only Network**:

**Infrastructure Backbone:** Connects the **Border Router**, **Router 1 (r1)**, and **Router 2 (r2)** via the `10.26.20.0/24` network.



**Net1 (Segmented LAN):** A bridged network (`net1a` and `net1b`) managed by **r1** using the `192.168.100.0/24` range.



**Net2 (Private LAN):** A private internal network managed by **r2** using the `192.168.200.0/24` range.



---

## Core Implementations

### 1. Dynamic Routing with BIRD (OSPF)

To eliminate the need for manual static route management, the **OSPF (Open Shortest Path First)** protocol was implemented using the **BIRD** internet routing daemon.


**Configuration:** Routers were configured to exchange topological information dynamically through the `enp0s3` interface.

 
**Security Adjustment:** For **r2**, BIRD was reconfigured to stop publishing internal status for the private `net2` segment to maintain NAT isolation.

 
**Validation:** Routing tables and OSPF interface statuses were verified using `birdc show ospf interface`.



### 2. Advanced DHCP Services with Kea

Centralized IP management was achieved by deploying the **Kea DHCP4** server on both **r1** and **r2**.


**Dynamic Allocation:** Kea provides automatic IPv4 configuration for the `web`, `ws1`, and `ws2` clients.


 
**Deployment:** Services were enabled and managed via `systemd` after validating the `/etc/kea/kea-dhcp4.conf` syntax.



### 3. Perimeter Security & NAT (nftables)

Security and external connectivity were managed through **nftables**.

 
**Packet Filtering (r1):** A "Default Deny" policy was established. Specific rules allow **SSH**, **OSPF**, **DNS**, and **ICMP** through `enp0s3`, while the `web` host is restricted to HTTP, HTTPS, and SSH traffic.


**NAT & Port Forwarding (r2):** Source NAT (SNAT) was implemented to masquerade `net2` traffic as r2’s external IP.


* **Inbound Access:** Port forwarding rules allow remote management of private clients:
**Port 52022**  `ws2` (Port 22).


**Port 53022**  `ws3` (Port 22).





### 4. Dual-Stack Capabilities (IPv6)

The environment was future-proofed with a parallel **statically routed IPv6 hierarchy**.

**Addressing:** Unique Local Addresses (ULA) were assigned (e.g., `FD00:C0:A8:64::/64` for net1).


**Forwarding:** IPv6 packet forwarding was globally enabled on both routers via `sysctl`.



---

## Administration & Troubleshooting

The lab utilized standard Linux networking toolsets for lifecycle management:

**NetworkManager:** Used `nmtui` and `nmcli` for persistent interface and bridge configurations.


**Verification:** Connectivity was rigorously tested using `traceroute` and `ping` for both IPv4 and IPv6 stacks.


**Security Auditing:** `nmap` was employed to confirm that firewall policies correctly filtered unauthorized ports while allowing essential services.



---

## Final Results

The project successfully delivered a secure, scalable enterprise edge simulation. Clients in isolated subnets can communicate across the inter-network via OSPF, receive dynamic configurations via Kea, and access external resources through NAT, all while protected by a granular nftables firewall.


## References
- [BIRD Internet Routing Daemon](https://bird.network.cz/)
- [Kea DHCP Server](https://kea.readthedocs.io/en/latest/)
- [nftables Official Documentation](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page)        
