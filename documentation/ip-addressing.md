# IP Addressing Plan

## Site Allocation

| Site | Summary Block |
|---|---|
| Headquarters | `10.10.0.0/16` |
| Branch 1 | `10.20.0.0/16` |
| Future Branch 2 | `10.30.0.0/16` |
| Future Branch 3 | `10.40.0.0/16` |
| WAN / Transit | `10.255.0.0/16` |

## Headquarters VLANs

| VLAN | Name | Network | Default Gateway |
|---|---|---|---|
| 20 | IT | `10.10.20.0/24` | `10.10.20.1` |
| 30 | FINANCE | `10.10.30.0/24` | `10.10.30.1` |
| 40 | HR | `10.10.40.0/24` | `10.10.40.1` |
| 50 | STAFF | `10.10.50.0/24` | `10.10.50.1` |
| 60 | VOICE | `10.10.60.0/24` | `10.10.60.1` |
| 70 | CCTV | `10.10.70.0/24` | `10.10.70.1` |
| 80 | SERVERS | `10.10.80.0/24` | `10.10.80.1` |
| 999 | NATIVE-BLACKHOLE | N/A | N/A |

HQ HSRP addressing convention:

- CORE1 SVI: `.2`
- CORE2 SVI: `.3`
- HSRP virtual gateway: `.1`

Example for VLAN 20:

- HQ-CORE1: `10.10.20.2/24`
- HQ-CORE2: `10.10.20.3/24`
- Virtual gateway: `10.10.20.1`

## Headquarters Server VLAN

| Server | IP |
|---|---|
| DHCP | `10.10.80.10` |
| DNS | `10.10.80.11` |
| WEB | `10.10.80.12` |
| FILE | `10.10.80.13` |
| AAA | `10.10.80.14` |
| SYSLOG | `10.10.80.15` |
| NVR | `10.10.80.20` |

Default gateway: `10.10.80.1`

## Branch 1 VLANs

| VLAN | Name | Network | Default Gateway |
|---|---|---|---|
| 20 | IT | `10.20.20.0/24` | `10.20.20.1` |
| 30 | FINANCE | `10.20.30.0/24` | `10.20.30.1` |
| 40 | HR | `10.20.40.0/24` | `10.20.40.1` |
| 50 | STAFF | `10.20.50.0/24` | `10.20.50.1` |
| 60 | VOICE | `10.20.60.0/24` | `10.20.60.1` |
| 70 | CCTV | `10.20.70.0/24` | `10.20.70.1` |

Branch 1 uses its multilayer core switch as the Layer 3 gateway. No HSRP is currently required at the branch because only one branch core switch is deployed.

## Routed Transit Networks

| Link | Network | Endpoints |
|---|---|---|
| HQ-R2 ↔ HQ-CORE1 | `10.10.254.0/30` | `10.10.254.1` / `10.10.254.2` |
| HQ-R2 ↔ HQ-CORE2 | `10.10.254.4/30` | `10.10.254.5` / `10.10.254.6` |
| BR1-R1 ↔ BR1-CORE1 | `10.20.254.0/30` | `10.20.254.1` / `10.20.254.2` |
| HQ-R2 ↔ BR1-R1 | `10.255.1.0/30` | `10.255.1.1` / `10.255.1.2` |

## DHCP Scope Strategy

Central DHCP is hosted at HQ on `10.10.80.10`.

Branch SVIs use:

```text
ip helper-address 10.10.80.10
```

Example Branch 1 IT scope:

- Gateway: `10.20.20.1`
- DNS: `10.10.80.11`
- Start address: `10.20.20.20`
- Mask: `255.255.255.0`

This keeps address assignment centralized while preserving Layer 3 segmentation between sites and VLANs.
