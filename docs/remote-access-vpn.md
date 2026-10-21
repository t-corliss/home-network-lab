# Remote Access VPN

## Purpose

This document describes the WireGuard remote-access design used to provide secure administrative access to selected internal services without exposing those services directly to the Internet.

The VPN is implemented as a dedicated routed security zone on pfSense.

## Design Principles

- Terminate remote-access VPN traffic on pfSense.
- Keep VPN clients in a dedicated Layer 3 security zone.
- Use split tunneling so only approved home-lab destinations traverse the VPN.
- Permit only narrowly defined administrative services.
- Do not expose RDP directly to the WAN.
- Preserve wired Management and Legacy / Recovery paths for break-glass access.
- Keep private keys, exported client configurations, and live firewall backups out of Git.

## Addressing

| Component | Address | Purpose |
|---|---|---|
| WireGuard network | `10.10.50.0/24` | Remote-access routed network |
| pfSense `WG_REMOTE` | `10.10.50.1/24` | WireGuard gateway/interface address |
| Approved client peer | `10.10.50.10/32` | Remote administrative client |

`10.10.50.0/24` is not VLAN 50.

No 802.1Q tagging, switch VLAN membership, PVID, or SG-1100 internal-switch VLAN entry is required for this network.

## Tunnel Design

WireGuard terminates on pfSense and listens on UDP port `51820` on the WAN interface.

The WireGuard tunnel is assigned to the pfSense interface `WG_REMOTE`.

The assigned interface uses:

```text
Interface: WG_REMOTE
IPv4 configuration: Static IPv4
Address: 10.10.50.1/24
Gateway: none
```

The approved client peer uses:

```text
Tunnel address: 10.10.50.10/32
Persistent keepalive: 25 seconds
```

The current public WAN address and all WireGuard key material are intentionally omitted from this repository.

## Remote Access Policy

| Source | Destination | Protocol / Port | Purpose |
|---|---|---|---|
| `10.10.50.10` | DC01 `10.10.30.11` | TCP `3389` | Remote Desktop administration |
| `10.10.50.10` | pfSense `10.10.10.1` | TCP `443` | pfSense web administration |
| `10.10.50.10` | UniFi Cloud Key `10.10.10.20` | TCP `443` | UniFi administration |

Other VPN-initiated internal traffic is blocked unless a specific exception is added.

The WAN rule allowing UDP `51820` permits traffic to the WireGuard service on pfSense itself. It is not a port forward to an internal host.

TCP `3389` is not exposed directly to the Internet.

## Split Tunneling

The remote client routes only explicitly approved destinations through the WireGuard interface.

Current destinations are:

```text
10.10.30.11/32
10.10.10.1/32
10.10.10.20/32
```

All unrelated Internet traffic continues to use the remote network's normal default gateway.

## Validation

The following tests were completed successfully from an external network:

- WireGuard establishes a current handshake.
- The client uses `10.10.50.10` as its tunnel address.
- RDP to DC01 at `10.10.30.11:3389` succeeds.
- pfSense HTTPS at `10.10.10.1:443` succeeds.
- Cloud Key HTTPS at `10.10.10.20:443` succeeds.
- Normal unrelated Internet traffic remains outside the VPN tunnel.
- No direct WAN RDP exposure is required.

## Recovery and Security Notes

WireGuard is a remote administration path, not a replacement for wired recovery access.

Remote VPN access depends on WAN connectivity, pfSense availability, the WireGuard service, correct interface addressing, routing, and firewall policy.

The dedicated Management and Legacy / Recovery paths should remain available for break-glass administration.