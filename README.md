# CCNA Portfolio Lab: Two-Site Enterprise Network

## 1. Scenario
Designed and deployed a multi-site enterprise network simulation representing a primary Headquarters (HQ) and a remote Branch office connected via a dedicated WAN link. The design evaluates multi-tier L3 switching vs. router-based inter-VLAN routing trade-offs, dynamic link-state routing convergence, granular security segmentation via access control lists, and automated dynamic host configuration.

## 2. Topology Diagram
![Network Topology](screenshots/topology)

## 3. IP Addressing Plan
| Segment | Network / Mask | Gateway | Description |
| :--- | :--- | :--- | :--- |
| HQ VLAN 10 – Management | `10.10.10.0/24` | `10.10.10.1` | Administrative traffic (`SW-HQ` SVI) |
| HQ VLAN 20 – Sales | `10.10.20.0/24` | `10.10.20.1` | Sales department endpoints (`SW-HQ` SVI) |
| HQ VLAN 30 – IT/Servers | `10.10.30.0/24` | `10.10.30.1` | Internal server infrastructure (`SW-HQ` SVI) |
| Branch VLAN 10 – Management | `10.20.10.0/24` | `10.20.10.1` | Branch admin (`R-BRANCH` sub-interface) |
| Branch VLAN 20 – Sales | `10.20.20.0/24` | `10.20.20.1` | Branch sales (`R-BRANCH` sub-interface) |
| WAN Link (R-HQ ↔ R-BRANCH) | `10.10.99.0/30` | `.1` (HQ) / `.2` (Branch) | Inter-site core WAN connection |
| Transit Link (R-HQ ↔ SW-HQ) | `10.10.88.0/30` | `.1` (R-HQ) / `.2` (SW-HQ) | Core-to-distribution dynamic routing link |

## 4. Architectural Rationale & Configuration Decisions
- **Inter-VLAN Routing Trade-offs**:
  - *HQ (L3 Switch + SVIs)*: Chosen for core scale and wire-speed hardware-based forwarding across local subnets.
  - *Branch (Router-on-a-Stick)*: Chosen for cost efficiency at smaller remote sites, accepting single-port bottleneck trade-offs.
- **Dynamic Routing (OSPF Area 0)**: Single-area OSPF enabled across `R-HQ`, `SW-HQ`, and `R-BRANCH` utilizing a dedicated `/30` transit block (`10.10.88.0/30`), eliminating static routing brittleness.
- **DHCP Provisioning**: Scopes configured on respective gateway devices (`SW-HQ` for HQ VLANs, `R-BRANCH` for Branch VLANs) with excluded gateway/static server ranges.
- **Security Segmentation**: Extended ACL (`BLOCK_SALES_TO_IT`) applied to restrict Sales-to-IT server lateral movement while preserving general reachability.

## 5. Verification & Proofs
| Test Case | Target / Device | Reference Artifact |
| :--- | :--- | :--- |
| VLAN Brief (HQ) | `SW-HQ` | `screenshots/vlan_brief_sw-hq` |
| VLAN Brief (Branch) | `SW-BRANCH` | `screenshots/vlan_brief_sw-branch` |
| OSPF Neighbors | `R-HQ`, `SW-HQ`, `R-BRANCH` | `screenshots/ospf_neighbor_*` |
| OSPF Routing Tables | `R-HQ`, `SW-HQ`, `R-BRANCH` | `screenshots/ospf_routes_*` |
| Cross-Site Reachability | Branch Sales → HQ Mgmt | `screenshots/ping_cross_site` |
| ACL Block (Compliance) | Sales → Server (`10.10.30.10`) | `screenshots/ping_acl_blocked` |
| ACL Permit (Exception) | Mgmt → Server (`10.10.30.10`) | `screenshots/ping_acl_permitted` |
| Server Static IP Setup | `SRV-IT` | `screenshots/acl_server` |
| Client DHCP Lease | End-user PC | `screenshots/ipconfig_dhcp` |

## 6. Interview Talking Points
- **Why two inter-VLAN routing methods?** Router-on-a-stick is cost-effective for small footprint branch offices, whereas L3 switches provide wire-speed multi-layer switching scalability for core enterprise sites.
- **Why OSPF over static routes?** Automatic link-state convergence, scalable metric calculation, and simplified multi-site management. (*Note: Single-link WAN topology introduces a Single Point of Failure, which can be mitigated via redundant WAN links*).
- **ACL logic & ordering**: Explicit source/destination wildcard matching evaluated top-down with an explicit/implicit catch-all.
- **Production Next-Steps**: Implementing NAT/PAT for outbound internet, redundant WAN paths (dual-homed), and HSRP/VRRP gateway high availability.