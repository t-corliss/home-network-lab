# Wireless and Guest Access

## Purpose

This document describes how the UniFi wireless environment integrates with the segmented network.

It covers access point management, SSID-to-VLAN mapping, guest isolation, captive portal behavior, Cloud Key dependencies, and the Layer 2 relationship between the wireless and wired networks.

## UniFi Architecture

The UniFi environment consists of:

- UniFi Cloud Key Plus
- UniFi AC Mesh access point
- Trusted wireless network
- Guest wireless network

The UniFi components provide wireless access and management, while pfSense remains responsible for Layer 3 routing, DHCP, DNS, and firewall policy.

UniFi therefore participates primarily in Layer 2 transport and wireless policy rather than acting as the network gateway.

## Third-Party Gateway Model

The UniFi Network application is configured to operate with a third-party gateway.

pfSense, rather than a UniFi gateway device, provides the Layer 3 gateway for the wireless VLANs.

As a result:

- UniFi defines the wireless networks and their VLAN associations.
- The access point tags client traffic according to the configured SSID.
- The Netgear switch transports those VLANs between the access point and pfSense.
- pfSense provides addressing, routing, DNS, and security policy for wireless clients.

## Access Point Management

The UniFi AC Mesh access point uses VLAN 10 for management.

Its management traffic is carried untagged on the switch port connected to the access point, with the port PVID set to `10`.

Wireless client traffic is carried separately using VLAN tags based on the SSID.

This allows the access point itself to remain on the Management VLAN while simultaneously serving clients on other VLANs.

## Trusted Wi-Fi

The trusted wireless network uses the SSID `Quesadilla`.

Clients connected to this SSID are assigned to VLAN 20.

The access point tags Trusted client traffic with VLAN ID `20` before sending it onto the wired network.

pfSense then provides:

- DHCP addressing
- DNS service
- default gateway service
- routed firewall policy

Trusted wireless clients therefore receive the same Layer 3 policy as other devices on VLAN 20.

## Guest Wi-Fi

The guest wireless network uses the SSID `Burrito`.

Clients connected to this SSID are assigned to VLAN 40.

The access point tags Guest client traffic with VLAN ID `40` before sending it onto the wired network.

pfSense provides addressing, DNS, routing, and firewall policy for the Guest VLAN.

The Guest wireless network also uses UniFi-specific controls to reduce direct communication between guest clients.

## Guest Client Isolation

The Guest wireless network uses UniFi Client Device Isolation to reduce direct communication between wireless clients on VLAN 40.

This Layer 2 control complements the routed firewall policy enforced by pfSense.

pfSense controls communication between the Guest VLAN and other routed networks, while UniFi client isolation helps restrict direct guest-to-guest communication that may otherwise remain within the same VLAN.

This distinction is important because same-VLAN traffic may not traverse pfSense.

## Captive Portal

The Guest wireless network uses the UniFi captive portal to present a guest access page before normal Internet use.

The captive portal depends on coordination between the wireless client, the UniFi access point, the Cloud Key, and pfSense.

### Authentication Flow

A simplified Guest portal flow is:

```text
Guest Client
   |
   v
UniFi Access Point
   |
   v
Guest VLAN 40
   |
   v
pfSense
   |
   +--> UniFi Cloud Key
   |      TCP 8880
   |
   v
Internet
```

The client first joins the Guest SSID and receives addressing from pfSense.

When portal interaction is required, the client must be able to reach the UniFi Cloud Key so that the captive portal can be presented and processed.

### Cloud Key Dependency

The UniFi Cloud Key hosts services required for Guest portal functionality.

Guest clients therefore require a narrowly defined exception that permits access to the Cloud Key on TCP port `8880`.

This exception exists only to support the captive portal and should not be interpreted as general Guest access to the Management VLAN.

### TCP 8880 Exception

The Guest firewall policy permits:

`Guest VLAN -> UniFi Cloud Key -> TCP 8880`

All other Guest access to protected Management resources remains blocked unless explicitly required.

## AP Port VLAN Behavior

The switch port connected to the UniFi AC Mesh access point carries both management and wireless client traffic.

Its VLAN behavior is:

- VLAN 10 untagged for access point management.
- VLAN 20 tagged for Trusted wireless clients.
- VLAN 40 tagged for Guest wireless clients.
- PVID `10` for untagged traffic entering the switch from the access point.

This allows a single physical Ethernet connection to carry multiple logical networks.

The detailed switch configuration remains authoritative in `switching.md`; this section documents only the wireless-facing behavior required to understand the access point design.

