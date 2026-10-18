# Network Architecture

## Purpose

This document describes the overall design of the home network lab, including device roles, physical and logical topology, Layer 2 and Layer 3 responsibilities, traffic flow, administrative access, recovery paths, and important design constraints.

## Design Principles

- Separate devices and services into distinct security zones.
- Use VLANs to provide Layer 2 segmentation.
- Use pfSense to route between VLANs and enforce Layer 3 firewall policy.
- Apply least-privilege access between network segments.
- Preserve dedicated recovery paths for management access.
- Validate changes one layer at a time before removing known-good access.

## Device Roles

### Netgate SG-1100 / pfSense

The Netgate SG-1100 serves as the network's Layer 3 gateway and firewall.

Its primary responsibilities include:

- Routing traffic between VLANs.
- Enforcing firewall policy between security zones.
- Providing DHCP and DNS services for the network.
- Providing Internet access through the WAN connection.
- Hosting administrative access to pfSense.
- Supporting recovery access through dedicated management paths.

### Netgear JGS516PE Managed Switch

The Netgear JGS516PE provides Layer 2 switching and VLAN transport between network devices.

Its primary responsibilities include:

- Carrying tagged VLAN traffic between pfSense and downstream devices.
- Providing untagged access ports for individual VLANs.
- Providing hybrid/trunk behavior where required.
- Connecting infrastructure, servers, recovery ports, and the wireless access point.
- Preserving Layer 2 separation between VLANs.

### UniFi Cloud Key Plus

The UniFi Cloud Key Plus provides centralized management for the UniFi wireless environment.

Its primary responsibilities include:

- Managing the UniFi access point.
- Hosting the UniFi Network application.
- Supporting the guest captive portal.
- Providing the management interface for UniFi devices.

### UniFi AC Mesh Access Point

The UniFi AC Mesh access point provides wireless access to multiple network segments.

Its primary responsibilities include:

- Providing trusted wireless access on VLAN 20.
- Providing guest wireless access on VLAN 40.
- Using VLAN 10 for infrastructure management.
- Transporting wireless client traffic to the wired network using VLAN tagging.

### Windows Server 2022

The Windows Server 2022 system provides server-side services within the dedicated Servers VLAN.

Its placement on VLAN 30 allows access to the server to be controlled independently from trusted clients, guest devices, and infrastructure management systems.

## Physical Topology

The physical network is centered around the Netgate SG-1100 firewall/router and the Netgear JGS516PE managed switch.

The primary physical connections are:

- The SG-1100 connects to the Netgear switch through the primary LAN uplink.
- The UniFi Cloud Key Plus connects directly to the Netgear switch.
- The UniFi AC Mesh access point connects directly to the Netgear switch.
- The Windows Server 2022 system connects directly to the Netgear switch.
- Administrative and recovery clients connect either through wireless access or designated wired recovery ports.

A simplified physical topology is shown below:

<p align="center">
  <img src="../diagrams/physical-topology.svg"
       alt="Physical topology of the home network lab"
       width="100%">
</p>

### Logical Topology

The network is divided into separate VLANs based on device role and trust level.

The active VLANs are:

- VLAN 1 - Legacy / Recovery
- VLAN 10 - Management
- VLAN 20 - Trusted
- VLAN 30 - Servers
- VLAN 40 - Guest

Each VLAN represents a separate Layer 2 broadcast domain.

Traffic that needs to move between VLANs is routed through pfSense, where firewall policy determines whether that communication is permitted.

A simplified logical topology is shown below:

```text
                         Internet
                            |
                         pfSense
                            |
        +-------------------+-------------------+
        |         |         |         |         |
     VLAN 1     VLAN 10   VLAN 20    VLAN 30   VLAN 40
     Legacy    Management Trusted    Servers    Guest
    /Recovery
```

pfSense provides the Layer 3 gateway and routed security boundary for each VLAN.

Traffic between VLANs must traverse pfSense and is evaluated against the firewall policy associated with the interface where that traffic enters the firewall.

Traffic between devices on the same VLAN may remain entirely at Layer 2 and may therefore not traverse pfSense.

Guest wireless clients on VLAN 40 use UniFi Client Device Isolation to reduce direct Layer 2 communication between clients that share the same VLAN.
```

## Layer 2 and Layer 3 Responsibilities

The network deliberately separates Layer 2 forwarding responsibilities from Layer 3 routing and security policy.

### Layer 2 Responsibilities

Layer 2 functions are primarily handled by the Netgear JGS516PE switch and the UniFi access point.

These responsibilities include:

- Maintaining VLAN separation on the switched network.
- Forwarding Ethernet frames within the same VLAN.
- Carrying tagged VLAN traffic across trunk and hybrid links.
- Providing untagged access for devices assigned to a single VLAN.
- Mapping wireless SSIDs to the appropriate VLANs.

Traffic between devices on the same VLAN may remain entirely at Layer 2 and may not traverse pfSense.

### Layer 3 Responsibilities

Layer 3 functions are handled by pfSense.

These responsibilities include:

- Providing the default gateway for each VLAN.
- Routing traffic between VLANs.
- Applying firewall policy to routed traffic.
- Providing access to external networks through the WAN connection.
- Providing DHCP and DNS services for the defined network segments.

Because inter-VLAN traffic must pass through pfSense, firewall policy can enforce security boundaries between the logical network zones.

## Traffic Flow Model

Traffic behavior depends on the source VLAN, destination VLAN, and firewall policy applied by pfSense.

### Trusted Client to Windows Server

A trusted client on VLAN 20 that connects to the Windows Server on VLAN 30 sends traffic to its default gateway on pfSense.

pfSense then evaluates the applicable firewall rules before routing the traffic toward the Servers VLAN.

```text
Trusted Client
VLAN 20
   |
   v
pfSense
Firewall Policy
   |
   v
Windows Server
VLAN 30
```

### Guest Client to Internet

A guest client on VLAN 40 sends Internet-bound traffic to pfSense.

pfSense permits the allowed outbound traffic and forwards it toward the WAN.

```text
Guest Client
VLAN 40
   |
   v
pfSense
Firewall Policy
   |
   v
Internet
```

### Guest Client to Internal Network

Traffic initiated by a guest client toward protected internal networks is routed to pfSense and blocked by firewall policy unless a specific exception exists.

The captive portal dependency on the UniFi Cloud Key is an example of a narrowly defined exception.

### Same-VLAN Traffic

When two devices are on the same VLAN, their traffic may be switched directly at Layer 2 without traversing pfSense.

This means pfSense firewall rules cannot necessarily control communication between devices that share the same VLAN.

Guest wireless uses UniFi Client Device Isolation for Layer 2 guest-to-guest isolation

## Administrative Access Model

Administrative access is intentionally limited to known management paths rather than being available equally from every network segment.

### Trusted Administrative Clients

Authorized administrative clients normally operate from the Trusted VLAN.

These clients may be granted routed access to selected infrastructure services in the Management VLAN and to specific server services where required.

Ordinary Trusted clients do not receive the same administrative privileges.

### Management VLAN

The Management VLAN is used for infrastructure administration, including devices such as:

- pfSense
- Netgear managed switch
- UniFi Cloud Key Plus
- UniFi access point

Access to management services is restricted by firewall policy and by the capabilities of the individual devices.

### Recovery Access

Dedicated wired recovery paths are maintained so that administrative access can be restored if normal Trusted-to-Management access fails.

The network preserves:

- a Management recovery path for controlled administrative access
- a Legacy / Recovery path for break-glass access

These recovery paths are intentionally kept separate from ordinary client access.

## Recovery Architecture

The network includes dedicated recovery paths so that administrative access can be restored if a normal management path becomes unavailable.

### Management Recovery Path

A dedicated wired Management access path is maintained for controlled recovery.

This path provides direct access to the Management VLAN and is intended for administrative clients that need to reach infrastructure services without relying on the normal Trusted-to-Management path.

### Legacy / Recovery Path

A separate Legacy / Recovery path is preserved as a break-glass option.

This path is intentionally more permissive than the normal segmented design and exists to provide emergency access when other administrative methods fail.

Because of its elevated privilege, it should be used only for recovery and troubleshooting.

### Recovery Design Principle

Recovery access should remain available while changes are being made to routing, firewall policy, VLAN transport, or management services.

A replacement path should be tested successfully before an existing recovery path is removed or restricted.

## Design Constraints

The current architecture includes several important constraints and assumptions.

- The Netgate SG-1100 uses an internal switch architecture that requires additional VLAN configuration beyond creating a pfSense VLAN interface.
- Inter-VLAN security policy depends on traffic being routed through pfSense.
- Same-VLAN communication may bypass pfSense and must be controlled through Layer 2 or endpoint mechanisms where isolation is required.
- The Netgear switch is most reliably administered from a host directly resident on the Management VLAN.
- Recovery access depends on preserving the dedicated Management and Legacy recovery paths.
- The documented security model currently focuses on IPv4. Equivalent policy should be established before routed IPv6 is introduced.
- Infrastructure addresses learned through DHCP should not be assumed permanent unless static mappings or reservations are configured.