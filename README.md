\# Home Network Lab



\## Overview



This repository documents the design, configuration, validation, recovery procedures, and operational lessons for my segmented home network and personal lab environment.



The network uses VLAN-based Layer 2 segmentation with pfsense providing Layer 3 routing and firewall policy between security zones.



\## Design Goals





\- Separate management, trusted clients, servers, guests, and recovery access.

\- Apply least-privilege policy between network segments.

\- Maintain reliable administrative and emergency recovery paths.

\- Document the environment well enough to support troubleshooting and future changes.

\- Use repeatable validation and change-control procedures.



\## Hardware



The current environment includes:



\- Netgate SG-1100 running pfSense

\- Netgear JGS516PE managed switch

\- UniFi Cloud Key Plus

\- UniFi AC Mesh wireless access point

\- Windows Server 2022

\- Administrative client systems



\## Network Zones



| VLAN | Name | Purpose |

|---|---|---|

| 1 | Legacy / Recovery | Break-glass and recovery access |

| 10 | Management | Infrastructure administration |

| 20 | Trusted | Trusted client devices |

| 30 | Servers | Server workloads |

| 40 | Guest | Isolated guest wireless access |



Detailed addressing and security policy are maintained in the documentation rather than duplicated here.



\## High-Level Topology





Internet

   |

pfSense

   |

   | VLAN trunk

   |

Netgear Managed Switch

   |

   +-- Management devices

   +-- Trusted clients

   +-- Windows Server

   +-- Recovery ports

   |

   +-- UniFi Access Point

          |

          +-- Trusted Wi-Fi

          +-- Guest Wi-Fi

