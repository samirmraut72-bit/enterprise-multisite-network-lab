# Troubleshooting Log

This project was deliberately treated like a real implementation: faults were isolated by testing each layer instead of repeatedly changing unrelated configuration.

## 1. HQ OSPF Routes Missing from HQ-R2

### Symptom
HQ-R2 had FULL OSPF adjacencies with both HQ core switches and Branch 1, but HQ VLAN routes were absent from `show ip route ospf`.

### Checks

```text
show ip ospf neighbor
show ip route ospf
show ip route connected
show ip ospf database
```

### Finding
HQ-CORE1 had all HQ VLANs connected and its Router LSA was visible on HQ-R2, proving that adjacency and LSA propagation were working.

### Resolution
A controlled OSPF process reset on HQ-R2 forced SPF recalculation. The HQ routes were then installed correctly.

## 2. Stale HSRP Configuration on Branch 1

### Symptom
Branch-to-HQ connectivity and DHCP behaved intermittently.

### Finding
BR1-CORE1 still contained HSRP virtual IP addresses copied from HQ, for example:

```text
standby 30 ip 10.10.30.1
standby 40 ip 10.10.40.1
standby 50 ip 10.10.50.1
```

These addresses belonged to HQ, not Branch 1.

### Resolution
Removed all stale Branch HSRP statements. Branch 1 currently has a single core switch, so the branch SVIs use only their real `10.20.x.1` gateway addresses.

## 3. Duplicate Gateway Addresses on BR1-ACCESS

### Symptom
Pings were inconsistent and Branch clients could not reliably reach HQ.

### Finding
The Layer 2 Branch access switch incorrectly had Layer 3 SVI addresses such as:

```text
Vlan30 10.20.30.1
Vlan40 10.20.40.1
Vlan50 10.20.50.1
Vlan60 10.20.60.1
Vlan70 10.20.70.1
```

The same gateway addresses correctly existed on BR1-CORE1, creating duplicate IP ownership.

### Resolution
Removed the SVI IP addresses and shut the Layer 3 SVIs on BR1-ACCESS. The access switch now operates at Layer 2 while BR1-CORE1 owns the gateways.

## 4. Incorrect Branch DHCP Pool Addresses

### Symptom
Branch PCs failed to obtain leases even though DHCP relay was configured.

### Finding
The Branch DHCP pools had correct Branch gateways but HQ start addresses. Example:

```text
BR1-IT gateway: 10.20.20.1
Incorrect start: 10.10.20.20
```

### Resolution
Corrected Branch start addresses to the relevant Branch networks, for example:

```text
BR1-IT start: 10.20.20.20
```

## 5. Port Security Violations

### Symptom
BR1-ACCESS reported repeated security violations on Fa0/1.

### Finding
The port already contained two sticky MAC addresses and the violation count had increased.

### Diagnostic Approach
Port security was temporarily removed from the single affected port to determine whether it was responsible for DHCP failure. DHCP still failed, proving the main fault was elsewhere.

### Lesson
Do not assume the first visible error is the root cause. Isolate one variable at a time.

## 6. Routed Interface IP Processing Issue

### Symptom
HQ-CORE2 could reach its directly connected HQ-R2 next hop but remote communication was inconsistent.

### Finding
`show ip interface gigabitEthernet0/1` reported:

```text
Internet protocol processing disabled
```

### Resolution
Re-applied the routed interface configuration:

```text
interface gigabitEthernet0/1
 no switchport
 ip address 10.10.254.6 255.255.255.252
 ip ospf network point-to-point
 no shutdown
```

## Troubleshooting Method Used

The general process used throughout the lab was:

1. Confirm physical/interface status.
2. Confirm VLAN membership and trunk forwarding.
3. Confirm the correct device owns each gateway IP.
4. Confirm local connected routes.
5. Confirm OSPF neighbor state.
6. Confirm OSPF routes in the routing table.
7. Test each routed hop with ping.
8. Confirm return-path routing.
9. Check ACLs and Layer 2 security controls.
10. Investigate application services such as DHCP only after network reachability is proven.

This approach turned the lab into a troubleshooting exercise rather than only a configuration exercise.
