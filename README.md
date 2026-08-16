# Enterprise Multi-Site Network Lab

A practical Cisco enterprise networking project designed and implemented in Cisco Packet Tracer to simulate a headquarters and branch-office environment with segmentation, redundancy, dynamic routing, centralized services, and Layer 2 security controls.

## Project Objective

The goal of this lab was to design a realistic multi-site enterprise network rather than a simple classroom topology. The environment was built to demonstrate how separate business departments, voice devices, CCTV systems, servers, redundant core infrastructure, and remote branches can communicate securely across a routed WAN.

## Architecture

### Headquarters
- Dual Cisco multilayer core switches
- Redundant HSRP default gateways
- LACP EtherChannel between core switches
- Department access switches for IT, Finance, HR, and Staff
- Dedicated server access switch
- Central DHCP, DNS, Web, File, AAA, Syslog, and NVR services
- Voice and CCTV VLANs

### Branch 1
- Cisco branch router
- Multilayer core switch
- Layer 2 access switch
- Departmental VLANs for IT, Finance, HR, and Staff
- Voice and CCTV segmentation
- Centralized DHCP delivered from HQ using DHCP relay
- OSPF routing across the WAN

## Addressing Strategy

| Site | Address Space |
|---|---|
| Headquarters | `10.10.0.0/16` |
| Branch 1 | `10.20.0.0/16` |
| WAN / Transit | `10.255.0.0/16` |

### HQ VLANs

| VLAN | Purpose | Subnet | Gateway |
|---|---|---|---|
| 20 | IT | `10.10.20.0/24` | `10.10.20.1` |
| 30 | Finance | `10.10.30.0/24` | `10.10.30.1` |
| 40 | HR | `10.10.40.0/24` | `10.10.40.1` |
| 50 | Staff | `10.10.50.0/24` | `10.10.50.1` |
| 60 | Voice | `10.10.60.0/24` | `10.10.60.1` |
| 70 | CCTV | `10.10.70.0/24` | `10.10.70.1` |
| 80 | Servers | `10.10.80.0/24` | `10.10.80.1` |
| 999 | Native / Blackhole | N/A | N/A |

### Branch 1 VLANs

| VLAN | Purpose | Subnet | Gateway |
|---|---|---|---|
| 20 | IT | `10.20.20.0/24` | `10.20.20.1` |
| 30 | Finance | `10.20.30.0/24` | `10.20.30.1` |
| 40 | HR | `10.20.40.0/24` | `10.20.40.1` |
| 50 | Staff | `10.20.50.0/24` | `10.20.50.1` |
| 60 | Voice | `10.20.60.0/24` | `10.20.60.1` |
| 70 | CCTV | `10.20.70.0/24` | `10.20.70.1` |

## Technologies Implemented

- VLAN segmentation
- 802.1Q trunking
- Inter-VLAN routing on multilayer switches
- HSRP gateway redundancy at HQ
- LACP EtherChannel between HQ cores
- OSPF Area 0 dynamic routing
- Routed point-to-point links
- Centralized DHCP
- DHCP relay using `ip helper-address`
- Access Control Lists
- Port Security with sticky MAC learning
- DHCP Snooping
- Dynamic ARP Inspection
- PortFast and BPDU Guard
- Voice VLANs
- CCTV isolation
- Native VLAN blackholing

## Central Services

| Service | Address |
|---|---|
| DHCP | `10.10.80.10` |
| DNS | `10.10.80.11` |
| Web | `10.10.80.12` |
| File | `10.10.80.13` |
| AAA | `10.10.80.14` |
| Syslog | `10.10.80.15` |
| NVR | `10.10.80.20` |

## Security Design

The network uses segmentation and Layer 2 protections rather than placing all endpoints into a flat LAN.

Examples include:

- CCTV devices are restricted from accessing sensitive departmental networks.
- Staff, HR, and Finance access is controlled using extended ACLs.
- IT remains the privileged administrative network.
- Access ports use PortFast and BPDU Guard.
- User-facing ports use sticky MAC port security.
- DHCP Snooping protects against unauthorized DHCP servers.
- Dynamic ARP Inspection helps mitigate ARP spoofing attacks.
- VLAN 999 is used as an unused native/blackhole VLAN.

## Troubleshooting Case Study

A major part of this project involved identifying and resolving real configuration faults rather than simply entering known-good commands.

Issues diagnosed during implementation included:

1. OSPF routes present in the LSDB but temporarily absent from the routing table.
2. Duplicate and stale HSRP configuration copied from HQ into Branch 1.
3. Duplicate Layer 3 gateway IP addresses accidentally configured on the Branch access switch.
4. Incorrect Branch DHCP pool start addresses using HQ subnets.
5. Port-security violations caused by sticky MAC entries.
6. A routed multilayer-switch uplink temporarily showing IP processing disabled.
7. Intermittent connectivity caused by conflicting gateway configuration.

The faults were isolated using commands such as:

```text
show ip route
show ip route ospf
show ip ospf neighbor
show ip ospf database
show ip interface brief
show interfaces trunk
show vlan brief
show standby brief
show port-security interface
show ip dhcp snooping
ping
```

This troubleshooting process reinforced a structured approach of testing Layer 1/2 connectivity, validating gateways and VLANs, checking route propagation, verifying return paths, and only then investigating application services such as DHCP.

## Verification

Successful testing included:

- Department endpoints receiving addresses through DHCP
- Branch clients receiving centrally assigned DHCP leases from HQ
- Inter-VLAN routing at HQ and Branch 1
- OSPF adjacency between HQ and Branch routers
- Branch-to-HQ connectivity across the WAN
- Access to centralized HQ services from Branch 1
- HSRP operation between HQ core switches
- LACP EtherChannel operation between HQ cores
- Security controls active on access switches

## Repository Structure

```text
enterprise-multisite-network-lab/
├── README.md
├── topology/
├── configs/
├── documentation/
│   ├── ip-addressing.md
│   ├── routing-design.md
│   ├── security-controls.md
│   └── troubleshooting.md
└── verification/
```

## Next Development Phase

Planned improvements include additional branch sites, network automation using Ansible, validation using pyATS, migration of the design into Cisco Modeling Labs, and eventually applying the same configuration-as-code workflow to physical Cisco equipment.

## Skills Demonstrated

Cisco IOS · Enterprise Networking · VLANs · Layer 2 Switching · Layer 3 Switching · OSPF · HSRP · EtherChannel · DHCP · ACLs · Port Security · DHCP Snooping · Dynamic ARP Inspection · Network Troubleshooting · WAN Design

---

**Portfolio project by Sameer Raut**
