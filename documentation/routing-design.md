# Routing Design

## Overview

The lab uses OSPF process 1 in Area 0 to provide dynamic routing between the headquarters and Branch 1.

The design intentionally separates Layer 2 user access from Layer 3 routed transit links. Multilayer switches perform inter-VLAN routing, while routers and routed switch interfaces provide site-to-site connectivity.

## OSPF Router IDs

| Device | Router ID |
|---|---|
| HQ-CORE1 | `1.1.1.1` |
| HQ-CORE2 | `2.2.2.2` |
| HQ-R2 | `10.10.10.10` |
| BR1-R1 | `20.20.20.20` |
| BR1-CORE1 | `21.21.21.21` |

Unique router IDs are critical. During troubleshooting, a duplicate/stale router ID was identified as a potential source of routing instability and was corrected.

## HQ Routed Links

### HQ-R2 ↔ HQ-CORE1

- HQ-R2 Gi0/0: `10.10.254.1/30`
- HQ-CORE1 Gi0/1: `10.10.254.2/30`

### HQ-R2 ↔ HQ-CORE2

- HQ-R2 Gi0/1: `10.10.254.5/30`
- HQ-CORE2 Gi0/1: `10.10.254.6/30`

These interfaces are configured as routed ports on the multilayer switches using `no switchport`.

## Branch 1 Routed Links

### HQ-R2 ↔ BR1-R1

- HQ-R2 Gi0/2: `10.255.1.1/30`
- BR1-R1 Gi0/0: `10.255.1.2/30`

### BR1-R1 ↔ BR1-CORE1

- BR1-R1 Gi0/1: `10.20.254.1/30`
- BR1-CORE1 Gi0/1: `10.20.254.2/30`

## Passive Interface Strategy

User-facing SVIs are advertised into OSPF but should not form neighbor adjacencies.

Example core configuration:

```text
router ospf 1
 passive-interface default
 no passive-interface gigabitEthernet0/1
```

This keeps OSPF hellos off departmental VLANs while still advertising the networks.

## Route Verification

Commands used to validate OSPF included:

```text
show ip ospf neighbor
show ip route ospf
show ip protocols
show ip ospf database
show ip ospf interface brief
```

Expected routing behavior on HQ-R2 includes both headquarters and Branch 1 routes.

Example learned networks:

```text
10.10.20.0/24
10.10.30.0/24
10.10.40.0/24
10.10.50.0/24
10.10.60.0/24
10.10.70.0/24
10.10.80.0/24

10.20.20.0/24
10.20.30.0/24
10.20.40.0/24
10.20.50.0/24
10.20.60.0/24
10.20.70.0/24
```

## Troubleshooting Example

At one stage, HQ-R2 had valid FULL OSPF adjacencies and the router LSA for HQ-CORE1 was present in the LSDB, but the HQ VLAN routes were not installed in the routing table.

A controlled reset of the OSPF process on HQ-R2 caused SPF recalculation and restored the HQ routes.

This demonstrated an important operational lesson: adjacency state alone is not enough. A network engineer must also verify the LSDB and final routing table.
