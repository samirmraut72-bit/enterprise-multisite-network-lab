# Security Controls

## Segmentation

The network separates departments and device classes into dedicated VLANs rather than using a flat Layer 2 design.

Primary segments include IT, Finance, HR, Staff, Voice, CCTV, and Servers.

## Extended ACLs

ACLs are applied at HQ to restrict lateral movement between business units while preserving access to required centralized services.

Examples of policy intent:

- CCTV is restricted from sensitive departmental networks.
- Staff has limited access to selected server resources.
- Finance and HR are isolated from each other where not required.
- IT remains the privileged administrative network.

## Port Security

User-facing access ports use sticky MAC learning and restrict mode.

Example policy:

```text
switchport port-security
switchport port-security maximum 2
switchport port-security violation restrict
switchport port-security mac-address sticky
```

Ports supporting a phone and PC allow two secure MAC addresses.

## DHCP Snooping

DHCP Snooping is enabled on user, voice, and CCTV VLANs. Only trusted uplinks are permitted to forward DHCP server responses.

This helps prevent rogue DHCP servers from issuing malicious gateway or DNS information.

## Dynamic ARP Inspection

DAI is enabled alongside DHCP Snooping to validate ARP messages against trusted bindings and reduce exposure to ARP spoofing attacks.

## Spanning Tree Edge Protection

Endpoint-facing switchports use:

```text
spanning-tree portfast
spanning-tree bpduguard enable
```

This allows endpoints to transition rapidly to forwarding while protecting against accidental or malicious switch connections.

## Native VLAN Hardening

VLAN 999 is used as an unused native/blackhole VLAN on trunks instead of relying on VLAN 1 for production traffic.

## Operational Lesson

Security controls can also create outages when incorrectly configured. During the lab, a port-security violation on a Branch IT port was investigated by checking secure MAC counts and violation counters before temporarily removing port security to isolate the DHCP fault.

This reinforced the need to distinguish between a security feature working as designed and an unrelated Layer 3 or service issue.
