# Recovery and Emergency Access

## Purpose

This document defines the recovery paths and emergency procedures used to restore administrative access when normal management connectivity is unavailable.

The goal is to preserve a known-good path into the network while troubleshooting routing, firewall, VLAN, switching, or management-plane problems.

## Recovery Philosophy

Recovery access should remain independent enough from the normal administrative path that a single configuration mistake does not remove every way back into the network.

The preferred approach is:

1. Preserve at least one known-good recovery path.
2. Change one layer or control at a time.
3. Validate access before removing or restricting the previous working path.
4. Use the least-privileged recovery method that still allows the problem to be corrected.
5. Restore the normal security posture after recovery is complete.

The dedicated Management recovery path should normally be preferred over the broader Legacy / Recovery path.

The Legacy / Recovery path exists as a break-glass option when the more controlled Management path is not sufficient.

## Before Making Risky Changes

Before changing trunk behavior, VLAN membership, firewall policy, management addressing, or other settings that could interrupt administrative access:

- Confirm that at least one recovery path is available.
- Confirm that the recovery client can obtain the expected address.
- Confirm that the relevant management interface is reachable.
- Save or export a current configuration backup where appropriate.
- Avoid changing multiple layers at the same time.
- Keep a rollback path available until the replacement configuration has been validated.

## Management Recovery - Port 8

Port 8 provides dedicated wired access to the Management VLAN.

It uses VLAN 10 untagged with PVID `10`.

This is the preferred recovery path for restoring administrative access when normal Trusted-to-Management connectivity is unavailable.

### Procedure

1. Connect an approved administrative laptop to switch port 8.
2. Confirm that the client receives a Management VLAN address in the `10.10.10.0/24` subnet.
3. If a reservation is configured, verify that the client receives its expected recovery address.
4. Test access to pfSense at `10.10.10.1`.
5. Test access to the Netgear switch management interface.
6. Test access to the UniFi Cloud Key if required.
7. Correct the affected configuration while preserving the recovery connection.
8. Validate the normal administrative path before disconnecting from port 8.

### Expected Behavior

The recovery client should be able to obtain Management VLAN connectivity without relying on the normal Trusted VLAN path.

Access to pfSense should remain narrowly controlled by the existing recovery-client policy rather than by broad Management-to-firewall access.

## Legacy / Recovery - Port 9

Port 9 provides break-glass wired access to the Legacy / Recovery VLAN.

It uses VLAN 1 untagged with PVID `1`.

This path is intentionally broader and more permissive than the Management recovery path and should be used only when the preferred recovery method is unavailable or insufficient.

### Procedure

1. Connect an approved administrative laptop to switch port 9.
2. Confirm that the client receives an address in the `192.168.1.0/24` subnet.
3. Verify reachability to pfSense at `192.168.1.1`.
4. Use the Legacy / Recovery path only for the minimum amount of time required to restore normal management access.
5. Correct the failed routing, firewall, VLAN, or management configuration.
6. Validate the Management recovery path on port 8.
7. Validate the normal Trusted-to-Management administrative path.
8. Disconnect from port 9 once a more controlled recovery path has been restored.

### Expected Behavior

The Legacy / Recovery network provides broad access intended for emergency troubleshooting and recovery.

Because this path is highly privileged, it should not be used as a normal client network or general-purpose administrative network.

## Recovering pfSense Access

If the normal pfSense administrative path is unavailable, use the narrowest available recovery path first.

### Preferred Method

1. Connect an approved administrative laptop to port 8.
2. Confirm Management VLAN addressing.
3. Attempt to reach pfSense at `10.10.10.1`.
4. If access succeeds, correct the affected firewall, VLAN, or interface configuration.
5. Re-test the normal administrative path before disconnecting the recovery client.

### Break-Glass Method

If Management recovery is not sufficient:

1. Connect an approved administrative laptop to port 9.
2. Confirm Legacy / Recovery addressing.
3. Attempt to reach pfSense at `192.168.1.1`.
4. Correct only the configuration required to restore a more controlled management path.
5. Validate port 8 recovery access.
6. Validate normal administrative access.
7. Exit the Legacy / Recovery path.

## Recovering Switch Access

The Netgear switch is most reliably managed from a host directly connected to the Management VLAN.

### Procedure

1. Connect an approved administrative laptop to port 8.
2. Confirm that the client receives an address in the `10.10.10.0/24` Management subnet.
3. Attempt to reach the switch management interface.
4. If the switch is unreachable but the client has correct Management addressing:
   - confirm basic IP reachability to other Management devices
   - verify that the client can reach pfSense at `10.10.10.1`
   - determine whether the problem is specific to the switch management interface
5. If the client does not receive correct Management addressing:
   - suspect VLAN 10 membership, PVID configuration, or Layer 2 transport on port 8
   - avoid making additional trunk or VLAN changes until a known-good recovery path is available
6. Do not modify port 1 unless another recovery path has been confirmed.
7. Once switch management access is restored, verify port 8 VLAN membership and PVID configuration directly.
8. Validate pfSense, Cloud Key, and normal administrative connectivity before leaving recovery mode.

## Emergency Troubleshooting Sequence

When administrative access or network reachability fails, use the following order:

1. Check physical link state.
2. Confirm the client receives the expected IP address.
3. Confirm the expected VLAN and PVID behavior.
4. Test the local default gateway.
5. Test access to the required management interface.
6. Review pfSense firewall logs if routed traffic is involved.
7. Use packet capture when the failure point is still unclear.
8. Change one layer at a time.
9. Validate recovery access before making additional changes.
10. Restore the normal administrative path before ending the recovery session.

## Restoring a pfSense Backup

A configuration backup should be used when a known-good configuration exists and manual correction would be slower, less reliable, or more error-prone.

Before restoring a backup:

1. Confirm that the backup predates the configuration problem.
2. Confirm that the backup is from the correct firewall.
3. Preserve at least one known-good recovery path.
4. Avoid restoring over a working configuration unless the expected rollback effect is understood.

After restoration:

1. Confirm that pfSense is reachable.
2. Confirm interface addressing.
3. Confirm VLAN connectivity.
4. Confirm DNS and DHCP behavior.
5. Confirm administrative access.
6. Validate critical firewall policy before returning the network to normal operation.

## Protected Ports and Settings

Certain interfaces and settings should be treated as high-risk during recovery because changes can affect multiple network segments or remove administrative access.

Use additional caution with:

- Netgear port 1, which carries the pfSense uplink and multiple VLANs.
- Netgear port 3, which carries access point management and wireless client VLANs.
- Netgear port 8, which provides Management recovery access.
- Netgear port 9, which provides Legacy / Recovery break-glass access.
- SG-1100 internal switch VLAN membership.
- pfSense interface assignments.
- firewall rules controlling Management and recovery access.

Do not modify multiple critical paths at the same time unless a separate recovery method has already been verified.

## When to Roll Back

Rollback should be considered when a change produces unexpected behavior and the cause cannot be identified quickly without risking additional loss of access.

Examples include:

- loss of administrative connectivity
- loss of VLAN transport
- incorrect addressing
- widespread DNS or DHCP failure
- unexpected firewall behavior
- failure affecting multiple network segments at once

A rollback should return the network to the last known-good state before further troubleshooting continues.

