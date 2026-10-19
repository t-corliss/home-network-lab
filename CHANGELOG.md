# Changelog

This file records significant validated changes to the home network lab.

Minor documentation edits and routine Git maintenance are not listed unless they materially affect the documented network state.

## Unreleased

Changes that have been documented but not yet grouped into a named release are listed here.

### Added

- Segmented VLAN architecture for Management, Trusted, Servers, Guest, and Legacy / Recovery networks.
- pfSense inter-VLAN firewall policy using least-privilege rules and administrative exceptions.
- Dedicated Management and Legacy recovery paths.
- UniFi Trusted and Guest wireless networks with VLAN assignment.
- Guest captive portal support through a narrow Cloud Key TCP `8880` exception.
- Guest client isolation for same-VLAN wireless separation.
- Network architecture, switching, addressing, firewall, wireless, recovery, validation, troubleshooting, and VLAN deployment documentation.

### Changed

- Replaced broad or temporary troubleshooting rules with validated least-privilege policy.
- Established a dedicated Management recovery path as the preferred administrative fallback.
- Added explicit Trusted-to-Guest isolation.
- Standardized documentation and change control through Git and GitHub.

### Fixed

- Corrected Guest firewall protocol configuration that prevented reliable connectivity.
- Corrected Servers VLAN DNS handling by placing the required DNS exception above the internal-network block.
- Corrected Management DNS service from TCP/UDP port `853` to port `53`.
- Added the required Guest-to-Cloud-Key TCP `8880` exception for captive portal operation.

## v1.0.0 10-18-26

### Added

- Added editable draw.io source files and rendered SVG diagrams for physical topology, VLAN transport, security policy, recovery access, and wireless SSID-to-VLAN flow.
- Embedded network diagrams into the relevant documentation pages and the repository README.
- Added repository security guidance for sensitive configuration data, backups, credentials, and sanitized examples.

### Changed

- Stabilized Management infrastructure addressing:
  - Cloud Key Plus: `10.10.10.20`
  - UniFi AC Mesh AP: `10.10.10.21`
  - Netgear managed switch: `10.10.10.22`
- Updated the Guest captive portal rule to reference the `CLOUD_KEY` alias.
- Clarified the exact contents and purpose of `MANAGEMENT_BLOCKED_NETWORKS`.
- Updated documentation to match the audited live configuration.
- Improved topology, recovery, wireless, and policy documentation with embedded diagrams.

### Fixed

- Added explicit Servers-to-Guest isolation because the broader internal-network alias does not include the Guest VLAN.
- Removed stale Management infrastructure address references.
- Removed stale VLAN 50 references.
- Corrected documentation structure, heading consistency, and logical topology presentation.