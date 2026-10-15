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