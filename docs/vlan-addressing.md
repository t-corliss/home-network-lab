# VLAN and Addressing Plan

## Purpose

This document defines the active VLANs, IPv4 subnets, gateway addresses, DHCP behavior, infrastructure addressing, and related addressing conventions used in the home network lab.

The goal is to maintain one authoritative reference for network segmentation and addressing without duplicating firewall policy or switch configuration details.

## VLAN Summary

| VLAN | Name | Subnet | Gateway | Purpose |
|---|---|---|---|---|
| 1 | Legacy / Recovery | 192.168.1.0/24 | 192.168.1.1 | Emergency and break-glass recovery |
| 10 | Management | 10.10.10.0/24 | 10.10.10.1 | Infrastructure administration |
| 20 | Trusted | 10.10.20.0/24 | 10.10.20.1 | Trusted client devices |
| 30 | Servers | 10.10.30.0/24 | 10.10.30.1 | Server workloads |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 | Isolated guest access |

## VLAN 1 - Legacy / Recovery

VLAN 1 uses the subnet `192.168.1.0/24` with pfSense at `192.168.1.1`.

This VLAN is retained as a privileged recovery path and should not be treated as a normal client network.

Its purpose is to provide break-glass access when other management or VLAN paths are unavailable.

Because this network is intentionally more permissive than the production segments, access to it should remain limited to recovery use.

## VLAN 10 - Management

VLAN 10 uses the subnet `10.10.10.0/24` with pfSense at `10.10.10.1`.

This VLAN is used for infrastructure administration.

Devices associated with this network include management interfaces for systems such as:

- pfSense
- Netgear managed switch
- UniFi Cloud Key Plus
- UniFi access point

Administrative access to this VLAN is intentionally restricted and is not available equally to all client devices.

## VLAN 20 - Trusted

VLAN 20 uses the subnet `10.10.20.0/24` with pfSense at `10.10.20.1`.

This VLAN is used for trusted client devices, including normal household clients and designated administrative systems.

Some administrative clients use known addresses within this VLAN so that pfSense can apply more specific routed access policy when those devices communicate with other network segments.

A known source IP address should not be treated as strong identity by itself.

## VLAN 30 - Servers

VLAN 30 uses the subnet `10.10.30.0/24` with pfSense at `10.10.30.1`.

This VLAN is dedicated to server workloads.

The Windows Server 2022 system resides on this network, allowing access to server services to be controlled independently from trusted clients, guest devices, and infrastructure management systems.

Separating servers into their own VLAN reduces the amount of implicit trust between client and server systems.

## VLAN 40 - Guest

VLAN 40 uses the subnet `10.10.40.0/24` with pfSense at `10.10.40.1`.

This VLAN is used for isolated guest access.

Guest clients are intended to receive Internet access without receiving general access to trusted clients, servers, management systems, or recovery networks.

Wireless clients on this VLAN also use UniFi client isolation to reduce direct communication between guest devices on the same Layer 2 network.

## DHCP Ranges

pfSense provides DHCP service for the active VLANs.

The current dynamic address pools are:

| VLAN | Network | DHCP Range |
|---|---|---|
| 1 - Legacy / Recovery | `192.168.1.0/24` | `192.168.1.100` - `192.168.1.199` |
| 10 - Management | `10.10.10.0/24` | `10.10.10.100` - `10.10.10.199` |
| 20 - Trusted | `10.10.20.0/24` | `10.10.20.100` - `10.10.20.199` |
| 30 - Servers | `10.10.30.0/24` | `10.10.30.100` - `10.10.30.199` |
| 40 - Guest | `10.10.40.0/24` | `10.10.40.100` - `10.10.40.199` |

# Static Mappings and Reservations

Some systems use DHCP reservations so that they receive predictable addresses while still obtaining their configuration through DHCP.

Known administrative clients use reserved addresses where stable source addressing is needed for firewall policy.

Examples include administrative client reservations such as:

- Trusted VLAN administrative clients at `10.10.20.10` and `10.10.20.11`
- Management recovery clients at `10.10.10.10` and `10.10.10.11`

A DHCP reservation improves address consistency but should not be treated as strong authentication or proof of device identity.

## Infrastructure Addresses

Infrastructure devices should use stable addressing when firewall rules, management access, or service dependencies rely on their IP addresses.

Current known addresses include:

| Device | VLAN | Address | Status |
|---|---|---|---|
| pfSense Management gateway | 10 | `10.10.10.1` | Configured |
| pfSense Trusted gateway | 20 | `10.10.20.1` | Configured |
| pfSense Servers gateway | 30 | `10.10.30.1` | Configured |
| pfSense Guest gateway | 40 | `10.10.40.1` | Configured |
| Windows Server 2022 | 30 | `10.10.30.11` | Known / stable |
| UniFi Cloud Key Plus | 10 | `10.10.10.20` | Configured |
| UniFi AC Mesh AP | 10 | `10.10.10.21` | Configured |
| Netgear JGS516PE | 10 | `10.10.10.22` | Configured |

## Addressing Conventions

The addressing plan follows several conventions intended to make the network easier to understand and maintain.

- pfSense uses the `.1` address as the default gateway on each active VLAN.
- Dynamic DHCP pools use the `.100` through `.199` range.
- Lower addresses are available for infrastructure systems, reservations, and other predictable assignments.
- Administrative clients may use reserved addresses when firewall policy depends on stable source addressing.
- Infrastructure addresses should not be treated as permanent unless they are explicitly configured or reserved.
- Addressing documentation should distinguish configured values from addresses that were only observed during validation.

## IPv6 Status

The current documented segmentation and firewall model is based on IPv4.

IPv6 should not be enabled for routed client traffic until equivalent addressing, routing, firewall, and validation policy has been defined.

If IPv6 is introduced later, the security assumptions documented for IPv4 should be reviewed rather than assumed to apply automatically.