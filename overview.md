# Mini Project ( Computer Network )

**Name:** Shrine Sigdel 
**Roll No:** 079bct080 
**Course:** Computer Networks Mini-Project Work

---

## 1. Introduction

This proposal presents a compact WAN/LAN design for **Large tech. organization (KtmTechnologies)**, a technology and electronics firm with a corporate HQ, an R&D campus, a production and logistics plant, and one branch sales office. It uses a multi-area OSPF hierarchy (Area 0 backbone plus Areas 1–2) to reflect the corporate structure. End devices are grouped into departmental LANs and VLANs (Finance, HR, Legal, Guest), and **10 routers** are interconnected using dedicated point-to-point links (P01–P14) across a redundant core with dual-homed distribution. IP addressing has been planned using Variable Length Subnet Masking (VLSM) on the private block **10.0.0.0/16** to minimize address wastage while leaving room for future growth.

---

## 2. Network Topology

### 2.1 Network Requirements (As per Specification)

- IP address block: At least /22 or larger
- Networks: Minimum 9 networks, at least 6 different sizes (excluding point-to-point links), with at least 3 VLANs over at least 3 switches
- Routers: At least 9, with 3 routers without LAN connectivity
- Routing: OSPF (internal), default route to ISP; static route from ISP back; at least 3 areas including backbone
- Servers: 2+ DNS (caching + own web resolution) in different LANs, 2+ Web servers on different subnets, ISP-level DNS, per-LAN DHCP
- Topology: Multiple paths in at least two networks

### 2.2 OSPF Area Plan

| Area | Member Networks |
|------|-----------------|
| Area 0 (Backbone) | Border + core routers, Data Center Server LAN, Guest VLAN |
| Area 1 | HQ Corporate (Finance, HR, Legal VLANs), Sales, Marketing |
| Area 2 | R&D, Production Floor, Warehouse & Logistics, Branch Office |

### 2.3 Logical Topology in CISCO Packet Tracer

The topology uses Cisco 2911 routers (with HWIC-2T for serial WAN links), Cisco 2960 switches and generic Server-PT devices. Three routers — **BR, CR1 and CR2** — are pure transit routers with **no LAN connectivity**, satisfying the minimum requirement. A redundant OSPF core loop exists between CR1 and CR2, and the Data Center (R11), R&D (R5) and HQ (R4) are dual-homed to both core routers to provide multiple paths. Production (R6), Warehouse (R7) and the remote Branch (R9) hang off the core within Area 2. Finance, HR and Legal VLANs are trunked to R4 (router-on-a-stick) across three HQ access switches.

| Device | Area | Connects to | LAN(s) served |
|--------|------|-------------|---------------|
| ISP-R1 | ISP | BR (P01) | — (holds ISP DNS) |
| BR | 0 | ISP, CR1, CR2 (P01–P03) | none (transit) |
| CR1 | 0 | BR, CR2, R11, R5, R4 (P02, P04, P05, P07, P09) | none (transit) |
| CR2 | 0 | BR, CR1, R11, R5, R4, R6 (P03, P04, P06, P08, P10, P12) | none (transit) |
| R11 | 0 | CR1, CR2 (P05, P06) | SW-DC (Server LAN), SW-GUEST |
| R4 | 1 | CR1, CR2, R8 (P09–P11) | SW-FIN/HR/LEGAL (Finance/HR/Legal VLANs) |
| R8 | 1 | R4 (P11) | SW-SALES, SW-MKT |
| R5 | 2 | CR1, CR2 (P07, P08) | SW-RD |
| R6 | 2 | CR2, R7 (P12, P13) | SW-PROD |
| R7 | 2 | R6, R9 (P13, P14) | SW-WH |
| R9 | 2 | R7 (P14) | SW-BRANCH |

**Switches:** SW-DC, SW-GUEST (Area 0); SW-FIN, SW-HR, SW-LEGAL, SW-SALES, SW-MKT (Area 1); SW-RD, SW-PROD, SW-WH, SW-BRANCH (Area 2).

This topology is designed for the idea of a network, so there is no PC connected. PCs and other end devices will be added as per need at the time of final configuration.

---

## 3. Addressing Description

### 3.1 IP Address Pool

- **Base network block:** 10.0.0.0 /16 (private address space) — deliberately distinct from 172.16.0.0/16
- **Effective usage:** LAN/VLAN subnets occupy 10.0.0.0 – 10.0.6.207; point-to-point links occupy 10.0.12.0/24
- **Reserved for growth:** 10.0.7.0/24 – 10.0.11.255 and 10.0.13.0/24 onward (~3,000+ addresses) for future plants, branches and servers
- **Upstream ISP block:** the single BR↔ISP link (P01) is the public WAN on documentation test block 198.51.100.0/30 (ISP router 198.51.100.1, BR 198.51.100.2). The ISP DNS server sits on a separate ISP LAN 198.51.100.4/30 (ISP 198.51.100.5, DNS 198.51.100.6). No internal 10.0.0.0/16 address is used on this link

### 3.2 Subnets – VLSM Table (Sorted by Host Requirement, Descending)

11 LAN/VLAN subnets with six different sizes (/23, /24, /25, /26, /27, /28) plus /30 for point-to-point links. Table below lists required hosts, allocated block size, subnet mask, resulting Network ID and broadcast address for each.

| Sn | Network | Hosts Req. | Hosts Alloc. | Subnet Mask | Network ID (CIDR) | Broadcast |
|----|---------|-----------|--------------|-------------|-------------------|-----------|
| 1 | Production Floor | 500 | 510 | 255.255.254.0 | 10.0.0.0/23 | 10.0.1.255 |
| 2 | R&D Lab | 250 | 254 | 255.255.255.0 | 10.0.2.0/24 | 10.0.2.255 |
| 3 | Warehouse & Logistics | 200 | 254 | 255.255.255.0 | 10.0.3.0/24 | 10.0.3.255 |
| 4 | Branch Office | 150 | 254 | 255.255.255.0 | 10.0.4.0/24 | 10.0.4.255 |
| 5 | Sales | 100 | 126 | 255.255.255.128 | 10.0.5.0/25 | 10.0.5.127 |
| 6 | Marketing | 80 | 126 | 255.255.255.128 | 10.0.5.128/25 | 10.0.5.255 |
| 7 | Data Center / Server LAN | 60 | 62 | 255.255.255.192 | 10.0.6.0/26 | 10.0.6.63 |
| 8 | Guest VLAN (Area 0) | 40 | 62 | 255.255.255.192 | 10.0.6.64/26 | 10.0.6.127 |
| 9 | Finance VLAN | 30 | 30 | 255.255.255.224 | 10.0.6.128/27 | 10.0.6.159 |
| 10 | HR VLAN | 25 | 30 | 255.255.255.224 | 10.0.6.160/27 | 10.0.6.191 |
| 11 | Legal VLAN | 10 | 14 | 255.255.255.240 | 10.0.6.192/28 | 10.0.6.207 |

### 3.3 Point-to-Point Router Links

14 inter-router links (P01–P14) including 1 ISP link, each using a /30 subnet (2 usable host addresses). Redundant links (core loop, dual-homed distributions) exist to eliminate the chance of disconnectivity in case of link failure.

| Link | Network ID | Usable Host IPs | Purpose |
|------|------------|-----------------|---------|
| P01 | 198.51.100.0/30 | 198.51.100.1, 198.51.100.2 | BR to ISP (public WAN link) |
| P02 | 10.0.12.0/30 | 10.0.12.1, 10.0.12.2 | BR to CR1 |
| P03 | 10.0.12.4/30 | 10.0.12.5, 10.0.12.6 | BR to CR2 |
| P04 | 10.0.12.8/30 | 10.0.12.9, 10.0.12.10 | CR1 to CR2 (core loop) |
| P05 | 10.0.12.12/30 | 10.0.12.13, 10.0.12.14 | CR1 to R11 (Data Center) |
| P06 | 10.0.12.16/30 | 10.0.12.17, 10.0.12.18 | CR2 to R11 (Data Center redundant) |
| P07 | 10.0.12.20/30 | 10.0.12.21, 10.0.12.22 | CR1 to R5 (R&D) |
| P08 | 10.0.12.24/30 | 10.0.12.25, 10.0.12.26 | CR2 to R5 (R&D redundant) |
| P09 | 10.0.12.28/30 | 10.0.12.29, 10.0.12.30 | CR1 to R4 (HQ) |
| P10 | 10.0.12.32/30 | 10.0.12.33, 10.0.12.34 | CR2 to R4 (HQ redundant) |
| P11 | 10.0.12.36/30 | 10.0.12.37, 10.0.12.38 | R4 to R8 (Sales/Marketing) |
| P12 | 10.0.12.40/30 | 10.0.12.41, 10.0.12.42 | CR2 to R6 (Production) |
| P13 | 10.0.12.44/30 | 10.0.12.45, 10.0.12.46 | R6 to R7 (Plant backbone) |
| P14 | 10.0.12.48/30 | 10.0.12.49, 10.0.12.50 | R7 to R9 (Branch) |

### 3.4 Server IP Addresses

| Server | IP Address | Subnet | Description |
|--------|------------|--------|-------------|
| Web Server 1 (public) | 10.0.6.10 | 10.0.6.0/26 (Data Center, Area 0) | Corporate public website / customer portal; DMZ in the backbone for shortest average path from all areas |
| DNS Server 1 (primary) | 10.0.6.11 | 10.0.6.0/26 (Data Center, Area 0) | Caching + authoritative resolution of corporate hostnames (e.g. www.ktmtechnologies.com); forwards external queries upstream |
| DHCP Server (DC) | 10.0.6.2 | 10.0.6.0/26 (Data Center, Area 0) | Issues dynamic IPs to end-hosts on the Data Center Server LAN |
| DNS Server 2 (secondary) | 10.0.4.10 | 10.0.4.0/24 (Branch, Area 2) | Secondary caching DNS in a different LAN |
| Web Server 2 (intranet) | 10.0.2.10 | 10.0.2.0/24 (R&D, Area 2) | Internal ERP/intranet portal on a different subnet from Web Server 1 |
| ISP DNS Server | 198.51.100.6 | ISP LAN 198.51.100.4/30 (public) | Extra-level DNS in upstream ISP that resolves web addresses into IPs |

**Additional notes:**

- DHCP Servers on each LAN will be assigned the 2nd usable address on their respective subnets (1st will be used as Default Gateway for simplicity in configuration). DHCP broadcasts do not cross routers, so each routed LAN needs its own server (or a relay):
  - Area 0: DHCP-DC on SW-DC (10.0.6.2), DHCP-GUEST on SW-GUEST (10.0.6.66)
  - Area 1: DHCP-HQ on SW-FIN (10.0.6.130) — one server, three DHCP pools, one per HQ VLAN (Finance/HR/Legal); R4 relays the other VLANs' requests to it via `ip helper-address`. DHCP-SALES on SW-SALES (10.0.5.2), DHCP-MKT on SW-MKT (10.0.5.130)
  - Area 2: DHCP-RD on SW-RD (10.0.2.2), DHCP-PROD on SW-PROD (10.0.0.2), DHCP-WH on SW-WH (10.0.3.2), DHCP-BRANCH on SW-BRANCH (10.0.4.2)

### 3.5 Additional Specifications

- **Addressing scheme:** VLSM on 10.0.0.0/16, subnets sorted and allocated largest-host-count first to minimize fragmentation
- **Routing protocol:** OSPFv2, multi-area (Area 0 backbone + Areas 1–2), with Area Border Routers (ABRs) CR1 and CR2 connecting each area to the backbone; BR advertises the default route toward the ISP
- **Internet path:** All Internet traffic forwarded to the upstream ISP via BR (default route); ISP returns packets using a static route (no dynamic routing with the ISP)
- **VLANs:** Inter-VLAN routing performed via Router-on-a-Stick on R4 (Finance, HR, Legal); a separate Guest VLAN is isolated in Area 0
- **DHCP:** One DHCP server per LAN (2nd usable address); HQ VLANs share a single server with three pools
- **DNS:** Internal DNS servers resolve corporate hostnames; configured to forward unresolved external queries to the ISP's/public DNS via the ISP link
- **Security:** Creation of Guest VLAN, departmental VLANs and DMZ separate the broadcast domains
- **Scalability:** ~3,000+ addresses (10.0.7.0/24 onward) remain unused for future plants, branches or server expansion

### 3.6 Requirements Compliance Summary

| Specification Requirement | Design Implementation |
|---------------------------|-----------------------|
| ≥ /22 address block | 10.0.0.0/16 |
| ≥ 9 networks, ≥ 6 sizes | 11 LAN/VLAN subnets with 6 different sizes (/23, /24, /25, /26, /27, /28) + /30 P2P |
| ≥ 3 VLANs over ≥ 3 switches | Finance/HR/Legal VLANs trunked across 3 HQ access switches to R4 |
| ≥ 9 routers, 3 without LAN | 10 routers; BR, CR1, CR2 have no LAN connectivity |
| OSPF with ≥ 3 areas + backbone | Areas 0, 1, 2 |
| OSPF to ISP / static return | BR public WAN link to ISP (P01, 198.51.100.0/30) → default route; static from ISP back, no dynamic routing |
| ≥ 2 DNS in different LANs | DNS1 (DC, 10.0.6.0/26) + DNS2 (Branch, 10.0.4.0/24) + ISP DNS |
| ≥ 2 Web servers on different subnets | Web1 10.0.6.10 (DC, 10.0.6.0/26), Web2 10.0.2.10 (R&D, 10.0.2.0/24) |
| Extra-level DNS in ISP network | ISP DNS 198.51.100.6 (ISP LAN 198.51.100.4/30) |
| Multiple paths in ≥ 2 networks | Dual-homed DC (P05/P06), R&D (P07/P08), HQ (P09/P10), core loop (P04) |

---

## 4. Conclusion

This proposal outlines a structured, VLSM-based IP addressing plan and a 3-area OSPF topology for the corporate network, covering the HQ, R&D, production, warehouse, one branch office, VLANs, servers, DMZ and 14 inter-router links. The design balances efficient address utilization with room for future growth and provides redundancy through a dual-homed redundant core, while still meeting every minimum requirement. It is ready for topology diagram integration and subsequent Cisco Packet Tracer implementation.
