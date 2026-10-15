# Adding a New VLAN

## Purpose

This document defines the procedure for adding a new VLAN to the home network lab.

The goal is to introduce new network segments in a controlled way while preserving management access, maintaining security boundaries, and validating each layer before moving to the next.

This procedure should be used for future VLAN additions rather than making ad hoc changes across pfSense, the SG-1100 internal switch, the Netgear switch, and UniFi independently.

## Pre-Change Planning

Before creating a new VLAN, define its intended role and security requirements.

Document:

- VLAN ID
- VLAN name
- IPv4 subnet
- pfSense gateway address
- DHCP requirements
- expected client types
- required access to internal networks
- required Internet access
- management requirements
- whether wireless access is needed
- whether dedicated access ports are needed
- rollback expectations

Confirm that at least one known-good recovery path is available before making changes.

Do not begin by configuring devices individually. Define the intended end state first.

## Step 1 - Define the Security Role

Before creating the VLAN, decide what trust level the new network should have.

Questions to answer include:

- Which existing VLANs should this network be able to initiate connections toward?
- Which existing VLANs should be able to initiate connections toward it?
- Does it need access to Management services?
- Does it need access to Servers?
- Does it need only Internet access?
- Does it require same-VLAN client isolation?
- Does it require any narrowly defined exceptions?

The firewall policy should be designed before the final broad egress rule is created.

## Step 2 - Create the VLAN on `mvneta0`

On the SG-1100, custom VLANs used on the LAN side are created with `mvneta0` as the parent interface.

When creating the VLAN:

1. Select `mvneta0` as the parent interface.
2. Enter the planned VLAN ID.
3. Assign a descriptive name or description.
4. Save the VLAN configuration.

Creating the VLAN in pfSense defines the logical VLAN interface, but it does not by itself guarantee that tagged traffic can reach the physical LAN port.

The SG-1100 internal switch must also be configured for the VLAN.

## Step 3 - Configure the SG-1100 Internal Switch

After creating the VLAN interface, add the same VLAN to the SG-1100 internal switch.

For VLANs transported through the physical LAN path, configure the VLAN with:

`0t,2t`

Both internal switch ports are tagged members of the VLAN.

This allows the VLAN to pass between the SG-1100 CPU and the physical LAN-facing switch port.

> [!WARNING]
> Creating a VLAN on `mvneta0` without configuring the corresponding SG-1100 internal switch membership can result in a VLAN that exists logically in pfSense but does not reach the external LAN connection.

Do not modify the built-in SG-1100 system VLANs `4090`, `4091`, or `4092` as part of normal custom VLAN creation.

## Step 4 - Assign the pfSense Interface

After the VLAN exists on `mvneta0` and the SG-1100 internal switch is configured to transport it, assign the VLAN as a pfSense interface.

When assigning the interface:

1. Add the new VLAN interface under pfSense interface assignments.
2. Enable the interface.
3. Give it a descriptive name that reflects the VLAN's role.
4. Configure the planned IPv4 gateway address.
5. Save and apply the configuration.

For example, a new VLAN using subnet `10.10.50.0/24` might use pfSense address `10.10.50.1`.

The interface address becomes the default gateway for clients on that VLAN.

## Step 5 - Configure Addressing and DHCP

After the interface is assigned, configure the addressing plan for the new VLAN.

Confirm:

- subnet
- pfSense gateway address
- DHCP pool
- any reserved addresses
- any infrastructure addresses that need to remain stable

If DHCP is required:

1. Enable DHCP service for the new interface.
2. Define a dynamic pool that does not overlap with gateway or reserved addresses.
3. Add reservations for systems that require predictable addressing.
4. Save and apply the DHCP configuration.

The addressing pattern used elsewhere in the lab can be reused where appropriate:

- pfSense gateway at `.1`
- lower addresses reserved for infrastructure and static mappings
- dynamic DHCP clients in a higher range such as `.100` through `.199`

Do not assume every new VLAN must use the same exact pattern if its requirements differ.

## Step 6 - Configure DNS

If clients on the new VLAN will use pfSense for DNS, confirm that DNS service is available on the new interface.

Verify that:

- pfSense listens for DNS queries from the new VLAN.
- firewall policy permits the new VLAN to reach pfSense DNS on port `53`.
- broader internal-network block rules do not unintentionally block the required DNS path.

If the new VLAN should use an external DNS service instead, document that design explicitly rather than assuming the same DNS model used by the existing VLANs.

Test DNS separately from general Internet connectivity.

## Step 7 - Configure Netgear VLAN Membership

After pfSense is prepared to route the new VLAN, configure the Netgear switch to carry it across the required ports.

Determine which ports need the new VLAN and whether each port should carry it as:

- tagged
- untagged
- not a member

For trunk or hybrid links, add the VLAN as a tagged member where required.

For access ports, add the VLAN as an untagged member and set the port PVID to the same VLAN ID.

Always verify the role of each affected port before applying changes.

### Tagged and Untagged Behavior

Use tagged membership when the connected device understands VLAN tagging and must carry multiple VLANs.

Use untagged membership when the connected device should receive ordinary Ethernet frames without VLAN tags.

For a normal access port:

```text
VLAN membership = Untagged
PVID = VLAN ID
```

For a trunk or hybrid port:

```text
one or more VLANs = Tagged
optional native / untagged VLAN
PVID = native / untagged VLAN
```

## Step 8 - Configure Access or Trunk Ports

After the VLAN exists on pfSense and the switch can carry it, configure the endpoint-facing ports that will use the new VLAN.

For an access port:

1. Add the new VLAN as an untagged member.
2. Set the port PVID to the new VLAN ID.
3. Remove unnecessary VLAN memberships that do not belong on that port.
4. Connect a test client.
5. Confirm that the client receives addressing from the expected subnet.

For a trunk or hybrid port:

1. Add the new VLAN as a tagged member.
2. Preserve any existing native or untagged VLAN required by the connected device.
3. Confirm that the connected device is configured to recognize the new VLAN tag.
4. Avoid removing existing VLANs until the new configuration has been validated.

Use the simplest possible port configuration during initial testing.

## Step 9 - Configure UniFi if Required

If the new VLAN will be used by a wireless SSID, configure the corresponding network in UniFi.

When adding wireless support:

1. Create or update the UniFi network definition with the correct VLAN ID.
2. Create or update the SSID that should use the VLAN.
3. Confirm that the access point uplink carries the VLAN as tagged traffic.
4. Confirm that the AP management VLAN remains unchanged unless intentionally redesigned.
5. Apply any required wireless isolation or guest controls.

Remember that UniFi does not provide the Layer 3 gateway in this design.

pfSense remains responsible for:

- default gateway service
- DHCP
- DNS
- inter-VLAN routing
- firewall policy

## Step 10 - Build Firewall Policy

Create the firewall policy for the new VLAN only after its intended trust model has been defined.

Start with the most specific required services and exceptions.

A recommended rule-building order is:

1. Allow required access to pfSense services such as DNS.
2. Allow narrowly defined internal exceptions.
3. Block access to protected internal networks that should remain unreachable.
4. Allow remaining Internet egress if required.

Avoid placing a broad `Any` allow rule above internal-network blocks.

Because pfSense uses first-match evaluation, rule order is part of the security policy.

### Example Policy Pattern

```text
Allow VLAN -> pfSense DNS
Allow VLAN -> specific required internal service
Block VLAN -> protected internal networks
Allow VLAN -> remaining destinations
```

This pattern allows the VLAN to reach only the internal services it actually needs while preserving general Internet access where appropriate.

If the VLAN should not have Internet access, omit the final broad egress rule.

### Review Existing Aliases

Before using an existing alias, confirm that its contents still match the intended policy.

A broad alias such as `INTERNAL_NETWORKS` may include networks or infrastructure addresses that require explicit exceptions.

If a new VLAN becomes part of the protected internal environment, update the relevant aliases and review every firewall policy that depends on them.

## Step 11 - Validate Layer 2

Begin by confirming that the new VLAN is being transported correctly at Layer 2.

Use a simple wired access port whenever possible.

Validate:

1. The test port has the expected VLAN membership.
2. The test port PVID matches the intended VLAN.
3. The client establishes physical link.
4. The client receives an address from the expected subnet.
5. The client's MAC address appears on the expected VLAN or switch port where visibility is available.

If the client does not receive the expected addressing, troubleshoot Layer 2 transport before moving on to routing or firewall policy.

A successful wired test helps isolate the VLAN transport path from wireless configuration.

## Step 12 - Validate Layer 3

After Layer 2 behavior is confirmed, validate Layer 3 connectivity.

Test:

1. The client has the expected IP address and subnet mask.
2. The client has the expected pfSense default gateway.
3. The client can reach the local pfSense gateway.
4. DNS works if pfSense is intended to provide DNS.
5. Internet routing works if the VLAN is intended to have Internet access.
6. Required routed internal services are reachable.

Do not assume that successful DHCP proves routing or DNS is functioning correctly.

## Step 13 - Validate Security Policy

After Layer 2 and Layer 3 connectivity are working, verify both permitted and blocked traffic.

Test:

- required internal services
- required management access
- Internet access where intended
- blocked access to protected internal networks
- blocked access to recovery networks where appropriate
- same-VLAN isolation where required
- application-specific exceptions

A security validation is incomplete if it tests only allowed traffic.

Confirm that prohibited traffic is actually blocked.

Review pfSense firewall logs if the observed behavior does not match the intended policy.

## Step 14 - Update Documentation

After the new VLAN has been validated, update the repository so the documented state matches the deployed state.

At minimum, review and update:

- `README.md` if the new VLAN changes the high-level network-zone summary
- `docs/architecture.md` if the logical architecture changes
- `docs/vlans-addressing.md` with the VLAN ID, subnet, gateway, and DHCP details
- `docs/switching.md` with any new VLAN membership, PVID, or port-role changes
- `docs/firewall-policy.md` with the intended security policy
- `docs/wireless.md` if the VLAN is used by a wireless SSID
- `docs/validation.md` with any new acceptance tests
- `docs/recovery.md` if recovery behavior changes
- `CHANGELOG.md` with the operational change

Do not treat documentation as an optional cleanup task.

A VLAN addition is not complete until the repository reflects the validated network state.

## Step 15 - Create Backup

After the new VLAN has been validated and the documentation has been updated, create a new known-good configuration backup.

The backup should represent the validated post-change state.

Confirm that:

- the network is functioning as intended
- administrative access works
- recovery paths remain available
- security policy has been tested
- the documentation is current

Store live configuration backups outside the Git repository.

Do not commit raw firewall or infrastructure backups that may contain sensitive information.

## Rollback Procedure

If the new VLAN causes unexpected behavior or administrative access becomes unreliable, return to the last known-good state before making additional changes.

A rollback may include:

1. Disable or remove the new firewall rules.
2. Remove the new VLAN from endpoint-facing switch ports.
3. Restore previous trunk or hybrid port membership where required.
4. Remove or disable the UniFi SSID or VLAN mapping if applicable.
5. Disable or remove the pfSense VLAN interface if necessary.
6. Remove the custom SG-1100 internal switch VLAN entry only after confirming that it is no longer required.
7. Restore a known-good pfSense backup if manual rollback is not sufficient.
8. Confirm that normal administrative access is restored.
9. Validate the existing network baseline before attempting the change again.

Do not continue stacking new changes on top of an unverified failed state.

Return to a known-good baseline first, then investigate the failure.
